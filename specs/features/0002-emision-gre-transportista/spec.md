# 0002 — Emisión de la GRE Transportista

| Campo | Valor |
|---|---|
| Código | 0002 |
| Módulo | Emisión de comprobantes de traslado (GRE Transportista) |
| Estado | Borrador |
| Versión | 1.0.0 |
| Fecha | 2026-10-21 |
| Responsables | Equipo de la plataforma GRE |
| Depende de | 0001 (tubería de emisión), 0003 (catálogos), 0004 (validación), 0005 (firma y envío) |
| Bloquea a | — |
| ADRs aplicables | [ADR-0001](../../adr/0001-seleccion-db.md), [ADR-0002](../../adr/0002-autenticacion-jwt.md) |
| Contrato XML | [contrato-xml-ubl.md](contrato-xml-ubl.md) |

## 1. Objetivo

Permitir que un **transportista** (o su ERP/TMS) registre una **Guía de Remisión Electrónica Transportista** (GRE-T, tipo de documento `31`), generándola desde una representación JSON, validándola contra las reglas de la SUNAT, firmándola digitalmente con el certificado del transportista y dejándola lista para el envío al Web Service.

La GRE-T documenta el traslado de bienes **desde** el remitente **hacia** el destinatario, en nombre del transportista que ejecuta el servicio. El estándar aplicado es UBL 2.1 `DespatchAdvice` con `CustomizationID` 2.0, según la R.S. 123-2022-SUNAT, Anexo VII b) y N.° 14.

A diferencia de la [GRE Remitente](../0001-emision-gre-remitente/spec.md), en la GRE-T el emisor del documento (`cac:DespatchSupplierParty`) es el propio transportista, el remitente es un tercero que viaja en `cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty`, y aparecen dos bloques condicionales propios del estándar Transportista: la **subcontratación del servicio** (`cac:Consignment`) y el **pagador del flete** (`cac:OriginatorCustomerParty`).

## 2. Alcance

### 2.1 Incluido

- Registro de la GRE-T desde JSON ([schema.json](../../contracts/schema.json), contrato `SolicitudEmision` con `tipoDocumento: "31"` y serie `V###`) con reserva transaccional del correlativo.
- Validación de estructura y de las reglas de negocio de la hoja «Guía-Transportista2_0» (64 campos de datos, 178 códigos de retorno), propia de la feature 0004.
- Generación del XML UBL 2.1 (`DespatchAdvice`) con `DespatchAdviceTypeCode` = `31` y la nomenclatura de archivo oficial `RUC-31-VAAA-CCCCCCCC-XML`.
- Declaración de la **subcontratación** del servicio de transporte (indicador y bloque `cac:Consignment`).
- Declaración del **pagador del flete** (remitente, subcontratador o tercero) con el bloque `cac:OriginatorCustomerParty`.
- Tratamiento de los indicadores `SUNAT_Envio_*` propios del estándar Transportista, sin duplicados.
- Firma digital XMLDSig y empaquetado ZIP (delegados a la feature 0005), con nomenclatura `RUC-31-VAAA-CCCCCCCC-ZIP`.
- Agrupación de varias GRE-Remitente en una misma GRE-T mediante documentos relacionados de tipo `09`.
- Consulta del detalle, del XML, del ZIP, del CDR e historial de envíos.
- Emisión desde gRPC, con el mismo modelo de dominio que REST.

### 2.2 Excluido

- La **GRE Remitente** (`09`), tratada en la [feature 0001](../0001-emision-gre-remitente/spec.md).
- La **comunicación de eventos del traslado** (tipo de evento `1`, `2`, `3`): el ítem 59 prohíbe expresamente su envío en el XML de la guía (código `3374`).
- La **anulación** del documento y la **baja por cambio de destinatario** (procedimiento administrativo de SUNAT).
- La generación de otros comprobantes electrónicos (factura, boleta, GRE-Remitente).
- La **GRE-Transportista simplificada**: la denominación no aparece en las fuentes entregadas y es **[NO VERIFICADO]**; lo verificable es el supuesto del Art. 3.3.b de la R.S. 123-2022 (una GRE-T que agrupa varias GRE-Remitente), que sí está en alcance (US-4).
- La concILIación con el DAM/DS y el Manifiesto de Carga: se valida la consistencia interna de los datos declarados, no se consultan esos sistemas.
- La **consulta del padrón de vehículos, conductores y RUC** de SUNAT: los códigos que exigen esa consulta (por ejemplo `4398`, `4400`, `3359`, `3389`) sólo se reportan como observaciones o errores de SUNAT en el envío, no se resuelven localmente.

## 3. Antecedentes y contexto

La R.S. 123-2022-SUNAT (Anexo VII b) y Anexo N.° 14) establece el estándar UBL 2.1 de la Guía de remisión transportista y obliga a que el documento se firme digitalmente y se envíe en un ZIP cuyo nombre coincida con el RUC emisor, el tipo de documento, la serie y el correlativo.

La hoja de validación «Guía-Transportista2_0» del XLSX oficial (`ValidacionesGREv20250421_0.xlsx`) define **64 campos de datos** (63 filas; los campos 27 y 28 comparten fila) y **178 códigos de retorno únicos** (265 ocurrencias). Cada campo está reflejado en [contrato-xml-ubl.md](contrato-xml-ubl.md) con su tag UBL, su obligatoriedad, su formato y sus códigos. Ese documento es la referencia normativa de esta feature.

La hoja «Guía-Remitente2_0» del mismo XLSX define, para el estándar Remitente, **90 campos** (81 filas; 82–90 comparten fila). La comparativa entre ambos estándares está en la sección 7 de [contrato-xml-ubl.md](contrato-xml-ubl.md).

Situación previa: el transportista genera el XML con una herramienta externa, lo firma, lo carga al WS y sigue el resultado por correo. Eso produce errores de nomenclatura del archivo (`1034`, `1035`, `1036`), duplicados de correlativo (`1033`), pérdida de trazabilidad y, sobre todo, una alta tasa de rechazos por reglas propias del estándar Transportista que no se cumplen por desconocimiento: subcontratación sin declarar (`3391`, `3390`, `3389`), pagador del flete ausente (`4388`), o conductor principal duplicado (`3358`).

## 4. Glosario

| Término | Definición |
|---|---|
| GRE-T | Guía de Remisión Electrónica Transportista; tipo de documento `31`. |
| Transportista | Emisor del documento (`cac:DespatchSupplierParty`, ítems 8–11); es el contribuyente que Prestige el servicio de transporte. |
| Remitente | Parte que entrega la mercancía; viaja en `cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty` (ítems 16–17). |
| Destinatario | Parte que recibe la mercancía; viaja en `cac:DeliveryCustomerParty` (ítems 18–19). |
| Subcontratador | Empresa que ejecuta el servicio por delegación del transportista; viaja en `cac:Shipment/cac:Consignment/cac:LogisticsOperatorParty` (ítems 60–61). |
| Consigna | Bloque `cac:Consignment` que identifica la delegación del servicio de transporte. |
| Pagador del flete | Parte que paga el servicio de transporte; se declara con un indicador `SUNAT_Envio_IndicadorPagadorFlete_*` y, si es un tercero, viaja en `cac:OriginatorCustomerParty` (ítems 62–63). |
| Serie | Código de 4 caracteres que identifica la secuencia (`V###`). |
| Correlativo | Número consecutivo dentro de la serie (1 a 99 999 999). |
| Identificador (`cbc:ID`) | `V###-########`, debe coincidir con el nombre del archivo. |
| Indicador (`SUNAT_Envio_*`) | Valor en `cbc:SpecialInstructions` que activa reglas del estándar (ítems 53–58). |
| Guías por eventos | Variante de la GRE en la que los eventos del traslado se comunican por separado; el ítem 59 prohíbe su uso en el XML de la guía. |
| Ítem de validación | Fila numerada (1..64) de la hoja de validación de SUNAT. |
| Código de retorno | Número (1001, 4388, …) con el que SUNAT responde por una regla incumplida. |
| Ticket | Número de Constancia de Recepción (CDR) devuelto por SUNAT. |

