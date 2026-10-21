# 0001 — Emisión de la GRE Remitente

| Campo | Valor |
|---|---|
| Código | 0001 |
| Módulo | Emisión de comprobantes de traslado (GRE Remitente) |
| Estado | Borrador |
| Versión | 1.0.0 |
| Fecha | 2026-10-21 |
| Responsables | Equipo de la plataforma GRE |
| Depende de | 0003 (catálogos), 0004 (validación), 0005 (firma y envío) |
| Bloquea a | 0002 (GRE Transportista) |
| ADRs aplicables | [ADR-0001](../../adr/0001-seleccion-db.md), [ADR-0002](../../adr/0002-autenticacion-jwt.md) |
| Contrato XML | [contrato-xml-ubl.md](contrato-xml-ubl.md) |

## 1. Objetivo

Permitir que un contribuyente (o su ERP) registre una **Guía de Remisión Electrónica Remitente** (GRE-R, tipo de documento `09`), generándola desde una representación JSON, validándola contra las reglas de la SUNAT, firmándola digitalmente y dejándola lista para el envío al Web Service.

La GRE-R documenta el traslado de bienes **desde** el remitente **hacia** el destinatario. El estándar aplicado es UBL 2.1 `DespatchAdvice` con `CustomizationID` 2.0, según la R.S. 123-2022-SUNAT, Anexo N.° 14.

## 2. Alcance

### 2.1 Incluido

- Registro de la GRE-R desde JSON ([schema.json](../../contracts/schema.json)) con reserva transaccional del correlativo.
- Validación de estructura, reglas de negocio por motivo de traslado, modalidad de transporte y catálogos vigentes.
- Generación del XML UBL 2.1 (`DespatchAdvice`) con la nomenclatura de archivo oficial.
- Firma digital XMLDSig y empaquetado ZIP (delegados a la feature 0005).
- Consulta del detalle, del XML, del ZIP, del CDR e historial de envíos.
- Emisión desde gRPC, con el mismo modelo de dominio que REST.

### 2.2 Excluido

- La **GRE Transportista** (`31`), que se trata en la [feature 0002](../0002-emision-gre-transportista/spec.md).
- La **comunicación de eventos del traslado** (tipo de evento `1`, `2`, `3`): la propia norma indica que **no debe enviarse** en la GRE.
- La **anulación** del documento y el **baja por cambio de destinatario** (procedimiento administrativo de SUNAT).
- La generación de otros comprobantes electrónicos (factura, boleta, GRE-Transportista).
- La concILIación con el DAM/DS y el Manifiesto de Carga: se valida la consistencia interna de los datos declarados, no se consultan esos sistemas.

## 3. Antecedentes y contexto

La R.S. 123-2022-SUNAT (Anexo N.° 14) establece el estándar UBL 2.1 de la Guía de remisión remitente/transportista y obliga a que el documento se firme digitalmente y se envíe en un ZIP cuyo nombre coincida con el RUC emisor, el tipo de documento, la serie y el correlativo.

La hoja de validación «Guía-Remitente2_0» del XLSX oficial (`ValidacionesGREv20250421_0.xlsx`) define **84 ítems** de control con sus códigos de retorno; cada ítem está reflejado en [contrato-xml-ubl.md](contrato-xml-ubl.md) con su tag UBL, su obligatoriedad, su formato y sus códigos. Ese documento es la referencia normativa de esta feature.

Situación previa: el proceso es manual (generación del XML con una herramienta externa, firma, carga al WS y seguimiento de la respuesta por correo), lo que genera errores de nomenclatura, duplicados de correlativo y pérdida de trazabilidad.

## 4. Glosario

