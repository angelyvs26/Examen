# 0001 — Diseño técnico: Emisión de la GRE Remitente

| Campo | Valor |
|---|---|
| Código | 0001 |
| Feature | [Emisión de la GRE Remitente](spec.md) |
| Estado | Borrador |
| Fecha | 2026-10-21 |
| ADRs aplicados | [ADR-0001](../../adr/0001-seleccion-db.md), [ADR-0002](../../adr/0002-autenticacion-jwt.md) |

## 1. Resumen de la solución

La emisión se implementa como una **tubería de seis etapas** sobre un agregado `GreDocumento`, orchestrated por un caso de uso de aplicación (`EmitirGreUseCase`). Cada etapa es una interfaz con una implementación intercambiable, lo que permite ejecutar el mismo flujo en modo *dry-run* (validación sin persistencia) sin duplicar reglas.

El JSON recibido se valida en tres capas: esquema JSON (contrato), reglas de negocio (feature 0004) y coherencia con los catálogos vigentes (feature 0003). A continuación se genera el XML UBL 2.1 mediante un **mapeador declarativo** campo a campo (tabla de correspondencia JSON ↔ tag UBL), se firma con XMLDSig y se empaqueta en ZIP. El correlativo se reserva en la **misma transacción** que persiste el documento, lo que garantiza que no existen huecos por fallo parcial.

Se eligió un modelo de **generación por plantilla con binding declarativo** en lugar de un motor de plantillas o de la construcción manual del árbol DOM, porque el estándar UBL impone un orden estricto de elementos y una cardinalidad exacta: el binding permite declarar ese orden y detectar cualquier desviación en el momento del mapeo, no en el XMLDSig final.

## 2. Contexto técnico

### 2.1 Restricciones

| ID | Restricción | Origen |
|---|---|---|
| C-1 | El orden de los elementos XML es fijo y validado por XSD. | UBL 2.1 / Anexo N.° 14 |
| C-2 | El XML debe ir en ISO-8859-1 sin BOM dentro del ZIP. | [NO VERIFICADO] en la fuente; convención SUNAT |
| C-3 | La firma XMLDSig debe cubrir exactamente el documento firmado (firma envelopada en `ext:UBLExtensions`). | Ítem 84 |
| C-4 | El nombre del archivo y el `cbc:ID` deben ser consistentes con el ZIP. | Errores 0161, 1035, 1036 |
| C-5 | La reserva de correlativo no admite huecos ni duplicados. | Regla de negocio del contribuyente |
| C-6 | El XML y el ZIP se regeneran desde la solicitud persistida, nunca desde el request original. | Trazabilidad (RNF-6) |
| C-7 | Las reglas de negocio dependen de la vigencia de los catálogos. | Feature 0003 |

### 2.2 Supuestos y dependencias

| ID | Supuesto / dependencia | Impacto si no se cumple |
|---|---|---|
| S-1 | El payload JSON es completo: el API no consulta datos maestros para completar campos. | Se requieren endpoints de consulta de maestros y reglas de completitud. |
| S-2 | La plataforma dispone del certificado digital del contribuyente para firmar (feature 0005). | No se puede emitir (no se puede enviar). |
| S-3 | El WS de SUNAT de homologación está disponible para pruebas end-to-end. | La validación end-to-end se limita al entorno de pruebas local. |
| S-4 | Las series se registran previamente en la plataforma (ver `POST /api/v1/series`). | La emisión devuelve `400` por serie inexistente. |

## 3. Arquitectura

```text
                 ┌─────────────────────────────────────────────────────────┐
  ERP / WMS ───▶ │  REST (Spring Web)      gRPC (Spring gRPC)           │
                 └───────────────┬─────────────────────┬───────────────────┘
                                 │                     │
                        ┌────────▼────────┐   ┌────────▼─────────┐
                        │ SecurityFilter  │   │ GreGrpcService   │
                        │ (JWT → RUC)     │   │ (mismo use case) │
                        └────────┬────────┘   └────────┬─────────┘
                                 └──────────┬───────────┘
                                            │
                                 ┌──────────▼───────────┐
                                 │  EmitirGreUseCase    │
                                 └───┬───┬───┬───┬───┬──┘
        ┌───────────────┬───────────────┘   │   │   │   └───────────────┐
        │               │                   │   │   └───────────────────┤
 ┌──────▼──────┐ ┌──────▼──────┐  ┌─────────▼───▼─┐ ┌───────────────▼──┐
 │ Validador   │ │ Validador   │  │ GeneradorXml │ │ Reservador       │
 │ de esquema  │ │ de negocio  │  │ UBL 2.1      │ │ de Correlativo   │
 │ (JSON)      │ │ (feature    │  │ (binding     │ │ (UPDATE RETURNING│
 │             │ │  0004)      │  │  declarativo)│ │  en transacción) │
 └─────────────┘ └─────────────┘  └───────────────┘ └──────────────────┘
                                            │
                                 ┌──────────▼───────────┐
                                 │ Firmador + Empaqueta-│  (feature 0005)
                                 │ dor ZIP              │
                                 └──────────┬───────────┘
                                            │
                                 ┌──────────▼───────────┐
                                 │ Persistencia         │  PostgreSQL (ADR-0001)
                                 │ (documento + ítems)  │
                                 └──────────────────────┘
```