## 5. Personas y permisos

| Actor | Descripción | Permisos |
|---|---|---|
| ERP / TMS del transportista | Sistema del transportista que origina los traslados que ejecuta. | `gre.emitir`, `gre.consultar`, `gre.enviar` sobre los RUC de su token. |
| Usuario del transportista | Analista que revisa y corrige GRE-T. | `gre.consultar`, `gre.enviar`. |
| Operador de la plataforma | Personal de soporte. | `admin` (consulta global, sin emitir en nombre del transportista). |
| Servicio | Proceso interno. | Emisión y envío con la credencial del RUC transportista. |

El RUC que opera es siempre el del **transportista**, y debe estar habilitado en el claim `ruc` del token ([ADR-0002](../../adr/0002-autenticacion-jwt.md)). El RUC del remitente y el del subcontratador viajan en el cuerpo del documento y **no** habilitan por sí mismos ninguna operación.

## 6. Historias de usuario

### US-1 — Registrar una GRE-T

**Como** ERP del transportista
**Quiero** registrar una GRE-T con los datos del traslado que ejecuto
**Para** que el traslado quede documentado ante la SUNAT sin intervención manual

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given un payload válido con `tipoDocumento: "31"` y serie `V###`, When se invoca `POST /api/v1/guias`, Then se devuelve `201` con el identificador `V###-########`, el XML generado y su hash SHA-256 | Must |
| 2 | Given un payload con tipo `31` y serie que no empieza por `V`, When se invoca la emisión, Then se devuelve `400` con el detalle del campo `identificador.serie` | Must |
| 3 | Given un payload con un RUC no habilitado en el token, When se invoca la emisión, Then se devuelve `403 RUC_NO_AUTORIZADO` | Must |
| 4 | Given un payload que ya fue emitido con la misma `Idempotency-Key` y el mismo cuerpo, When se reintenta, Then se devuelve `200` con la GRE-T original | Must |
| 5 | Given dos emisiones concurrentes de la misma serie `V###`, When se procesan, Then los correlativos son consecutivos y sin duplicados | Must |
| 6 | Given una GRE-T emitida, When se consulta el detalle, Then se obtienen el XML, el estado, el hash y el historial de envíos | Should |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-1 | La GRE-T se identifica con el tipo de documento `31` y su serie empieza por `V`. | Ítem 6 (1050, 1051) e ítem 3 (1001) |
| RN-2 | El identificador sigue el formato `[V][A-Z0-9]{3}-[0-9]{1,8}` y su serie y correlativo deben coincidir con los del nombre del archivo. | Ítem 3 (1001, 1035, 1036, 1033, 1032) |
| RN-3 | `cbc:UBLVersionID` = `2.1` y `cbc:CustomizationID` = `2.0`. | Ítems 1 y 2 (2110, 2111, 2112, 2113) |
| RN-4 | La fecha de emisión no puede ser futura ni diferir en más de 1 día de la fecha de recepción; la hora sigue el formato `hh:mm:ss`. | Ítems 4 y 5 (3436, 2108, 2329, 3437, 3438) |
| RN-5 | El emisor es el transportista, que va en `cac:DespatchSupplierParty` con `cbc:ID[@schemeID='6']` y los atributos `schemeName`, `schemeAgencyName` y `schemeURI` del Catálogo 06; su RUC debe coincidir con el del nombre del archivo y estar activo, habido y afecto a los tributos autorizados; su razón social es obligatoria. | Ítem 8 (2678, 2511, 4255, 4256, 4257, 1034, 3349, 3350, 3439) e ítem 9 (1037, 4338) |
| RN-6 | El transportista puede consignar su número de Registro MTC y la entidad emisora de su autorización especial (Catálogo D-37), pero sólo un número de autorización. | Ítems 10 y 11 (4391, 4392, 4393, 4396, 3353, 4394, 4395, 4397) |
| RN-7 | El transportista no debe ser igual al remitente. | Ítem 16 (2560) |
| RN-8 | El remitente va en `cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty` con su tipo y número de documento (Catálogo 06) y su razón social. | Ítems 16 y 17 (2541, 2542, 3383, 3384, 3385, 3386, 4420, 4421, 3387, 4422) |
| RN-9 | El destinatario va en `cac:DeliveryCustomerParty`; si existe una GRE-Remitente como documento relacionado, el tipo y número de identidad del destinatario debe coincidir con el de esa GRE. | Ítems 18 y 19 (2759, 2760, 2757, 2758, 3443, 3444, 4373, 4374, 3434, 2761, 4152) |
| RN-11 | El vehículo principal es obligatorio: placa de 6 a 8 caracteres alfanuméricos en mayúsculas, Tarjeta Única de Circulación Electrónica o Certificado de Habilitación vehicular, y entidad autorizadora con un único número de autorización. | Ítems 35, 36 y 37 (2566, 2567, 4398, 4399, 3355, 4400, 4406, 3356, 4403, 4407, 4405) |
| RN-15 | Los conductores se declaran con `cac:JobTitle` = `Principal` (uno solo) o `Secundario` (hasta dos); exigen tipo y número de documento, nombres, apellidos y número de licencia de 9 a 10 caracteres. | Ítems 41 a 48 (3357, 3358, 2570, 2571, 2568, 2569, 3359, 3360, 3361, 4409, 2572, 2573, 4412, 4376, 3362) |
| RN-16 | El punto de partida y el punto de llegada declaran ubigeo (6 dígitos, Catálogo 13), dirección detallada y georreferencia con coordenadas `n(3,8)` y unidad `DD`. | Ítems 29 a 34 (2775, 2776, 3363, 2577, 4076, 3413, 3368, 2574, 4178, 4068, 4431) |
| RN-17 | La fecha de inicio de traslado es obligatoria, con formato `YYYY-MM-DD` y no anterior a la fecha de emisión. | Ítem 50 (3406, 3343, 3407) |
| RN-18 | El peso bruto total de la carga es obligatorio, con formato `n(12,3)` y unidad `KGM` o `TNE`. | Ítem 52 (2880, 2523, 2881) |
| RN-19 | El detalle de bienes es obligatorio salvo cuando el documento relacionado `09` tiene serie electrónica; cada línea exige número de orden, cantidad `n(12,10)`, unidad de medida (Catálogos 03 y 65) y descripción. | Ítems 20 a 23 (3435, 4434, 2023, 3458, 2752, 2580, 2780, 2883, 4320, 2781, 4084) |
| RN-20 | Cada línea puede declarar código del bien, código de producto SUNAT (Catálogo 25), GTIN, partida arancelaria e indicador de bien normalizado como propiedad del ítem (Catálogo 55). | Ítems 24 a 28 (4085, 3372, 3002, 3373, 3425, 3375, 4335, 3426, 3352, 3064, 3377, 3378, 3429, 3396, 3379) |
| RN-21 | La anotación opcional sobre los bienes a transportar se declara en una línea con `cbc:ID` = `0` y descripción de hasta 500 caracteres. | Ítem 51 (4429, 4430) |
| RN-22 | Las observaciones del documento se consignan en `cbc:Note` con un máximo de 250 caracteres. | Ítem 7 (4186) |
| RN-23 | `cac:Shipment/cbc:ID` = `SUNAT_Envio` (identificador del traslado). | Ítem 49 |
| RN-24 | No debe consignarse `cac:TransportEvent/cbc:TransportEventTypeCode` (uso exclusivo de las guías por eventos). | Ítem 59 (3374) |
| RN-25 | La firma digital del transportista es obligatoria. | Ítem 64 (1059, 2336) |