| Término | Definición |
|---|---|
| GRE-R | Guía de Remisión Electrónica Remitente; tipo de documento `09`. |
| Serie | Código de 4 caracteres que identifica la secuencia (`T###`). |
| Correlativo | Número consecutive dentro de la serie (1 a 99 999 999). |
| Identificador (`cbc:ID`) | `T###-########`, debe coincidir con el nombre del archivo. |
| Motivo de traslado | Catálogo 20; determina qué datos son obligatorios. |
| Modalidad de traslado | Catálogo 18: `01` público, `02` privado. |
| Indicador (`SUNAT_Envio_*`) | Valor en `cbc:SpecialInstructions` que activa reglas del estándar. |
| Ítem de validación | Fila numerada (1..84) de la hoja de validación de SUNAT. |
| Código de retorno | Número (1001, 4383, …) con el que SUNAT responde por una regla incumplida. |
| Ticket | Número de Constancia de Recepción (CDR) devuelto por SUNAT. |

## 5. Personas y permisos

| Actor | Descripción | Permisos |
|---|---|---|
| ERP / WMS | Sistema del contribuyente que origina los traslados. | `gre.emitir`, `gre.consultar`, `gre.enviar` sobre los RUC de su token. |
| Usuario del contribuyente | Analista que revisa y corrige GRE. | `gre.consultar`, `gre.enviar`. |
| Operador de la plataforma | Personal de soporte. | `admin` (consulta global, sin emitir en nombre del contribuyente). |
| Servicio | Proceso interno. | Emisión y envío con la credencial del RUC. |

## 6. Historias de usuario

### US-1 — Registrar una GRE-R

**Como** ERP del contribuyente
**Quiero** registrar una GRE-R con los datos del traslado
**Para** que el traslado quede documentado ante la SUNAT sin intervención manual

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given un payload válido, When se invoca `POST /api/v1/guias`, Then se devuelve `201` con el identificador `T###-########`, el XML generado y su hash SHA-256 | Must |
| 2 | Given un payload con un RUC no habilitado en el token, When se invoca la emisión, Then se devuelve `403 RUC_NO_AUTORIZADO` | Must |
| 3 | Given un payload que ya fue emitido con la misma `Idempotency-Key` y el mismo cuerpo, When se reintenta, Then se devuelve `200` con la GRE original | Must |
| 4 | Given dos emisiones concurrentes de la misma serie, When se procesan, Then los correlativos son consecutivos y sin duplicados | Must |
| 5 | Given un motivo de traslado que exige datos adicionales ausentes, When se emite, Then se devuelve `422` con el ítem de validación y el código de retorno de SUNAT | Must |
| 6 | Given una GRE emitida, When se consulta el detalle, Then se obtienen el XML, el estado, el hash y el historial de envíos | Should |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-1 | La GRE-R se identifica con el tipo de documento `09` y su serie empieza por `T`. | Contrato XML, ítem 6 |
| RN-2 | El identificador sigue el formato `[T][A-Z0-9]{3}-[0-9]{1,8}` y su serie y correlativo deben coincidir con los del nombre del archivo. | Ítem 3 (códigos 1035/1036) |
| RN-3 | `cbc:UBLVersionID` = `2.1` y `cbc:CustomizationID` = `2.0`. | Ítems 1 y 2 (2110-2113) |
| RN-4 | La fecha de emisión no puede ser futura ni diferir en más de 1 día de la fecha de recepción. | Ítem 4 (2108, 2329) |
| RN-5 | El RUC del remitente va en `cbc:ID[@schemeID='6']` con los atributos `schemeName`, `schemeAgencyName` y `schemeURI` del Catálogo 06. | Ítem 8 (2511, 4255-4257) |
| RN-6 | El motivo de traslado (Catálogo 20) determina la obligatoriedad de proveedor, comprador, pesos, bultos, contenedores y puerto/aeropuerto. | Ítems 17-20, 22-31 |
| RN-7 | En modalidad `01` (público) el transportista es obligatorio y debe ser distinto del remitente; en modalidad `02` (privado) el transportista no debe existir. | Ítems 32, 43-46 |
| RN-8 | El peso bruto (`cbc:GrossWeightMeasure`) es obligatorio con unidad `KGM` o `TNE`. | Ítem 26 (2880, 2523) |
| RN-9 | Los motivos `08`, `09` y `19` exigen peso neto y sustento de la diferencia de peso, salvo el indicador `SUNAT_Envio_IndicadorTrasladoTotal`. | Ítems 24 y 25 (4383, 3395, 4387, 3418) |
| RN-10 | Se admiten como máximo 2 contenedores y 2 precintos, sin repetir. | Ítems 28-31 (3420, 3422, 3423) |
| RN-11 | El conductor principal (`cbc:JobTitle` = `Principal`) es único; se admiten hasta 2 conductores secundarios. | Ítems 53-60 (3357, 3358) |
| RN-12 | El documento relacionado tipo `91` (Manifiesto de Carga) exige que las líneas declaren los conceptos `7024` a `7028` y que concilien con contenedores y precintos. | Ítems 12, 13, 81-90 |
| RN-13 | La firma digital XMLDSig es obligatoria. | Ítem 84 (1059, 2336) |
| RN-14 | El ZIP contiene un único comprobante y su nombre coincide con el del XML. | Errores 0158, 0161 |