| Componente | Responsabilidad | Ubicación |
|---|---|---|
| `EmitirGreUseCase` | Orquesta la tubería y define la transacción | `com.examen.gre.aplicacion.emision` |
| `ValidadorEsquemaSolicitud` | Valida el JSON contra [schema.json](../../contracts/schema.json) | `com.examen.gre.emision.validador.esquema` |
| `ValidadorReglasRemitente` | Reglas de la hoja «Guía-Remitente2_0» | `com.examen.gre.emision.validador.reglas` |
| `GeneradorXmlRemitente` | JSON → XML UBL 2.1 | `com.examen.gre.emision.ubl` |
| `MapaJsonUbl` | Tabla declarativa de correspondencia campo ↔ tag | `com.examen.gre.emision.ubl.mapeo` |
| `ReservadorCorrelativo` | Reserva atómica del correlativo | `com.examen.gre.correlativo` |
| `FirmadorXml` | Firma XMLDSig | `com.examen.gre.firma` |
| `EmpaquetadorZip` | ZIP y nomenclatura oficial | `com.examen.gre.empaquetado` |
| `GreRepositorio` | Persistencia del agregado | `com.examen.gre.persistencia` |

## 4. Decisiones de diseño

### 4.1 Mapeo declarativo JSON ↔ UBL

- **Contexto**: el XML UBL tiene ~90 puntos de mapeo con cardinalidad condicional (depende de motivo, modalidad, documentos relacionados e indicadores). Escribirlo a mano en código implica duplicar la lógica de reglas en dos lugares.
- **Decisión**: una tabla declarativa (`MapaJsonUbl`) que describe, por cada campo, su tag UBL, su namespace, su tipo, su obligatoriedad y su regla de presencia. El generador interpreta la tabla y falla rápido si el JSON no encaja.
- **Alternativas descartadas**: *motor de plantillas* (freemarker/handlebars) — no valida cardinalidad ni orden; *construcción manual del DOM* — el orden y la cardinalidad dependen del código que se escribe, con alto riesgo de desviación silenciosa.
- **Consecuencias**: el orden de los elementos y las reglas de presencia quedan en un solo artefacto revisable y versionable; añadir un campo es una entrada más en la tabla. A cambio, la tabla debe mantenerse sincronizada con el XSD (se cubre con un test que valida cada golden file contra el XSD oficial).

### 4.2 Reserva de correlativo dentro de la transacción de emisión

- **Contexto**: la numeración no admite huecos; un fallo posterior a la reserva dejaría un número quemado.
- **Decisión**: `ReservadorCorrelativo.reservar(ruc, tipo, serie)` ejecuta `UPDATE serie_correlativo SET ultimo_correlativo = ultimo_correlativo + 1 WHERE ... RETURNING ultimo_correlativo` dentro de la misma transacción que persiste el documento. La restricción `UNIQUE (ruc_emisor, tipo_documento, serie, correlativo)` es la última línea de defensa.
- **Alternativas descartadas**: *secuencia de base de datos* — no permite reiniciar la serie en un número exacto; *bloqueo pesimista explícito (`SELECT ... FOR UPDATE`)* — equivalente, pero la sentencia `UPDATE ... RETURNING` es más simple de auditar.
- **Consecuencias**: ante cualquier excepción la transacción se revierte y el correlativo queda libre. El throughput por serie se limita a una operación de base de datos; con el volumen previsto (decenas de miles de documentos al mes) es irrelevante.

### 4.3 Regeneración del XML desde la solicitud persistida