**Validaciones / errores esperados**

| Código | Condición | Tipo | Comportamiento |
|---|---|---|---|
| `1050`/`1051` | `cbc:DespatchAdviceTypeCode` ausente o con valor distinto de `31`. | ERROR | El generador lo emite siempre correcto; se defienden en el validador. |
| `1001` | La serie-correlativo del `cbc:ID` no cumple `[V][A-Z0-9]{3}-[0-9]{1,8}`. | ERROR | `422`, ítem 3. |
| `1034`/`1035`/`1036` | El RUC, la serie o el correlativo del `cbc:ID` no coinciden con el nombre del archivo. | ERROR | El generador los construye desde la misma fuente. |
| `1033`/`1032` | El comprobante ya fue registrado con otros datos, o ya está informado con estado anulado o rechazado. | ERROR | Se rechaza el envío; la GRE queda `RECHAZADA`. |
| `2108`/`2329`/`3436` | Fecha de emisión fuera de la ventana de 1 día. | ERROR | Se rechaza en validación local; no se envía. |
| `3437`/`3438` | Hora de emisión ausente o con formato distinto de `hh:mm:ss`. | ERROR | `422`, ítem 5. |
| `2110`/`2112` | Versión UBL o `CustomizationID` distintos de 2.1 / 2.0. | ERROR | El generador los emite siempre correctos; se defienden en el validador. |
| `2511`/`2678` | Tipo de documento de identidad del transportista ausente o no aceptado. | ERROR | `422`, ítem 8. |
| `1034` | El RUC del transportista no coincide con el del nombre del archivo. | ERROR | `422`, ítem 8. |
| `3349`/`3350`/`3439` | RUC del transportista inactivo, no habido o no afecto a los tributos autorizados. | ERROR | `422`, ítem 8. |
| `1037` | Falta la razón social del transportista. | ERROR | `422`, ítem 9. |
| `3353` | Se consigna más de un número de autorización (MTC o entidad autorizadora). | ERROR | `422`, ítems 10 y 11. |
| `4391`/`4392`/`4393`/`4394`/`4395`/`4396`/`4397` | Registro MTC o entidad autorizadora incompletos o no vigentes. | OBSERV | Se acepta con observación y se registra en `gre_observacion`. |
| `2560` | El transportista y el remitente tienen el mismo tipo y número de documento. | ERROR | `422`, ítem 16. |
| `2541`/`2542`/`3383`/`3384`/`3385`/`3386` | Datos de identidad del remitente ausentes, inválidos o no existentes. | ERROR | `422`, ítem 16. |
| `3387` | Falta la razón social del remitente. | ERROR | `422`, ítem 17. |
| `2757`/`2758`/`2759`/`2760`/`3443`/`3444` | Datos de identidad del destinatario ausentes, inválidos o no existentes. | ERROR | `422`, ítem 18. |
| `3434` | El destinatario no coincide con el de la GRE-Remitente relacionada. | ERROR | `422`, ítem 18. |
| `2761` | Falta la razón social del destinatario. | ERROR | `422`, ítem 19. |
| `2566`/`2567` | Placa del vehículo principal ausente o con formato inválido. | ERROR | `422`, ítem 35. |
| `4398`/`4399`/`4400`/`4403`/`4405`/`4406`/`4407` | Placa, TUC o entidad autorizadora del vehículo no encontradas en las bases. | OBSERV | Se acepta con observación y se registra en `gre_observacion`. |
| `3355`/`3356` | TUC o número de autorización del vehículo con formato inválido o duplicado. | ERROR | `422`, ítems 36, 37 y 40. |
| `4389` | Se consignan más de dos vehículos secundarios. | OBSERV | Se acepta con observación; la plataforma lo eleva a bloqueante. |
| `3357`/`3358` | Falta el conductor principal o hay más de uno. | ERROR | `422`, ítem 41. |
| `2570`/`2571`/`2568`/`2569` | Datos de identidad del conductor ausentes o inválidos. | ERROR | `422`, ítems 42 y 46. |
| `3359` | El documento de identidad del conductor no existe en el padrón. | ERROR | `422`, ítems 42 y 46. |
| `3360`/`3361` | Faltan los nombres o los apellidos del conductor. | ERROR | `422`, ítems 43 y 47. |
| `2572`/`2573` | Número de licencia del conductor ausente o con formato inválido. | ERROR | `422`, ítems 44 y 48. |
| `4376` | Se consignan más de dos conductores secundarios. | OBSERV | Se acepta con observación; la plataforma lo eleva a bloqueante. |
| `3362` | Se repite la información de un conductor secundario. | ERROR | `422`, ítem 45. |
| `2775`/`2776`/`3363`/`3368` | Ubigeo de partida o de llegada ausente, inválido o inexistente. | ERROR | `422`, ítems 29 y 32. |
| `2577`/`2574` | Falta la dirección detallada de partida o de llegada. | ERROR | `422`, ítems 30 y 33. |
| `3413` | El georreferencia no cumple el formato `n(3,8)`. | ERROR | `422`, ítems 31 y 34. |
| `3406`/`3407` | Fecha de inicio de traslado ausente o con formato inválido. | ERROR | `422`, ítem 50. |
| `3343` | La fecha de inicio de traslado es anterior a la fecha de emisión. | ERROR | `422`, ítem 50. |
| `2880`/`2881` | Falta el peso bruto total o su unidad de medida. | ERROR | `422`, ítem 52. |
| `2523` | El peso bruto total no cumple el formato `n(12,3)`. | ERROR | `422`, ítem 52. |
| `3435`/`2023`/`3458`/`2752` | El detalle de bienes es obligatorio y no se ha consignado. | ERROR | `422`, ítem 20. |
| `4434` | Se ha consignado el detalle de bienes cuando no corresponde. | OBSERV | Se acepta con observación; la plataforma lo eleva a bloqueante. |
| `2580`/`2780` | Cantidad de la línea de bien ausente o con formato inválido. | ERROR | `422`, ítem 21. |
| `2883` | Falta la unidad de medida de la línea de bien. | ERROR | `422`, ítem 22. |
| `2781` | Falta la descripción de la línea de bien. | ERROR | `422`, ítem 23. |
| `3372`/`3002`/`3373`/`3425` | Código de producto SUNAT ausente, inválido o no encontrado. | ERROR | `422`, ítem 25. |
| `3375` | El GTIN no cumple el formato. | ERROR | `422`, ítem 26. |
| `4429`/`4430` | La anotación opcional sobre los bienes a transportar falta o no cumple el formato. | OBSERV | Se acepta con observación y se registra en `gre_observacion`. |
| `4186` | Las observaciones del documento superan los 250 caracteres. | OBSERV | Se acepta con observación y se registra en `gre_observacion`. |
| `3433`/`2536` | La GRE-Remitente relacionada no existe en SUNAT o fue dada de baja. | ERROR | `422` local; se confirma en el envío. |
| `3374` | Se consigna el tipo de evento. | ERROR | El generador nunca lo emite; el validador lo bloquea. |
| `1059`/`2336` | Falta la firma digital o no cumple los requisitos de vigencia. | ERROR | Se rechaza el envío; el documento no pasa a `FIRMADA`. |
| `0154` | El RUC del archivo no corresponde al RUC autorizado del usuario. | ERROR | `422` en el envío; la GRE queda `RECHAZADA`. |
| `0161` | El nombre del XML no coincide con el del ZIP. | ERROR | Se rehace el empaquetado; el error es del generador. |
| `0306` | El XML no se puede parsear. | ERROR | Se regenera desde la fuente persistida. |

