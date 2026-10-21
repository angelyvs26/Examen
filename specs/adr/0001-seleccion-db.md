# ADR-0001: Selección de la base de datos de gestión

| Campo | Valor |
|---|---|
| Estado | Aceptada |
| Fecha | 2026-10-21 |
| Decisores | Equipo de arquitectura (com.examen.gre) |
| Supervisores | Responsable de plataforma |
| Módulo afectado | Transversal (emisión GRE, catálogos, validación, firma/envío) |
| Etiquetas | #adr #arquitectura #persistencia |

## Contexto

El sistema es un emisor de la Guía de Remisión Electrónica (GRE) de la SUNAT (Perú) que debe:

1. Persistir la GRE emitida (cabecera, ítems, documentos relacionados) con su XML UBL 2.1 firmado, su ZIP, su estado y el ticket/CDR devuelto por SUNAT.
2. Generar **numeración correlativa por serie** (`T###-NNNNNNNN` / `V###-NNNNNNNN`) sin huecos bajo concurrencia, con bloqueo por serie y verificación de estado contra SUNAT.
3. Registrar la trazabilidad de cada envío: fecha/hora, intentos, códigos de retorno ERROR/OBSERV/REST y su clasificación interna (ver [mapeo-errores.md](../features/0004-validacion-documento/mapeo-errores.md)).
4. Servir como dato de referencia los catálogos SUNAT (01, 06, 13, 18, 20, 25, 55, 61, 62, 63, 64, 65, D-37) y las listas de apoyo (RENIEC, MTC, establecimientos, DAM/DS, Manifiesto de Carga), con carga batch y versionado por fecha de vigencia.
5. Operar con multi-tenant por RUC emisor: un mismo RUC puede tener series propias y sólo su personal debe verlas.

Restricciones del contexto:

- El servicio es **stateful** en la parte de correlativos (no se puede emitir dos veces el mismo correlativo para la misma serie, ni dejar huecos no justificados).
- El flujo es transaccional: emit + reservar correlativo + registrar estado deben ser atómicos o compensables.
- El XML firmado es un **blob inmutable** de hasta ~300 KB por documento; se necesita consultarlo sin reinterpretarlo.
- La base de datos no es el sistema de registro de SUNAT: es un sistema de apoyo, por lo que **no puede ser la única copia** de un comprobante ya aceptado (debe poder reconstruirse desde el XML).
- Debe existir en un stack autoalojado (Docker/Compose o Kubernetes) sin dependencia de servicios SaaS de pago, y ser operable por un equipo pequeño.

## Problema

¿Qué motor de base de datos relacional (o no relacional) se usa para la gestión de la GRE, de modo que soporte numeración correlativa transaccional, almacenamiento del XML firmado, consultas por catálogo y trazabilidad, con el menor costo operativo y una ruta simple de HA?

## Opciones consideradas

### Opción A — PostgreSQL 16

- **Descripción**: base relacional open source con MVCC, `SERIALIZABLE`/locks, secuencias, `JSONB`, `CHECK`, particionado por rango de fecha, extensión `pgcrypto`, replicación streaming y `LISTEN/NOTIFY`.
- **Ventajas**:
  - Secuencias y locks de fila permiten un correlativo por serie con `INSERT ... ON CONFLICT` sobre una tabla de contador, sin huecos y sin aplicación distribuida extra.
  - Restricciones declarativas (`CHECK`, `UNIQUE`, `FK`)	modelan directamente invariantes del dominio (serie + correlativo único, fecha no futura, total ≥ 0).
  - `JSONB` indexable para guardar y consultar el payload de emisión y las respuestas de SUNAT sin duplicar tablas.
  - Curva de aprendizaje baja para el equipo Java: driver JDBC estable, Spring Data JPA/JDBC, migraciones con Flyway/Liquibase(first-class).
  - Extensible a escala (réplicas de lectura particionadas por fecha) sin cambiar de motor.