**Validaciones / errores esperados**

| Código | Condición | Tipo | Comportamiento |
|---|---|---|---|
| `2108` | Fecha de emisión fuera de la ventana de 1 día. | ERROR | Se rechaza en validación local; no se envía. |
| `2110`/`2112` | Versión UBL o `CustomizationID` distintos de 2.1 / 2.0. | ERROR | El generador los emite siempre correctos; se defienden en el validador. |
| `1035`/`1036` | La serie o el correlativo del `cbc:ID` no coinciden con el nombre del archivo. | ERROR | El generador los construye desde la misma fuente. |
| `3404`/`3405` | Motivo de traslado vacío o fuera del Catálogo 20. | ERROR/OBSERV | `422` con detalle de catálogo. |
| `4383` | Falta el peso neto en los motivos `08`/`09`/`19`. | ERROR | `422`, ítem 24. |
| `3447` | Falta el proveedor en los motivos `02`/`07`. | ERROR | `422`, ítem 17. |
| `3331` | Falta el comprador en el motivo `03`. | ERROR | `422`, ítem 19. |
| `2561` | Falta el transportista en modalidad `01`. | ERROR | `422`, ítem 43. |
| `2532` | Modalidad de traslado fuera del Catálogo 18. | ERROR | `422`, ítem 32. |
| `3446` | Unidad de medida no permitida para el tipo de documento relacionado. | ERROR/OBSERV | `422`, ítem 76. |
| `3372` | Falta el código de producto SUNAT con bien normalizado. | ERROR | `422`, ítem 79. |
| `1059` | Falta la firma digital. | ERROR | Se rechaza el envío; el documento no pasa a `FIRMADA`. |
| `0154` | El RUC del archivo no corresponde al RUC autorizado del usuario. | ERROR | `422` en el envío; la GRE queda `RECHAZADA`. |
| `0161` | El nombre del XML no coincide con el del ZIP. | ERROR | Se rehace el empaquetado; el error es del generador. |
| `0306` | El XML no se puede parsear. | ERROR | Se regenera desde la fuente persistida. |

### US-2 — Consultar el estado de una GRE

**Como** usuario del contribuyente
**Quiero** consultar una GRE emitida, su XML y su historial de envíos
**Para** responder a un reclamo o a una fiscalización sin volver al ERP

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given una GRE emitida, When se solicita su detalle, Then se devuelve el estado, el hash, el XML y los intentos de envío | Must |
| 2 | Given una GRE con observaciones, When se consulta el detalle, Then se listan las observaciones con su código de retorno y su tipo (`ERROR`/`OBSERV`/`REST`) | Must |
| 3 | Given una GRE de otro RUC, When se solicita su detalle, Then se devuelve `404` (no se revela su existencia) | Must |
| 4 | Given una GRE aún no enviada, When se solicita el CDR, Then se devuelve `404` con el estado actual | Should |