### US-2 — Declarar la subcontratación del servicio de transporte

**Como** ERP del transportista
**Quiero** declarar que el servicio de transporte se ejecuta por delegación de un subcontratador
**Para** que SUNAT conozca la cadena de transporte real y no rechace la guía

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given un payload con el indicador `SUNAT_Envio_IndicadorTrasporteSubcontratado` y el bloque del subcontratador, When se emite, Then el XML contiene `cac:Shipment/cac:Consignment` con `cbc:ID` = `SUNAT_Envio` y `cac:LogisticsOperatorParty` | Must |
| 2 | Given un payload con el indicador de subcontratación y sin el número de RUC del subcontratador, When se emite, Then se devuelve `422` con el ítem 60 y el código `4424` (observación en SUNAT, bloqueante en la plataforma) | Must |
| 3 | Given un payload con el subcontratador del mismo RUC que el transportista, When se emite, Then se devuelve `422` con el código `3390` | Must |
| 4 | Given un payload con el subcontratador cuyo tipo de documento no es `6` (RUC), When se emite, Then se devuelve `422` con el código `3391` | Must |
| 5 | Given un payload con datos del subcontratador pero sin el indicador de subcontratación, When se emite, Then el XML no contiene `cac:Consignment` y la plataforma rechaza la emisión con `422` por política local más estricta que SUNAT | Should |
| 6 | Given un payload con el subcontratador cuyo RUC no existe en el padrón, When se envía a SUNAT, Then SUNAT responde con el código `3389` y la GRE queda `RECHAZADA` | Should |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-12 | La subcontratación se declara con el indicador `SUNAT_Envio_IndicadorTrasporteSubcontratado` y se documenta en `cac:Shipment/cac:Consignment` con `cbc:ID` = `SUNAT_Envio` y `cac:LogisticsOperatorParty`. | Ítem 57 (3344), ítem 60 |
| RN-12.1 | El subcontratador es de tipo de documento `6` (RUC), debe existir en el listado, no puede ser el transportista y exige su razón social. | Ítem 60 (4425, 3391, 4424, 3390, 3389) e ítem 61 (4426, 4427) |
| RN-14 | Los indicadores se escriben uno por tag `cbc:SpecialInstructions`, sin repetirse, y su valor debe pertenecer al conjunto `SUNAT_Envio_*` admitido para el tipo de guía. | Ítems 53–58 (3344, 3388) |

**Validaciones / errores esperados**

| Código | Condición | Tipo | Comportamiento |
|---|---|---|---|
| `3344` | El indicador de transporte subcontratado (o cualquier otro) se repite. | ERROR | `422`, ítem 57. |
| `3388` | El valor del indicador no pertenece al conjunto admitido para el tipo de guía. | ERROR | `422`, ítem 54. |
| `3391` | El tipo de documento de identidad del subcontratador no es `6`. | ERROR | `422`, ítem 60. |
| `3390` | El subcontratador es el transportista. | ERROR | `422`, ítem 60. |
| `3389` | El RUC del subcontratador no existe. | ERROR | `422` local; el padrón se verifica en el envío. |
| `4425` | Falta el `@schemeID` del subcontratador. | OBSERV | Bloqueante en la plataforma: se devuelve `422` con el ítem 60. |
| `4424` | Falta el número de documento de identidad del subcontratador. | OBSERV | Bloqueante en la plataforma: se devuelve `422` con el ítem 60. |
| `4426`/`4427` | Falta la razón social del subcontratador o no cumple el formato. | OBSERV | Bloqueante en la plataforma: se devuelve `422` con el ítem 61. |

### US-3 — Declarar el pagador del flete

**Como** ERP del transportista
**Quiero** declarar quién paga el servicio de transporte
**Para** que SUNAT pueda imputar el flete al pagador correcto (remitente, subcontratador o tercero)

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given un payload con `SUNAT_Envio_IndicadorPagadorFlete_Remitente`, When se emite, Then el XML contiene ese indicador y no contiene `cac:OriginatorCustomerParty` | Must |
| 2 | Given un payload con `SUNAT_Envio_IndicadorPagadorFlete_Tercero` y los datos del tercero pagador, When se emite, Then el XML contiene el indicador y el bloque `cac:OriginatorCustomerParty` | Must |
| 3 | Given un payload sin indicador de pagador de flete, When se emite, Then se devuelve `422` con el código `4388` (observación en SUNAT, bloqueante en la plataforma) | Must |
| 4 | Given un payload con dos indicadores de pagador de flete, When se emite, Then se devuelve `422` con el código `3344` | Must |
| 5 | Given un payload con `SUNAT_Envio_IndicadorPagadorFlete_Tercero` y sin datos del pagador, When se emite, Then se devuelve `422` con el ítem 62 y el código `4401` | Must |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-13 | El pagador del flete se declara con exactamente uno de los indicadores `SUNAT_Envio_IndicadorPagadorFlete_Remitente`, `SUNAT_Envio_IndicadorPagadorFlete_Subcontratador` o `SUNAT_Envio_IndicadorPagadorFlete_Tercero`. | Ítem 58 (4388, 3344) |
| RN-13.1 | Sólo con el indicador `_Tercero` se completa `cac:OriginatorCustomerParty` con el tipo y número de documento (Catálogo 06) y la razón social del pagador. | Ítems 62 y 63 (4401, 3399, 4402, 3400, 3401, 3402, 4370, 4423) |
| RN-13.2 | El tipo de documento del pagador del flete es `1` (DNI) o `6` (RUC); en el primer caso el número tiene 8 dígitos y en el segundo 11. | Ítem 62 (3399, 3400, 3401, 3402) |
| RN-12.2 | El indicador `SUNAT_Envio_IndicadorPagadorFlete_Subcontratador` exige que exista el bloque `cac:Consignment` (subcontratador declarado). | Ítem 58 (4388) en relación con el ítem 60 |