- **Contexto**: un XML firmado no puede modificarse; si hubiera que regenerarlo, la firma quedaría invalidada.
- **Decisión**: se persiste la **solicitud JSON normalizada** junto al XML firmado y su hash. El XML se regenera a partir de la solicitud persistida, nunca desde el request original.
- **Alternativas descartadas**: *persistir sólo el XML* — no permite reempaquetar ni auditar; *persistir sólo el JSON* — se perdería el XML exacto que se envió a SUNAT.
- **Consecuencias**: duplicación del almacenamiento (JSON + XML + ZIP), aceptable a este volumen. Habilita la verificación de integridad (hash) y el reenvío tras un fallo de transporte.

### 4.4 Idempotencia por clave del cliente

- **Contexto**: un ERP que reintenta tras un timeout no debe duplicar la GRE.
- **Decisión**: tabla `idempotencia` con clave única `(cliente, clave)` → hash del cuerpo normalizado + `gre_id`. Si la clave existe con el mismo hash, se devuelve la GRE existente (`200`); con otro hash, `409`.
- **Alternativas descartadas**: *deduplicación por RUC+serie+correlativo* — depende de que el cliente reporte el correlativo, que es justamente lo que no se conoce hasta después de reservar.
- **Consecuencias**: exige que el cliente envíe `Idempotency-Key`; se documenta como recomendación fuerte en el contrato y se valida su presencia con un warning si falta.

### 4.5 Firma fuera del caso de uso de emisión

- **Contexto**: la firma necesita el certificado del contribuyente, que puede no estar disponible en el momento del registro.
- **Decisión**: la emisión deja el documento en estado `VALIDADA`; la firma (feature 0005) lo lleva a `FIRMADA` y es requisito previo al envío. Si el certificado no está disponible, el documento queda registrado y se reintenta la firma más tarde.
- **Alternativas descartadas**: *firmar dentro de la misma transacción* — acopla la disponibilidad del certificado a la disponibilidad de la base de datos.
- **Consecuencias**: la emisión es rápida y disponible aunque el certificado esté temporalmente caído; el estado del documento hace explícita la situación.

## 5. Modelo de datos

### 5.1 Entidades

| Entidad | Campos | Notas |
|---|---|---|
| `GreDocumento` | id, rucEmisor, tipoDocumento, serie, correlativo, estado, fechaEmision, horaEmision, observacion, solicitudJson, xmlFirmado, hashXml, nombreArchivo, zipArchivo, creadoEn | Aggregate root. |
| `GreItem` | id, greId, orden, codigo, descripcion, cantidad, unidadMedida, codigoProductoSunat, gtin | `@OneToMany` con orden por `orden`. |
| `GrePropiedadItem` | id, greItemId, codigo, nombre, valor | Catálogo 55. |
| `GreDocumentoRelacionado` | id, greId, tipo, descripcion, numero, rucEmisor | |
| `GreEnvio` | id, greId, intento, estado, codigoRetorno, descripcionRespuesta, ticket, fechaEnvio | Un registro por intento. |
| `GreObservacion` | id, greId, item, tagUbl, codigoRetorno, tipoRetorno, mensaje | |
| `SerieCorrelativo` | rucEmisor, tipoDocumento, serie, ultimoCorrelativo, estado | |
| `Idempotencia` | cliente, clave, hashCuerpo, greId, creadoEn | Único por `(cliente, clave)`. |
| `Auditoria` | sujeto, rucEmisor, operacion, recurso, datos, fecha | |

### 5.2 Persistencia

| Tabla | Propósito | Índices |
|---|---|---|
| `gre_documento` | Cabecera y contenido de cada GRE | PK `id`; único `(ruc_emisor, tipo_documento, serie, correlativo)`; `(ruc_emisor, estado, fecha_emision DESC)` |
| `gre_item` | Bienes a trasladar | FK `gre_id`; único `(gre_id, orden)` |
| `gre_propiedad_item` | Conceptos del Catálogo 55 | FK `gre_item_id`; único `(gre_item_id, codigo, valor)` |
| `gre_documento_relacionado` | Documentos que sustentan el traslado | FK `gre_id`; único `(gre_id, tipo, numero)` |
| `gre_envio` | Intentos de envío | FK `gre_id`; `(gre_id, intento)` |
| `gre_observacion` | Observaciones de SUNAT | FK `gre_id`; `(gre_id, codigo_retorno)` |
| `serie_correlativo` | Contador por serie | PK `(ruc_emisor, tipo_documento, serie)` |
| `idempotencia` | Claves de idempotencia | PK `(cliente, clave)` |
| `auditoria` | Traza de operaciones | `(recurso, fecha)`; particionado mensual |