- **Desventajas**:
  - No es "serverless": hay que operar el clúster (aunque en modo single-node para el despliegue inicial).
  - Algunas características (secuencias, JSONB) no son portables a otros motores: el SQL queda atado a Postgres.
- **Costos / esfuerzo**: bajo. Docker Compose para desarrollo, imagen oficial para producción, backup con `pg_dump`/`barman`.

### Opción B — MySQL 8 / MariaDB

- **Descripción**: base relacional de altísima adopción.
- **Ventajas**: operación muy conocida, amplio parque de herramientas de backup y monitoreo, `utf8mb4` bien soportado.
- **Desventajas**:
  - `CHECK` y restricciones ignoradas en versiones antiguas; la fuerza de las invariantes depende de triggers, lo que complica el modelo.
  - Aislamiento por defecto `REPEATABLE READ` con fenómenos de gap lock, que entorpece la concurrencia sobre la tabla de correlativos.
  - `JSON` sin índices GIN equivalentes: el filtrado por atributos del payload es más costoso.
- **Costos / esfuerzo**: bajo.

### Opción C — MongoDB

- **Descripción**: base de documentos con esquema flexible.
- **Ventajas**: encaje natural con el XML/JSON y agilidad de desarrollo.
- **Desventajas**:
  - La numeración correlativa transaccional exige transacciones multi-documento o contadores externos, con alta probabilidad de huecos.
  - La integridad referencial entre GRE, ítems, documentos relacionados y catálogos es débil o inexistente.
  - Reportes de trazabilidad y consultas por rango de fechas son menos eficientes que en un motor relacional.
- **Costos / esfuerzo**: medio.

### Opción D — Oracle / SQL Server Enterprise

- **Descripción**: motores corporativos con soporte y licenciamiento pago.
- **Ventajas**: robustez, herramientas de soporte y funcionalidad avanzada.
- **Desventajas**: costo de licenciamiento fuera del perfil del proyecto; dificultad de operación; overestimate de capacidades no necesarias.
- **Costos / esfuerzo**: alto.

## Decisión

Se elige **PostgreSQL 16 como motor de datos de gestión**, con el siguiente esquema de uso:

- **Correlativos**: tabla `serie_correlativo` con clave primaria `(ruc_emisor, tipo_documento, serie)` y columna `ultimo_correlativo`; la reserva se hace dentro de la transacción de emisión con `UPDATE ... RETURNING` (bloqueo de fila), garantizando unicidad y ausencia de huecos por rollback.
- **Documentos**: `gre_documento` (cabecera, estado, XML y ZIP en `BYTEA` o en almacenamiento de objetos con URI + hash SHA-256), `gre_item`, `gre_documento_relacionado`, `gre_envio` (intentos), `gre_observacion` (códigos de retorno con su severidad).
- **XML firmado**: se guarda el `BYTEA` en la base (el volumen objetivo — decenas de miles de documentos/mes — lo permite) con `CHECK` de longitud y hash; la alternativa de objeto S3 queda registrada como opción de evolución.
- **Catálogos**: `catalogo`, `catalogo_item` con `vigencia_desde` / `vigencia_hasta` y `codigo_sunat` indexado; se cargan desde los CSV de [datos/](../features/0003-catalogos-referencia/datos/) mediante jobs de importación idempotentes (ver feature 0003).
- **Aislamiento por RUC**: columna `ruc_emisor` en todas las tablas operativas, con **Row Level Security** activada y una política por tenant, de modo que el filtro por RUC no dependa solo de la aplicación.
- **Auditoría**: columnas `creado_en`, `actualizado_en`, `usuario_creador` y una tabla `auditoria` append-only con los cambios de estado de cada GRE.
- **Migraciones**: Flyway con scripts versionados en `src/main/resources/db/migration`, aplicados automáticamente en el despliegue.

Se descarta una base NoSQL como almacenamiento principal. Un almacén de objetos (S3/MinIO) queda como opción para el XML firmado si el volumen lo justifica (ver ADR-0004 pendiente).