**Validaciones / errores esperados**

| Código | Condición | Tipo | Comportamiento |
|---|---|---|---|
| `4388` | No se consigna ninguno de los tres indicadores de pagador de flete. | OBSERV | Bloqueante en la plataforma: se devuelve `422` con el ítem 58, porque el pagador no puede determinarse. |
| `3344` | Se repite un indicador de pagador de flete. | ERROR | `422`, ítem 58. |
| `3399` | Tipo de documento de identidad del pagador del flete inválido. | ERROR | `422`, ítem 62. |
| `3400` | El número de documento del pagador del flete no cumple el formato. | ERROR | `422`, ítem 62. |
| `3401`/`3402` | El DNI o el RUC del pagador del flete no existe. | ERROR | `422` local; el padrón se verifica en el envío. |
| `4401`/`4402`/`4370` | Falta el tipo de documento, el número o la razón social del pagador del flete. | OBSERV | Bloqueante en la plataforma: se devuelve `422` con los ítems 62 y 63. |

### US-4 — Agrupar varias GRE-Remitente en una GRE-T

**Como** ERP del transportista
**Quiero** consignar en una sola GRE-T varias GRE-Remitente que comparten remitente, destinatario, punto de partida y punto de llegada
**Para** no emitir una guía por cada GRE-Remitente cuando el traslado es único

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given un payload con dos documentos relacionados de tipo `09` con serie electrónica, When se emite, Then el XML contiene dos `cac:AdditionalDocumentReference` y la emisión se acepta | Must |
| 2 | Given un payload con tres documentos relacionados de tipo `09` con serie **no** electrónica, When se emite, Then se devuelve `422` con el código `3346` | Must |
| 3 | Given un payload con dos documentos relacionados de tipo `09` con el mismo número, When se emite, Then se devuelve `422` con el código `3340` | Must |
| 4 | Given un payload con un documento relacionado de tipo `09` cuya serie electrónica tiene `cbc:ID` = `0`, When se emite, Then no se exige el detalle de bienes y la emisión se acepta | Should |
| 5 | Given un payload con un documento relacionado de tipo `09` que no existe en SUNAT, When se envía, Then SUNAT responde con `3433` y la GRE queda `RECHAZADA` | Should |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-10 | Se permite un documento relacionado; hasta dos si alguno es de tipo `31`, `65`, `66`, `67`, `68` o `69`; más de uno si hay un `09` con serie electrónica. | Ítem 13 (2692, 3403, 3345, 3346, 4251, 4252, 4253) |
| RN-10.1 | El par «código de tipo de documento relacionado + `cbc:ID`» no puede repetirse. | Ítem 14 (3340) |
| RN-10.2 | Para los tipos `01`, `03`, `04`, `12`, `48`, `09` y `31` el documento relacionado exige el RUC de su emisor, con `@schemeID='6'`; para el tipo `31` debe ser el RUC del transportista y para el `09` el del remitente. | Ítem 15 (3380, 3381, 3408, 3382, 4255, 4256, 4257) |
| RN-10.3 | El número del documento relacionado sigue el formato propio de su tipo (`3441`); para el tipo `09` la GRE consignada debe existir y no haber sido dada de baja. | Ítem 14 (3376, 3441, 3341, 2536, 3342, 4404, 3433, 4382, 3207) |
| RN-10.4 | Una GRE-T puede agrupar varias GRE-Remitente siempre que coincidan el remitente, el destinatario, el punto de partida y el punto de llegada. | R.S. 123-2022-SUNAT, Art. 3.3.b |
| RN-19 | El detalle de bienes a transportar es obligatorio salvo cuando el documento relacionado `09` tiene serie electrónica; en ese caso el detalle no debe consignarse. | Ítems 20 y 51 (3435, 4434, 2023, 3458, 2752, 4429, 4430) |
| RN-9 | Si existe una GRE-Remitente como documento relacionado, el tipo y número de identidad del destinatario debe coincidir con el de esa GRE. | Ítem 18 (3434) |

**Validaciones / errores esperados**

| Código | Condición | Tipo | Comportamiento |
|---|---|---|---|
| `2692` | El tipo de documento relacionado no es válido para la GRE-T (Catálogo 61). | ERROR | `422`, ítem 13. |
| `3403` | Falta el número del documento relacionado. | ERROR | `422`, ítem 13. |
| `3345` | Se consignan más de dos documentos relacionados sin que ninguno sea `31`, `65`, `66`, `67`, `68` o `69`. | ERROR | `422`, ítem 13. |
| `3346` | Se consignan más de un documento relacionado y ninguno es `31`, `65`, `66`, `67`, `68`, `69` o `09` de serie electrónica. | ERROR | `422`, ítem 13. |
| `3340` | Documento relacionado duplicado. | ERROR | `422`, ítem 14. |
| `3376` | Falta el código del tipo de documento relacionado. | ERROR | `422`, ítem 14. |
| `3441` | El número del documento relacionado no cumple el formato previsto para su tipo. | ERROR | `422`, ítem 14. |
| `3341`/`3342` | El número del documento relacionado no existe o no está activo. | ERROR | `422` local; se confirma en el envío. |
| `2536` | La GRE-Remitente consignada fue dada de baja por cambio de destinatario. | ERROR | `422` local; se confirma en el envío. |
| `3207` | El documento relacionado no cumple con el formato de la serie electrónica. | ERROR | `422` local; se confirma en el envío. |
| `3433` | La GRE-Remitente consignada no existe en SUNAT. | ERROR | `422` local; se confirma en el envío. |
| `4382`/`4404` | La GRE-Remitente consignada presenta una inconsistencia de formato. | OBSERV | Se acepta con observación y se registra en `gre_observacion`. |
| `3380`/`3381`/`3408`/`3382` | RUC del emisor del documento relacionado ausente, incorrecto, inexistente o con tipo de documento distinto de `6`. | ERROR | `422`, ítem 15. |
| `4371`/`4410`/`4372` | Descripción del tipo de documento relacionado ausente, incoherente o con formato inválido. | OBSERV | Se acepta con observación y se registra en `gre_observacion`. |
| `3435`/`2023`/`3458`/`2752` | El detalle de bienes falta cuando es obligatorio. | ERROR | `422`, ítem 20. |
| `4434` | Se ha consignado el detalle de bienes cuando no corresponde. | OBSERV | Bloqueante en la plataforma: se devuelve `422`, ítem 20. |

### US-5 — Consultar el estado de una GRE-T

**Como** usuario del transportista
**Quiero** consultar una GRE-T emitida, su XML y su historial de envíos
**Para** responder a un reclamo o a una fiscalización sin volver al ERP

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given una GRE-T emitida, When se solicita su detalle, Then se devuelve el estado, el hash, el XML y los intentos de envío | Must |
| 2 | Given una GRE-T con observaciones, When se consulta el detalle, Then se listan las observaciones con su código de retorno y su tipo (`ERROR`/`OBSERV`/`REST`) | Must |
| 3 | Given una GRE-T de otro RUC, When se solicita su detalle, Then se devuelve `404` (no se revela su existencia) | Must |
| 4 | Given una GRE-T aún no enviada, When se solicita el CDR, Then se devuelve `404` con el estado actual | Should |