### US-3 — Validar antes de emitir

**Como** ERP del contribuyente
**Quiero** validar el payload sin persistirlo
**Para** corregir los datos en origen y no acumular documentos rechazados

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given un payload con un error de negocio, When se invoca la validación, Then se devuelve la lista de violaciones con ítem, tag UBL, código de retorno y mensaje | Must |
| 2 | Given la misma operación, When se valida dos veces, Then no se crea ningún documento ni correlativo | Must |
| 3 | Given un payload válido, When se valida, Then `valido = true` y la lista de violaciones vacía | Must |

## 7. Flujo principal

1. El ERP obtiene un token del emisor OAuth 2.0 (ver [ADR-0002](../../adr/0002-autenticacion-jwt.md)).
2. El ERP envía la solicitud JSON con la cabecera `Idempotency-Key`.
3. El servicio valida el payload contra el esquema (400 si no cumple el contrato).
4. El servicio verifica que el RUC del payload figure en el claim `ruc` del token (403 si no).
5. El servicio carga los catálogos vigentes y ejecuta las reglas de negocio de la feature 0004 (422 si hay violaciones `ERROR`).
6. El servicio reserva el correlativo de la serie dentro de la misma transacción (400/409 si la serie no existe o está anulada).
7. El servicio genera el XML UBL 2.1 a partir de la solicitud validada.
8. El servicio firma el XML y arma el ZIP (feature 0005).
9. El servicio persiste el documento, sus ítems y sus documentos relacionados, y devuelve `201` con el identificador, el estado `FIRMADA` y el hash.
10. El envío a SUNAT es un paso posterior y explícito (`POST /api/v1/guias/{id}/envios`).

## 8. Flujos alternativos y de error

| ID | Situación | Disparo | Respuesta del sistema | Estado resultante |
|---|---|---|---|---|
| FA-1 | Payload inválido contra el esquema | JSON malformado o tipo/rango incorrecto | `400` con el detalle de cada campo | — (no se persiste) |
| FA-2 | RUC no habilitado | `rucEmisor` ausente del claim `ruc` | `403 RUC_NO_AUTORIZADO` | — |
| FA-3 | Regla de negocio incumplida | Motivo/modalidad/documentos relacionados incompletos | `422` con la lista de violaciones | — |
| FA-4 | Serie inexistente o anulada | `serie` no registrada para el RUC | `400`/`409` con el detalle de la serie | — |
| FA-5 | Conflicto de idempotencia | Misma clave, cuerpo distinto | `409 IDEMPOTENCIA_CONFLICTO` | — |
| FA-6 | Error de base de datos durante la emisión | Falla la transacción | `500`, se revierte la transacción completa (incluida la reserva de correlativo) | — |
| FA-7 | SUNAT rechaza el documento | El WS responde con un `ERROR` | La GRE queda `RECHAZADA` con el código de retorno; no se reintenta | `RECHAZADA` |
| FA-8 | SUNAT responde con observaciones | El WS responde con `OBSERV` | La GRE queda `ACEPTADA_CON_OBSERVACIONES` con la observación persistida | `ACEPTADA_CON_OBSERVACIONES` |
| FA-9 | Error de transporte en el envío | Timeout o 5xx | Reintento con backoff exponencial; al agotar los intentos, `PENDIENTE_REINTENTO` | `PENDIENTE_REINTENTO` |

## 9. Requisitos funcionales