### 5.3 Migraciones

| Versión | Descripción | Script |
|---|---|---|
| V1 | Esquema base: GRE, ítems, propiedades, documentos relacionados, envíos, observaciones, series, idempotencia y auditoría. | `src/main/resources/db/migration/V1__gre_base.sql` |
| V2 | Row Level Security por `ruc_emisor` y función de contexto de sesión. | `src/main/resources/db/migration/V2__rls_por_ruc.sql` |
| V3 | Tablas de catálogos con vigencia (feature 0003). | `src/main/resources/db/migration/V3__catalogos.sql` |

## 6. Flujo de ejecución

### 6.1 Secuencia principal

| Paso | Actor / Componente | Acción | Resultado |
|---|---|---|---|
| 1 | `SecurityFilter` | Valida el JWT y extrae `sub`, `scope` y `ruc` | Contexto de seguridad disponible |
| 2 | `GreController` | Valida el JSON contra `schema.json` | `400` si el payload es inválido |
| 3 | `EmitirGreUseCase` | Comprueba la idempotencia | `200` con la GRE existente si la clave y el cuerpo coinciden |
| 4 | `ValidadorReglasRemitente` | Ejecuta las reglas de la feature 0004 | `422` con las violaciones `ERROR` |
| 5 | `EmitirGreUseCase` | Abre la transacción | — |
| 6 | `ReservadorCorrelativo` | `UPDATE ... RETURNING` sobre la serie | Correlativo reservado |
| 7 | `GeneradorXmlRemitente` | Mapea la solicitud al XML UBL 2.1 | Árbol XML en memoria |
| 8 | `FirmadorXml` | Firma con XMLDSig | XML firmado |
| 9 | `EmpaquetadorZip` | Arma el ZIP con la nomenclatura oficial | Bytes del ZIP |
| 10 | `GreRepositorio` | Persiste documento, ítems, documentos relacionados y auditoría | Transacción confirmada |
| 11 | `GreController` | Devuelve `201` con el identificador, el estado y el hash | — |

### 6.2 Manejo de errores

| Situación | Excepción / código | Tratamiento | Resultado observable |
|---|---|---|---|
| JSON inválido | `SolicitudInvalidaException` | Se traduce a `400` con el detalle por campo | `400` + lista de errores |
| RUC no autorizado | `RucNoAutorizadoException` | Se traduce a `403` | `403 RUC_NO_AUTORIZADO` |
| Regla de negocio incumplida | `ReglasNegocioException` | Se traduce a `422` con las violaciones | `422` + lista de violaciones |
| Serie inexistente o anulada | `SerieNoValidaException` | Se traduce a `400`/`409` | `400`/`409` |
| Conflicto de idempotencia | `IdempotenciaConflictoException` | Se traduce a `409` | `409 IDEMPOTENCIA_CONFLICTO` |
| Certificado no disponible | `CertificadoNoDisponibleException` | El documento queda en `VALIDADA`; se reintenta la firma | `201` con estado `VALIDADA` |
| Fallo de base de datos | `DataAccessException` | Rollback de la transacción completa | `500` con `traceId` |

## 7. Integraciones

