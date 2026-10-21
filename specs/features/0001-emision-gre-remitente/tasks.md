# 0001 — Tareas: Emisión de la GRE Remitente

| Campo | Valor |
|---|---|
| Código | 0001 |
| Feature | [Emisión de la GRE Remitente](spec.md) |
| Diseño | [design.md](design.md) |
| Estado | Pendiente |

## Convenciones

- Identificadores `T-<bloque>-<secuencia>`; estables y no reutilizables.
- Cada tarea declara el requisito de origen (`RF-…`) y la prueba que la cierra.
- Estimaciones en puntos de historia.
- Estado: `[ ]` pendiente · `[~]` en curso · `[x]` completada · `[-]` descartada.

## Fase 0 — Fundaciones

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-00-01 | Configurar el proyecto Maven (`com.examen.gre`), Spring Boot 3, Lombok y los perfiles `dev`/`test`/`prod` | — | 2 | — | [ ] |
| T-00-02 | Configurar Spring Security como resource server JWT (RS256) con JWKS remoto y validación de `scope` y claim `ruc` | RF-10 | 3 | T-00-01 | [ ] |
| T-00-03 | Configurar PostgreSQL, HikariCP y Flyway con los perfiles de datos de prueba | RF-2 | 2 | T-00-01 | [ ] |
| T-00-04 | Crear el módulo `emision` con sus paquetes `aplicacion`, `dominio`, `infraestructura` y `interfaz` | — | 1 | T-00-01 | [ ] |
| T-00-05 | Definir el contrato de errores `application/problem+json` (RFC 9457) y el manejador global | RF-1 | 2 | T-00-01 | [ ] |
| T-00-06 | Implementar el aspecto de auditoría (sujeto, RUC, operación, recurso) | RF-10 | 2 | T-00-04 | [ ] |
| T-00-07 | Publicar la especificación OpenAPI desde el código y verificar que coincide con `contracts/openapi.yaml` | — | 2 | T-00-05 | [ ] |

## Fase 1 — Dominio y contratos

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-10-01 | Modelar el agregado `GreDocumento` con su máquina de estados (BORRADOR → VALIDADA → FIRMADA → ENVIADA → ACEPTADA/RECHAZADA) | RF-6 | 3 | T-00-04 | [ ] |
| T-10-02 | Implementar `ValidadorEsquemaSolicitud` sobre `schema.json` con mensajes por campo | RF-1 | 3 | T-00-04 | [ ] |
| T-10-03 | Implementar `NumeracionHelper`: formato `[T][A-Z0-9]{3}-[0-9]{1,8}` y nomenclatura de archivo `RRRRRRRRRRR-09-SSSS-NNNNNNNN` | RF-4 | 1 | T-10-01 | [ ] |
| T-10-04 | Definir la tabla declarativa `MapaJsonUbl` (tag, namespace, tipo, obligatoriedad, regla de presencia) | RF-4 | 5 | T-10-01 | [ ] |
| T-10-05 | Implementar `GeneradorXmlRemitente` a partir de `MapaJsonUbl`, con orden fijo de elementos UBL | RF-4 | 5 | T-10-04 | [ ] |
| T-10-06 | Implementar los catálogos de atributos `list*`/`scheme*` (PE:SUNAT, PE:INEI, UNSPSC, listURI) | RF-4 | 2 | T-10-05 | [ ] |
| T-10-07 | Implementar la idempotencia por `Idempotency-Key` (hash del cuerpo normalizado) | RF-1 | 3 | T-10-01 | [ ] |
| T-10-08 | Validar el XML generado contra el XSD oficial de UBL 2.1 en el pipeline de build | RNF-7 | 2 | T-10-05 | [ ] |

## Fase 2 — Persistencia

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-20-01 | Migración V1: tablas `gre_documento`, `gre_item`, `gre_propiedad_item`, `gre_documento_relacionado`, `gre_envio`, `gre_observacion` | RF-6 | 3 | T-00-03 | [ ] |
| T-20-02 | Migración V2: Row Level Security por `ruc_emisor` y función de contexto de sesión | RNF-5 | 3 | T-20-01 | [ ] |
| T-20-03 | Migración V3: tablas de catálogos con vigencia | RF-3 | 2 | T-20-01 | [ ] |
| T-20-04 | `SerieCorrelativoRepository` con reserva atómica (`UPDATE ... RETURNING`) y restricción `UNIQUE` | RF-2 | 3 | T-20-01 | [ ] |
| T-20-05 | `GreRepositorio` con guardado del agregado completo (cabecera, ítems, propiedades, documentos relacionados) en una transacción | RF-6 | 3 | T-20-01 | [ ] |
| T-20-06 | Tabla `idempotencia` con clave única `(cliente, clave)` | RF-1 | 1 | T-20-01 | [ ] |
| T-20-07 | Repositorio de envíos y de observaciones con sus índices | RF-7 | 2 | T-20-01 | [ ] |