| ID | Requisito | Origen |
|---|---|---|
| RF-1 | Registrar la GRE-R desde JSON con validación de esquema | US-1 |
| RF-2 | Reservar el correlativo de forma transaccional y sin duplicados | US-1, RN-2 |
| RF-3 | Validar las reglas de negocio de la hoja «Guía-Remitente2_0» | US-1, US-3 |
| RF-4 | Generar el XML UBL 2.1 conforme al estándar `DespatchAdvice` | US-1, RN-1..RN-3 |
| RF-5 | Firmar el XML y empaquetarlo en el ZIP con la nomenclatura oficial | US-1, RN-13, RN-14 |
| RF-6 | Persistir el documento y exponerlo en consulta y descarga | US-2 |
| RF-7 | Exponer el historial de envíos con sus códigos de retorno | US-2 |
| RF-8 | Permitir el envío explícito a SUNAT con reintentos clasificados | FA-7..FA-9 |
| RF-9 | Exponer la misma operación por gRPC | Contrato gRPC |
| RF-10 | Registrar auditoría de toda la operación | RNF-4 |

## 10. Requisitos no funcionales

| ID | Categoría | Requisito | Métrica |
|---|---|---|---|
| RNF-1 | Rendimiento | Emisión (validación + XML + firma) sin envío | p95 < 1,5 s con 2 000 ítems |
| RNF-2 | Rendimiento | Validación sin persistencia | p95 < 500 ms |
| RNF-3 | Disponibilidad | API de emisión | 99,5 % mensual |
| RNF-4 | Trazabilidad | Cada emisión queda auditada con sujeto, RUC, serie, correlativo y hash | 100 % de las emisiones |
| RNF-5 | Seguridad | El RUC efectivo proviene del token | 0 accesos con RUC no autorizado |
| RNF-6 | Integridad | El XML persistido conserva su hash SHA-256 | Verificación en cada descarga |
| RNF-7 | Compatibilidad | El XML generado es aceptado por el validador oficial de SUNAT | 100 % de los casos de prueba |
| RNF-8 | Idempotencia | Repetir la misma `Idempotency-Key` no crea un segundo documento | 0 duplicados |

## 11. Modelo de datos

| Entidad | Campos principales | Notas |
|---|---|---|
| `GreDocumento` | id, ruc_emisor, tipo_documento, serie, correlativo, estado, fecha_emision, hora_emision, hash_xml, xml_firmado (BYTEA), nombre_archivo | Único por `(ruc_emisor, tipo_documento, serie, correlativo)`. |
| `SerieCorrelativo` | ruc_emisor, tipo_documento, serie, ultimo_correlativo, estado | Reserva con `UPDATE ... RETURNING` (ver [ADR-0001](../../adr/0001-seleccion-db.md)). |
| `GreItem` | gre_id, orden, descripcion, cantidad, unidad_medida, codigo_producto_sunat, gtin | Único por `(gre_id, orden)`. |
| `GrePropiedadItem` | gre_item_id, codigo (Catálogo 55), nombre, valor | Único por `(gre_item_id, codigo, valor)`. |
| `GreDocumentoRelacionado` | gre_id, tipo, descripcion, numero, ruc_emisor | Único por `(gre_id, tipo, numero)`. |
| `GreEnvio` | gre_id, intento, estado, codigo_retorno, descripcion_respuesta, ticket, fecha | Un registro por intento. |
| `GreObservacion` | gre_id, item, tag_ubl, codigo_retorno, tipo_retorno, mensaje | Violaciones con `OBSERV` o `ERROR` de SUNAT. |
| `Auditoria` | sujeto, ruc_emisor, operacion, recurso, fecha, detalle | Append-only. |

## 12. Contratos

| Contrato | Ubicación | Descripción |
|---|---|---|
| OpenAPI | [openapi.yaml](../../contracts/openapi.yaml) | Superficie REST de emisión, consulta, envío y catálogos. |
| JSON Schema | [schema.json](../../contracts/schema.json) | Contrato de entrada del payload de emisión y sus reglas condicionales. |
| gRPC | [grpc_service.proto](../../contracts/grpc_service.proto) | Superficie gRPC equivalente. |
| XML UBL | [contrato-xml-ubl.md](contrato-xml-ubl.md) | Correspondencia campo a campo entre el JSON y el XML UBL 2.1, con los 84 ítems de validación. |