**Tecnología elegida**: PostgreSQL 16.x, driver `org.postgresql:postgresql` (42.x), ORM con Spring Data JPA + Hibernate 6, control de migraciones con Flyway 10.

**Configuración clave**: `max_connections` dimensionado por el pool de la aplicación (HikariCP, `maximum-pool-size` = 20); `log_min_duration_statement = 500 ms`; `statement_timeout` por sesión para las consultas de reportes; particionado mensual de `gre_envio` y `auditoria` cuando el volumen lo justifique.

## Consecuencias

### Positivas

- La invariancia crítica del dominio (correlativo único por serie) queda garantizada por el motor y no por el código de aplicación.
- El mismo modelo sirve para catálogos versionados, con consultas por vigencia simples y correctas.
- Stack estándar y barato: una imagen oficial, backups con herramientas conocidas, sin licencias.
- El Row Level Security hace que un error de filtrado en la capa de aplicación no exponga GRE de otro RUC.

### Negativas / riesgos

- **Dependencia del dialecto Postgres**: el SQL de la capa de repositorio no es portable a otros motores → mitigación: aislar el SQL específico (`ON CONFLICT`, `JSONB`) en clases dedicadas y no dispersarlo en el dominio.
- **Un solo punto de falla en la instalación inicial** → mitigación: réplicas de lectura para reportes y backups automáticos con prueba de restauración trimestral.
- **Un volumen alto de XML en la base degrada el backup** → mitigación: `pg_dump` comprimido + verificación de restauración + umbral para migrar a almacenamiento de objetos.
- **Secuencias y locks pueden generar contención** si un mismo emisor emite masivamente → mitigación: particionar la tabla de correlativos por RUC cuando el análisis lo indique.

### Impacto en otros módulos

| Módulo | Impacto | Acción requerida |
|---|---|---|
| 0001 Emisión GRE Remitente | Alto | La emisión es transaccional con la reserva de correlativo; la idempotencia se apoya en la clave única `(ruc, tipo, serie, correlativo)`. |
| 0002 Emisión GRE Transportista | Alto | Idem, con series `V###` y reserva independiente por serie. |
| 0003 Catálogos | Alto | Importación idempotente por catálogo y versión; se requiere `vigencia_desde` en cada fila. |
| 0004 Validación | Medio | Los códigos de retorno se persisten como tabla normalizada y consultable. |
| 0005 Firma y envío | Medio | Los reintentos se registran en `gre_envio` con su número de intento y clasificación de error. |

## Validación

- Prueba de concurrencia: 50 emisiones simultáneas sobre la misma serie verifican correlativos consecutivos sin huecos ni duplicados (constraint `UNIQUE`).
- Prueba de rollback: una emisión fallida después de reservar el correlativo devuelve el número (transacción única) y la serie queda reutilizable.
- Prueba de RLS: dos RUC emisores concurrentes no pueden leer las GRE del otro (`\set RUC_EMISOR` de sesión).
- Prueba de restauración: se restaura el último backup en un entorno aislado y se cuadra el número de documentos y el correlativo máximo por serie.
- Revisión mensual de `pg_stat_statements` para detectar consultas que excedan el umbral de 500 ms.

## Alternativas en evaluación

- **Almacenamiento de objetos (S3/MinIO) para el XML firmado** con referencia en base de datos: se reevalúa cuando el volumen stored supere ~50 GB o el backup diario exceda la ventana de mantenimiento.
- **MySQL 8** como alternativa si el equipo de operaciones solo tiene experiencia en MySQL y no en Postgres (evaluar en la fase de fundación).

## Referencias

- R.S. 123-2022-SUNAT, Anexo N.° 14 (Estándar UBL 2.1 – Guía de remisión remitente / transportista).
- SUNAT, XLSX de validaciones de la GRE `ValidacionesGREv20250421_0.xlsx` (hojas «General», «CódigosRetorno»).
- [Mapeo de errores SUNAT → sistema interno](../features/0004-validacion-documento/mapeo-errores.md).
- PostgreSQL 16 — Documentación oficial.