### US-6 — Validar antes de emitir

**Como** ERP del transportista
**Quiero** validar el payload sin persistirlo
**Para** corregir los datos en origen y no acumular documentos rechazados

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given un payload con un error de negocio, When se invoca la validación, Then se devuelve la lista de violaciones con ítem, tag UBL, código de retorno y mensaje | Must |
| 2 | Given la misma operación, When se valida dos veces, Then no se crea ningún documento ni correlativo | Must |
| 3 | Given un payload válido, When se valida, Then `valido = true` y la lista de violaciones vacía | Must |

## 7. Flujo principal

1. El ERP del transportista obtiene un token del emisor OAuth 2.0 (ver [ADR-0002](../../adr/0002-autenticacion-jwt.md)).
2. El ERP envía la solicitud JSON con `tipoDocumento: "31"`, serie `V###` y la cabecera `Idempotency-Key`.
3. El servicio valida el payload contra el esquema (400 si no cumple el contrato).
4. El servicio verifica que el RUC del payload figure en el claim `ruc` del token (403 si no).
5. El servicio selecciona la estrategia de validación del tipo `31` y carga los catálogos vigentes; ejecuta las reglas de la feature 0004 (422 si hay violaciones `ERROR`).
6. El servicio reserva el correlativo de la serie `V###` dentro de la misma transacción (400/409 si la serie no existe o está anulada).
7. El servicio genera el XML UBL 2.1 con la estrategia del tipo `31`: emisor en `cac:DespatchSupplierParty`, remitente en `cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty`, y los bloques condicionales `cac:Consignment` y `cac:OriginatorCustomerParty` según los indicadores declarados.
8. El servicio firma el XML con el certificado del transportista y arma el ZIP `RUC-31-VAAA-CCCCCCCC-ZIP` (feature 0005).
9. El servicio persiste el documento, sus ítems, sus documentos relacionados, la consigna y el pagador del flete, y devuelve `201` con el identificador, el estado `FIRMADA` y el hash.
10. El envío a SUNAT es un paso posterior y explícito (`POST /api/v1/guias/{id}/envios`).

## 8. Flujos alternativos y de error

| ID | Situación | Disparo | Respuesta del sistema | Estado resultante |
|---|---|---|---|---|
| FA-1 | Payload inválido contra el esquema | JSON malformado, tipo/rango incorrecto o serie que no empieza por `V` | `400` con el detalle de cada campo | — (no se persiste) |
| FA-2 | RUC no habilitado | `rucEmisor` ausente del claim `ruc` | `403 RUC_NO_AUTORIZADO` | — |
| FA-3 | Regla de negocio incumplida | Vehículo, conductor, indicador, documentos relacionados o pagador del flete incompletos | `422` con la lista de violaciones | — |
| FA-4 | Serie inexistente, anulada o de prefijo incorrecto | `V###` no registrada para el RUC, o serie `T###` con tipo `31` | `400`/`409` con el detalle de la serie | — |
| FA-5 | Conflicto de idempotencia | Misma clave, cuerpo distinto | `409 IDEMPOTENCIA_CONFLICTO` | — |
| FA-6 | Error de base de datos durante la emisión | Falla la transacción | `500`, se revierte la transacción completa (incluida la reserva de correlativo) | — |
| FA-7 | SUNAT rechaza el documento | El WS responde con un `ERROR` | La GRE-T queda `RECHAZADA` con el código de retorno; no se reintenta | `RECHAZADA` |
| FA-8 | SUNAT responde con observaciones | El WS responde con `OBSERV` | La GRE-T queda `ACEPTADA_CON_OBSERVACIONES` con la observación persistida | `ACEPTADA_CON_OBSERVACIONES` |
| FA-9 | Error de transporte en el envío | Timeout o 5xx | Reintento con backoff exponencial; al agotar los intentos, `PENDIENTE_REINTENTO` | `PENDIENTE_REINTENTO` |
| FA-10 | Subcontratación declarada sin datos de subcontratador | Indicador de subcontratación presente, bloque `cac:Consignment` ausente | `422` con el ítem 60 y el código `4424` | — |
| FA-11 | Subcontratador igual al transportista | RUC del subcontratador = RUC del transportista | `422` con el código `3390` | — |
| FA-12 | Pagador del flete no declarado o duplicado | Ninguno o más de un indicador `SUNAT_Envio_IndicadorPagadorFlete_*` | `422` con los códigos `4388` o `3344` | — |
| FA-13 | Certificado digital del transportista no disponible | El certificado no está vigente o no está cargado | El documento queda `VALIDADA`; se reintenta la firma más tarde | `VALIDADA` |

## 9. Requisitos funcionales

| ID | Requisito | Origen |
|---|---|---|
| RF-1 | Registrar la GRE-T desde JSON reutilizando el contrato `SolicitudEmision` con `tipoDocumento: "31"` y validación de esquema | US-1, RN-1, RN-2 |
| RF-2 | Reservar el correlativo de la serie `V###` de forma transaccional y sin duplicados | US-1, RN-2 |
| RF-3 | Validar las reglas de negocio de la hoja «Guía-Transportista2_0» (ítems 1..64) | US-1, US-6, RN-1..RN-25 |
| RF-4 | Generar el XML UBL 2.1 conforme al estándar `DespatchAdvice` con `DespatchAdviceTypeCode` = `31` | US-1, RN-1..RN-5 |
| RF-5 | Firmar el XML con el certificado del transportista y empaquetarlo en el ZIP con la nomenclatura `RUC-31-VAAA-CCCCCCCC-ZIP` | US-1, RN-25 |
| RF-6 | Persistir el documento y exponerlo en consulta y descarga | US-5 |
| RF-7 | Exponer el historial de envíos con sus códigos de retorno | US-5, FA-7, FA-8 |
| RF-8 | Permitir el envío explícito a SUNAT con reintentos clasificados | FA-7..FA-9 |
| RF-9 | Exponer la misma operación por gRPC | Contrato gRPC |
| RF-10 | Registrar auditoría de toda la operación | RNF-4 |
| RF-11 | Declarar la subcontratación del servicio de transporte (indicador + bloque `cac:Consignment`) | US-2, RN-12, RN-12.1 |
| RF-12 | Declarar el pagador del flete (remitente, subcontratador o tercero) y generar `cac:OriginatorCustomerParty` cuando corresponda | US-3, RN-13, RN-13.1, RN-13.2 |
| RF-13 | Agrupar varias GRE-Remitente en una misma GRE-T mediante documentos relacionados de tipo `09` | US-4, RN-10, RN-10.1..RN-10.3 |
| RF-14 | Registrar y validar la serie `V###` asociada al RUC del transportista, rechazando series con prefijo distinto de `V` para el tipo `31` | US-1, RN-1, RN-2 |
| RF-15 | Reportar las observaciones de SUNAT distinguiendo `ERROR` y `OBSERV` y persistiéndolas por ítem | US-5, US-6, FA-7, FA-8 |

## 10. Requisitos no funcionales