## 13. Criterios de aceptación de la feature

- [ ] Un payload de venta (motivo `01`, modalidad `02`) se convierte en un XML UBL 2.1 que el validador oficial de SUNAT acepta sin observaciones.
- [ ] Los motivos `08`, `09` y `19` exigen peso neto y sustento; sin ellos, la emisión se rechaza con `422` y el ítem 24.
- [ ] Los motivos `02`, `07` y `13` exigen proveedor; el motivo `03` y el `13` exigen comprador.
- [ ] La modalidad `01` exige transportista y la `02` lo prohíbe; el driver de la regla detecta la presencia indebida.
- [ ] 50 emisiones concurrentes sobre la misma serie producen 50 correlativos consecutivos sin huecos.
- [ ] Repetir la `Idempotency-Key` no crea un segundo documento.
- [ ] Un token sin el RUC del payload recibe `403` y no se persiste nada.
- [ ] El XML descargado conserva el hash informado en la emisión.

## 14. Trazabilidad

| Requisito | Artefacto | Prueba |
|---|---|---|
| RF-1 | `SolicitudEmision` (schema.json) | `SolicitudEmisionTest` — validación de esquema property-based |
| RF-2 | `SerieCorrelativoRepository` | `EmisionConcurrenteIT` — 50 hilos, misma serie |
| RF-3 | `ValidacionGreRemitente` | `ValidacionRemitenteTest` — tabla de ítems 1..84 |
| RF-4 | `GeneradorXmlRemitente` | `XmlRemitenteTest` — golden files contra XSD UBL |
| RF-5 | `FirmadorXml`, `EmpaquetadorZip` | `FirmaYZipTest` — firma verificable y nombre de archivo |
| RF-6 | `GreController` | `GreControllerIT` — consulta, descarga y 404 cruzado |
| RF-7 | `EnvioRepository` | `EnvioIT` — historial con códigos de retorno |
| RF-8 | `ClienteSunat` (feature 0005) | `EnvioResilienteIT` — backoff y clasificación de errores |
| RF-9 | `GreGrpcService` | `GreGrpcServiceIT` — paridad con REST |
| RF-10 | `AuditoriaAspecto` | `AuditoriaTest` — registro por operación |

## 15. Riesgos y supuestos

| ID | Riesgo / supuesto | Impacto | Mitigación |
|---|---|---|---|
| R-1 | La SUNAT modifica el XLSX de validaciones sin aviso. | Alto | Importación versionada de catálogos (feature 0003) y pruebas de regresión sobre el XLSX vigente. |
| R-2 | Divergencia entre el JSON del API y el XML que exige SUNAT. | Alto | El XML se valida contra el XSD oficial en CI; los golden files se regeneran con el XLSX vigente. |
| R-3 | Duplicidad de correlativos bajo concurrencia. | Alto | Reserva transaccional con bloqueo de fila y restricción `UNIQUE`. |
| R-4 | Los ítems `[NO VERIFICADO]` del contrato XML se interpretan de forma distinta a SUNAT. | Medio | Validar esos puntos contra el XLSX antes de la salida a producción. |
| R-5 | El cliente no dispone de certificado digital válido. | Medio | Diagnóstico temprano en el pre-chequeo de la feature 0005. |
| S-1 | Se asume que el ERP del cliente puede serializar la cantidad con 10 decimales (`n(12,10)`). | — | El generador redondea y ajusta; se documenta la restricción. |

## 16. Preguntas abiertas

- ¿La numeración de series se valida contra el catálogo de series habilitadas de SUNAT o contra el registro interno de la plataforma? (responsable: negocio)
- ¿Se admite emisión en lote (múltiples GRE en una sola petición)? Queda fuera de alcance hasta que el volumen lo justifique.