| Sistema | Protocolo | Autenticación | Operaciones |
|---|---|---|---|
| SUNAT (WS de comprobantes) | SOAP 1.1 + WS-Security | Certificado digital X.509 (PKCS#12) con `UsernameToken` | `sendPack`, `sendSummary`, `getStatus` |
| PostgreSQL | JDBC | usuario/contraseña en almacén de secretos | consultas y transacciones |
| Emisor OAuth 2.0 | HTTPS | `client_credentials` / `authorization_code` + PKCE | `/oauth2/token`, `/oauth2/jwks` |

## 8. Seguridad

| Aspecto | Aplicación |
|---|---|
| Autenticación | OAuth 2.0 / JWT RS256 validado localmente ([ADR-0002](../../adr/0002-autenticacion-jwt.md)). |
| Autorización | Scope por operación + claim `ruc` por recurso; comprobación en el filtro y en el caso de uso. |
| Datos sensibles | El certificado y las credenciales del WS viven en el almacén de secretos; nunca en la base de datos de aplicación. |
| Auditoría | Se registra emisor, serie, correlativo, hash y sujeto en `auditoria`. |
| Datos personales | DNI de conductores y contactos: se cifran en reposo (AES-256) y no se exponen en las respuestas de consulta. |

## 9. Observabilidad

| Señal | Tipo | Etiquetas | Uso |
|---|---|---|---|
| `gre.emision.iniciada` | Contador | `ruc`, `tipo` | Volumen emitido |
| `gre.emision.fallida` | Contador | `ruc`, `motivo` (`esquema`, `negocio`, `tecnico`) | Tasa de rechazo local |
| `gre.emision.duracion` | Histograma | `ruc`, `items` | RNF-1 |
| `gre.correlativo.conflicto` | Contador | `ruc`, `serie` | Detección de concurrencia anómala |
| `gre.emision.log` | Log estructurado | `gre_id`, `traceId` | Trazabilidad end-to-end |

## 10. Pruebas

| Nivel | Qué cubre | Herramienta |
|---|---|---|
| Unitario | Reglas de negocio, mapeo JSON↔UBL, formato de numeración | JUnit 5 + AssertJ |
| Property-based | Round-trip JSON → XML → lectura del XML | jqwik |
| Integración | Correlativos concurrentes, RLS, idempotencia | Testcontainers (PostgreSQL) |
| Contrato | XSD UBL oficial y OpenAPI | XSD Validator + springdoc |
| End-to-end | Emisión → firma → ZIP → envío en homologación de SUNAT | WireMock + entorno de homologación |
| Golden files | XML esperado por motivo/modalidad | Ficheros en `src/test/resources/golden` |

## 11. Despliegue y operación

| Aspecto | Detalle |
|---|---|
| Variables de entorno | `DB_URL`, `DB_USER`, `DB_PASSWORD`, `SUNAT_WS_URL`, `SUNAT_CERT_ALIAS`, `SUNAT_CERT_PASSWORD`, `OAUTH_ISSUER`, `OAUTH_JWKS_URL` |
| Recursos | 2 vCPU / 2 GB por réplica; pool Hikari de 20 conexiones |
| Estrategia | Rolling update con 2 réplicas; las emisiones en curso terminan antes del corte |
| Migraciones | Flyway en el arranque; sólo se despliega una versión a la vez |
| Plan de rollback | Reversión de la imagen; las migraciones son retrocompatibles durante al menos una versión |

## 12. Cumplimiento normativo

| Requisito normativo | Dónde se implementa |
|---|---|
| UBL 2.1 `DespatchAdvice`, `CustomizationID` 2.0 | `GeneradorXmlRemitente` + `MapaJsonUbl` |
| Formato de serie-correlativo `[T][A-Z0-9]{3}-[0-9]{1,8}` | `NumeracionHelper` |
| Catálogos 01, 06, 13, 18, 20, 25, 55, 61, 62, 63, 64, 65, D-37 | Feature 0003 (consulta en tiempo de validación) |
| Firma digital XMLDSig | `FirmadorXml` (feature 0005) |
| ZIP con nomenclatura `RRRRRRRRRRR-09-SSSS-NNNNNNNN` | `EmpaquetadorZip` (feature 0005) |
| Reglas de los ítems 1..84 | Feature 0004 (`ValidadorReglasRemitente`) |

## 13. Trazabilidad con la spec

| Requisito | Diseño | Prueba |
|---|---|---|
| RF-1 | `ValidadorEsquemaSolicitud` | `SolicitudEmisionTest` |
| RF-2 | `ReservadorCorrelativo` | `EmisionConcurrenteIT` |
| RF-3 | `ValidadorReglasRemitente` | `ValidacionRemitenteTest` |
| RF-4 | `GeneradorXmlRemitente` + `MapaJsonUbl` | `XmlRemitenteTest` (golden files) |
| RF-5 | `FirmadorXml`, `EmpaquetadorZip` | `FirmaYZipTest` |
| RF-6 | `GreController`, `GreRepositorio` | `GreControllerIT` |
| RF-7 | `EnvioRepository` | `EnvioIT` |
| RF-8 | `ClienteSunat` (feature 0005) | `EnvioResilienteIT` |
| RF-9 | `GreGrpcService` | `GreGrpcServiceIT` |
| RF-10 | `AuditoriaAspecto` | `AuditoriaTest` |

## 14. Trabajo pendiente / decisiones diferidas

- Definir el formato del golden file de XML (una línea por tag o árbol indentado) y su proceso de actualización cuando cambie el XLSX de SUNAT.
- Evaluar la emisión en lote (múltiples GRE por petición) cuando el volumen de un mismo ERP lo justifique.
- Decidir si el PDF de la GRE se genera en el servicio o se delega al cliente (fuera de alcance actual).