## Fase 3 — Integración

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-30-01 | Integrar `ValidadorReglasRemitente` (feature 0004) como puerto del caso de uso | RF-3 | 3 | T-10-01, T-20-05 | [ ] |
| T-30-02 | Integrar la consulta de catálogos vigentes (feature 0003) en la fase de validación | RF-3 | 2 | T-30-01 | [ ] |
| T-30-03 | Integrar `FirmadorXml` (feature 0005) y dejar el documento en `FIRMADA` | RF-5 | 3 | T-10-05 | [ ] |
| T-30-04 | Integrar `EmpaquetadorZip` con la nomenclatura oficial y verificación nombre ↔ `cbc:ID` | RF-5 | 2 | T-30-03 | [ ] |
| T-30-05 | Orquestar `EmitirGreUseCase` con la transacción única (reserva + persistencia) | RF-2, RF-4 | 5 | T-20-04, T-20-05, T-30-03, T-30-04 | [ ] |
| T-30-06 | Cliente SOAP de SUNAT (`sendPack`, `getStatus`) con WS-Security y lectura del CDR | RF-8 | 5 | T-30-05 | [ ] |
| T-30-07 | Clasificador de errores de SUNAT y política de reintento con backoff exponencial | RF-8 | 3 | T-30-06 | [ ] |

## Fase 4 — Endpoints y experiencia de uso

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-40-01 | `POST /api/v1/guias` con idempotencia y traducción de errores | RF-1 | 3 | T-30-05 | [ ] |
| T-40-02 | `GET /api/v1/guias` con filtros y paginación | RF-6 | 2 | T-40-01 | [ ] |
| T-40-03 | `GET /api/v1/guias/{id}` con verificación de pertenencia al RUC del token | RF-6, RNF-5 | 2 | T-40-02 | [ ] |
| T-40-04 | `GET /api/v1/guias/{id}/xml` y `/zip` con verificación del hash | RF-6, RNF-6 | 2 | T-40-03 | [ ] |
| T-40-05 | `POST /api/v1/guias/{id}/envios` y `GET /api/v1/guias/{id}/envios` | RF-7, RF-8 | 3 | T-30-07 | [ ] |
| T-40-06 | `POST /api/v1/guias/{id}/validacion` (dry-run sin persistencia) | RF-3 | 2 | T-30-01 | [ ] |
| T-40-07 | `GET /api/v1/guias/{id}/cdr` | RF-7 | 1 | T-40-05 | [ ] |
| T-40-08 | `POST /api/v1/series` y `GET /api/v1/series` | RF-2 | 2 | T-20-04 | [ ] |
| T-40-09 | `GreGrpcService` con paridad de comportamiento respecto a REST | RF-9 | 5 | T-40-01 | [ ] |

## Fase 5 — Observabilidad, pruebas y documentación

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-50-01 | Métricas de emisión (contadores, histograma de duración) y logs estructurados | RNF-1 | 2 | T-40-01 | [ ] |
| T-50-02 | Golden files del XML por motivo y modalidad, regenerables desde el XLSX | RNF-7 | 3 | T-10-05 | [ ] |
| T-50-03 | Prueba de concurrencia: 50 emisiones simultáneas sobre la misma serie | RF-2 | 3 | T-30-05 | [ ] |
| T-50-04 | Prueba de aislamiento multi-RUC (RLS + filtro de token) | RNF-5 | 2 | T-40-03 | [ ] |
| T-50-05 | Prueba de idempotencia (misma clave / mismo cuerpo y clave distinta / cuerpo distinto) | RF-1 | 2 | T-40-01 | [ ] |
| T-50-06 | Prueba end-to-end contra el entorno de homologación de SUNAT | RNF-7 | 3 | T-40-05 | [ ] |
| T-50-07 | Documentación de uso: ejemplo de request/response, errores frecuentes y guía de integración | — | 2 | T-40-09 | [ ] |
| T-50-08 | Panel básico de observabilidad con las métricas de la feature | RNF-3 | 2 | T-50-01 | [ ] |

## Resumen

| Fase | Tareas | Puntos | Completadas |
|---|---|---|---|
| 0 — Fundaciones | 7 | 14 | 0 |
| 1 — Dominio y contratos | 8 | 22 | 0 |
| 2 — Persistencia | 7 | 17 | 0 |
| 3 — Integración | 7 | 24 | 0 |
| 4 — Endpoints | 9 | 22 | 0 |
| 5 — Observabilidad, pruebas y documentación | 8 | 19 | 0 |
| **Total** | **46** | **118** | **0** |

## Definición de terminado (DoD)

- [ ] Todas las tareas están en `[x]` o `[-]`.
- [ ] Cada requisito `RF-…` de la spec tiene al menos una prueba que lo cubre.
- [ ] El XML generado es aceptado por el validador oficial de SUNAT en todos los casos de prueba.
- [ ] `contracts/openapi.yaml` y `contracts/schema.json` reflejan la API implementada.
- [ ] Métricas, logs y alertas definidos en el diseño están implementados.
- [ ] La documentación de uso está actualizada con ejemplos de request/response.

## Fuera de alcance

- Emisión de la GRE Transportista (feature 0002).
- Anulación y baja de documentos.
- Emisión en lote.
- Generación del PDF de la GRE.