| ID | Categoría | Requisito | Métrica |
|---|---|---|---|
| RNF-1 | Rendimiento | Emisión (validación + XML + firma) sin envío | p95 < 1,5 s con 500 ítems |
| RNF-2 | Rendimiento | Validación sin persistencia | p95 < 500 ms |
| RNF-3 | Disponibilidad | API de emisión | 99,5 % mensual |
| RNF-4 | Trazabilidad | Cada emisión queda auditada con sujeto, RUC, serie, correlativo y hash | 100 % de las emisiones |
| RNF-5 | Seguridad | El RUC efectivo del transportista proviene del token | 0 accesos con RUC no autorizado |
| RNF-6 | Integridad | El XML persistido conserva su hash SHA-256 | Verificación en cada descarga |
| RNF-7 | Compatibilidad | El XML generado es aceptado por el validador oficial de SUNAT | 100 % de los casos de prueba |
| RNF-8 | Idempotencia | Repetir la misma `Idempotency-Key` no crea un segundo documento | 0 duplicados |
| RNF-9 | Aislamiento | Una GRE-T de un RUC no es visible ni modificable desde otro RUC | 0 fugas entre tenants (RLS + filtro de token) |
| RNF-10 | Protección de datos | Los DNI de conductores y de los contactos no se exponen en las respuestas de consulta | 0 exposures en los campos de respuesta |
| RNF-11 | Mantenibilidad | El conjunto de reglas de la GRE-T está aislado del de la GRE-Remitente | 0 condicionales de tipo de documento dentro de las clases de regla |

## 11. Modelo de datos

| Entidad | Campos principales | Notas |
|---|---|---|
| `GreDocumento` | id, ruc_emisor, tipo_documento, serie, correlativo, estado, fecha_emision, hora_emision, hash_xml, xml_firmado (BYTEA), nombre_archivo, solicitud_json | Único por `(ruc_emisor, tipo_documento, serie, correlativo)`. `tipo_documento` = `31` en esta feature. |
| `SerieCorrelativo` | ruc_emisor, tipo_documento, serie, ultimo_correlativo, estado | Reserva con `UPDATE ... RETURNING` (ver [ADR-0001](../../adr/0001-seleccion-db.md)). |
| `GreItem` | gre_id, orden, descripcion, cantidad, unidad_medida, codigo_producto_sunat, gtin | Único por `(gre_id, orden)`. |
| `GrePropiedadItem` | gre_item_id, codigo (Catálogo 55), nombre, valor | Único por `(gre_item_id, codigo, valor)`. |
| `GreDocumentoRelacionado` | gre_id, tipo, descripcion, numero, ruc_emisor | Único por `(gre_id, tipo, numero)` (código `3340`). |
| `GreConsignacion` | gre_id, ruc_subcontratador, nombre_subcontratador | 0..1 por documento. Existe si y sólo si se declara el indicador de subcontratación. |
| `GrePagadorFlete` | gre_id, rol (`REMITENTE`/`SUBCONTRATADOR`/`TERCERO`), tipo_documento_identidad, numero_documento, nombre | 0..1 por documento. Sólo se persiste con rol `TERCERO`. |
| `GreEnvio` | gre_id, intento, estado, codigo_retorno, descripcion_respuesta, ticket, fecha | Un registro por intento. |
| `GreObservacion` | gre_id, item, tag_ubl, codigo_retorno, tipo_retorno, mensaje | Violaciones con `OBSERV` o `ERROR` de SUNAT. |
| `Auditoria` | sujeto, ruc_emisor, operacion, recurso, fecha, detalle | Append-only. |

Invariante estructural: en un documento con `tipo_documento = '31'`, la existencia de `GreConsignacion` y el indicador de subcontratación son equivalentes; y la existencia de `GrePagadorFlete` implica el indicador `SUNAT_Envio_IndicadorPagadorFlete_Tercero`.

## 12. Contratos

| Contrato | Ubicación | Descripción |
|---|---|---|
| OpenAPI | [openapi.yaml](../../contracts/openapi.yaml) | Superficie REST de emisión, consulta, envío y catálogos. Ya cubre `tipoDocumento` `31` y la serie `V###`; esta feature no añade endpoints nuevos. |
| JSON Schema | [schema.json](../../contracts/schema.json) | Contrato de entrada del payload de emisión (`SolicitudEmision`) con el condicional de serie `^V[A-Z0-9]{3}$` para `tipoDocumento = "31"`. |
| gRPC | [grpc_service.proto](../../contracts/grpc_service.proto) | Superficie gRPC equivalente; `TIPO_DOCUMENTO_GRE_TRANSPORTISTA` = `31`, serie `V###`. |
| XML UBL | [contrato-xml-ubl.md](contrato-xml-ubl.md) | Correspondencia campo a campo entre el JSON y el XML UBL 2.1 del estándar Transportista, con los 64 ítems de validación. |
| Códigos de retorno | [codigos-retorno-sunat.md](../0004-validacion-documento/codigos-retorno-sunat.md) | Catálogo de códigos de retorno de SUNAT employed por la validación. |
| Mapeo de errores | [mapeo-errores.md](../0004-validacion-documento/mapeo-errores.md) | Comportamiento del sistema ante `ERROR`, `OBSERV` y errores de transporte. |
| Catálogos | [cat-01](../0003-catalogos-referencia/catalogos/cat-01-codigo-tipo-documento.md), [cat-03](../0003-catalogos-referencia/catalogos/cat-03-codigo-tipo-unidad-medida-comercial.md), [cat-06](../0003-catalogos-referencia/catalogos/cat-06-codigo-tipo-documento-identidad.md), [cat-13](../0003-catalogos-referencia/catalogos/cat-13-codigo-ubicacion-geografica-ubigeo.md), [cat-18](../0003-catalogos-referencia/catalogos/cat-18-modalidad-transporte.md), [cat-20](../0003-catalogos-referencia/catalogos/cat-20-motivo-traslado.md), [cat-25](../0003-catalogos-referencia/catalogos/cat-25-codigo-producto-sunat.md), [cat-55](../0003-catalogos-referencia/catalogos/cat-55-codigo-identificacion-concepto-tributario.md), [cat-61](../0003-catalogos-referencia/catalogos/cat-61-documentos-relacionados-gre.md), [cat-62](../0003-catalogos-referencia/catalogos/cat-62-bienes-normalizados.md), [cat-65](../0003-catalogos-referencia/catalogos/cat-65-unidades-medida-gre-dam-ds.md), [cat-D-37](../0003-catalogos-referencia/catalogos/cat-d37-entidades-autorizadoras-translado.md) | Catálogos SUNAT vigentes aplicados en la validación. |

## 13. Criterios de aceptación de la feature

- [ ] Un payload de GRE-T (transportista, remitente, destinatario, vehículo principal, conductor principal y un documento relacionado `09`) se convierte en un XML UBL 2.1 con `DespatchAdviceTypeCode` = `31` que el validador oficial de SUNAT acepta sin observaciones.
- [ ] El nombre del archivo XML y del ZIP es `RUC-31-VAAA-CCCCCCCC-XML` / `RUC-31-VAAA-CCCCCCCC-ZIP` y su serie y correlativo coinciden con el `cbc:ID`.
- [ ] El transportista aparece en `cac:DespatchSupplierParty` y el remitente en `cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty`; el validador rechaza con `2560` cuando ambos coinciden.
- [ ] El vehículo principal es obligatorio y su placa admite de 6 a 8 caracteres alfanuméricos en mayúsculas; con más de dos vehículos secundarios se devuelve `422` con el código `4389`.
- [ ] El conductor principal es único (`3358`) y se admiten como máximo dos secundarios (`4376`).
- [ ] Con el indicador de subcontratación se genera `cac:Consignment` con `cbc:ID` = `SUNAT_Envio`; sin datos del subcontratador la emisión se rechaza con `422` y el código `4424`.
- [ ] Con el indicador `SUNAT_Envio_IndicadorPagadorFlete_Tercero` se genera `cac:OriginatorCustomerParty`; sin indicador se devuelve `422` con el código `4388`.
- [ ] El XML generado nunca contiene `cac:TransportEvent/cbc:TransportEventTypeCode` (código `3374`).
- [ ] Se pueden agrupar dos GRE-Remitente de serie electrónica en una misma GRE-T; con una tercera de serie no electrónica se devuelve `422` con el código `3346`.
- [ ] 50 emisiones concurrentes sobre la misma serie `V###` producen 50 correlativos consecutivos sin huecos.
- [ ] Repetir la `Idempotency-Key` no crea un segundo documento.
- [ ] Un token sin el RUC del transportista recibe `403` y no se persiste nada.
- [ ] El XML descargado conserva el hash informado en la emisión.

## 14. Trazabilidad

| Requisito | Artefacto | Prueba |
|---|---|---|
| RF-1 | `SolicitudEmision` (schema.json) | `SolicitudEmisionTransportistaTest` — condicional de serie `V###` |
| RF-2 | `SerieCorrelativoRepository` | `EmisionConcurrenteTransportistaIT` — 50 hilos, misma serie `V###` |
| RF-3 | `ValidacionGreTransportista` | `ValidacionTransportistaTest` — tabla de ítems 1..64 |
| RF-4 | `GeneradorXmlTransportista` | `XmlTransportistaTest` — golden files contra XSD UBL |
| RF-5 | `FirmadorXml`, `EmpaquetadorZip` | `FirmaYZipTransportistaTest` — firma verificable y nombre de archivo `31` |
| RF-6 | `GreController` | `GreControllerTransportistaIT` — consulta, descarga y 404 cruzado |
| RF-7 | `EnvioRepository` | `EnvioTransportistaIT` — historial con códigos de retorno |
| RF-8 | `ClienteSunat` (feature 0005) | `EnvioResilienteTransportistaIT` — backoff y clasificación de errores |
| RF-9 | `GreGrpcService` | `GreGrpcServiceTransportistaIT` — paridad con REST |
| RF-10 | `AuditoriaAspecto` | `AuditoriaTransportistaTest` — registro por operación |
| RF-11 | `ConsignacionMapper`, `ConsignacionValidator` | `SubcontratacionTest` — presencia/ausencia de `cac:Consignment` y códigos `3390`, `3391`, `4424` |
| RF-12 | `PagadorFleteMapper`, `PagadorFleteValidator` | `PagadorFleteTest` — indicadores `4388`, `3344` y bloque `cac:OriginatorCustomerParty` |
| RF-13 | `DocumentoRelacionadoValidator` | `AgrupacionGuiasTest` — límites `3345`, `3346`, `3340` |
| RF-14 | `SerieService` | `SeriePrefijoTest` — rechazo de series `T###` para el tipo `31` |
| RF-15 | `ObservacionRepository` | `ObservacionesTransportistaIT` — persistencia de `ERROR` y `OBSERV` |

## 15. Riesgos y supuestos

| ID | Riesgo / supuesto | Impacto | Mitigación |
|---|---|---|---|
| R-1 | La SUNAT modifica el XLSX de validaciones sin aviso. | Alto | Importación versionada de catálogos (feature 0003) y pruebas de regresión sobre el XLSX vigente. |
| R-2 | Divergencia entre el JSON del API y el XML que exige SUNAT. | Alto | El XML se valida contra el XSD oficial en CI; los golden files se regeneran con el XLSX vigente. |
| R-3 | Duplicidad de correlativos bajo concurrencia sobre la serie `V###`. | Alto | Reserva transaccional con bloqueo de fila y restricción `UNIQUE`. |
| R-4 | El literal del indicador de subcontratación difiere entre el contrato XML (`SUNAT_Envio_IndicadorTrasporteSubcontratado`) y `schema.json` (`SUNAT_Envio_IndicadorTransporteSubcontratado`). | Alto | Verificar el literal en la hoja «Guía-Transportista2_0» antes del despliegue y unificar el contrato; queda registrado en la sección 16. |
| R-5 | Los ítems `[NO VERIFICADO]` del contrato XML (URIs de namespace, secuencia XSD, indicador M1/L) se interpretan de forma distinta a SUNAT. | Medio | Validar esos puntos contra el XLSX y el XSD oficial antes de la salida a producción. |
| R-6 | El padrón de vehículos, conductores, placas y RUC no es consultable localmente. | Medio | Los códigos que exigen esa consulta se reportan como observaciones o errores de SUNAT; se documenta la dependencia del WS de consulta de SUNAT. |
| R-7 | El cliente no dispone de certificado digital vigente del transportista. | Medio | Diagnóstico temprano en el pre-chequeo de la feature 0005. |
| R-8 | El agrupamiento de varias GRE-Remitente exige verificar en SUNAT el remitente, el destinatario, el punto de partida y el punto de llegada de cada una. | Medio | La plataforma no consulta la GRE-Remitente: la coherencia se delega a SUNAT, que responde `3433`/`3434`. Documentar el flujo de corrección. |
| S-1 | Se asume que el ERP del transportista puede serializar la cantidad con 10 decimales (`n(12,10)`). | — | El generador redondea y ajusta; se documenta la restricción. |
| S-2 | Se asume que el transportista conoce el RUC del remitente y del subcontratador en el momento de emitir. | — | Si no, la emisión se rechaza con `3383` o `4424`; el ERP debe enrichir la solicitud. |

## 16. Preguntas abiertas

- ¿Cuál es el literal exacto del indicador de transporte subcontratado que acepta el servicio de recepción de SUNAT: `SUNAT_Envio_IndicadorTrasporteSubcontratado` (como figura en [contrato-xml-ubl.md](contrato-xml-ubl.md), ítems 54 y 57) o `SUNAT_Envio_IndicadorTransporteSubcontratado` (como figura en [schema.json](../../contracts/schema.json))? (responsable: negocio + integración)
- ¿El servicio de recepción de SUNAT exige la credencial WS-Security del transportista como usuario secundario con perfil de «envío de comprobantes electrónicos», o admite el certificado del contribuyente? (responsable: integración)
- ¿Se habilita el uso de `SUNAT_Envio_IndicadorTrasladoVehiculoM1L` en la GRE-Transportista? En el contrato figura como **[NO VERIFICADO]** y no consta en la hoja «Guía-Transportista2_0». (responsable: negocio)
- ¿La plataforma debe resolver la consulta al padrón de SUNAT (RUC, DNI, placas, licencias) para evitar los rechazos `3389`, `3359`, `4398`, `4400` y `4412`, o se deja esa verificación exclusivamente en SUNAT? (responsable: negocio + arquitectura)
- ¿Se admite emisión en lote (varias GRE-T en una sola petición)? Queda fuera de alcance hasta que el volumen lo justifique. (responsable: producto)
