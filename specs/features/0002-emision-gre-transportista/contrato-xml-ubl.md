# Contrato XML UBL 2.1 — GRE de Remisión Electrónica Transportista (GRE-Transportista)

> Documento de contrato técnico. Fuente primaria: hoja **"Guía-Transportista2_0"** del archivo oficial
> `ValidacionesGREv20250421_0.xlsx` (SUNAT, versión 2025-04-21), complementada con la
> **R.S. N.° 123-2022/SUNAT** (Anexo VII, Anexo N.° 14 — estándar UBL 2.1 Guía de remisión remitente / transportista).
> Los tags, atributos, códigos de catálogo y códigos de retorno se reproducen **sin traducir**.
> Todo dato que no pudo ser verificado en las fuentes se marca con **[NO VERIFICADO]**.

---

## 1. Identificación

| Ítem | Valor |
|---|---|
| Documento | Guía de Remisión Electrónica – Transportista (GRE-Transportista) |
| Tipo de documento (Catálogo N.° 01) | `31` (`/DespatchAdvice/cbc:DespatchAdviceTypeCode`) |
| Nombre del archivo XML | `RUC-31-VAAA-CCCCCCCC-XML` (posiciones 1-11 RUC, 13-14 tipo, 16-19 serie, 21-28 correlativo, 30-32 extensión `XML`) |
| Nombre del archivo ZIP | `RUC-31-VAAA-CCCCCCCC-ZIP` (extensión `ZIP`) |
| Numeración / serie | `V###-NNNNNNNN` — formato exigido: `[V][A-Z0-9]{3}-[0-9]{1,8}` (campo 3) |
| Versión UBL | `2.1` → `/DespatchAdvice/cbc:UBLVersionID` (campo 1) |
| Versión de la estructura del documento | `2.0` → `/DespatchAdvice/cbc:CustomizationID` (campo 2) |
| Emisor del documento | Transportista → `/DespatchAdvice/cac:DespatchSupplierParty` (campo 8) |
| Emisor de la mercancía | Remitente → `/DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty` (campo 16) |
| Fecha / hora de emisión | `YYYY-MM-DD` (campo 4) / `hh:mm:ss` (campo 5) |
| Firma digital | Obligatoria (campo 64) en `ext:UBLExtensions/ext:UBLExtension/ext:ExtensionContent/ds:Signature` |
| Campos documentados | **64** (63 filas de tabla; los campos 27 y 28 comparten fila en la fuente) |
| Códigos de retorno cubiertos | **178 únicos** (265 ocurrencias) |
| Subsets de ítems | `cac:DespatchLine` (incremental) |
| Estándar base | UBL 2.1 – `DespatchAdvice` (SUNAT "Estándar UBL 2.1") |

---

## 2. Namespaces

Los prefijos se han deducido de las rutas de tags de la fuente (están escritos literalmente en la hoja
"Guía-Transportista2_0" y en el Anexo VII b) de la R.S. 123-2022). **Los URI exactos no aparecen en las
fuentes entregadas**, por lo que se marcan como no verificados.

| Prefijo | Namespace (URI) | Estado |
|---|---|---|
| (por defecto) | `urn:oasis:names:specification:ubl:schema:xsd:DespatchAdvice-2` | **[NO VERIFICADO]** — prefijo no declarado en las fuentes; se infiere por el nombre del elemento raíz `DespatchAdvice` |
| `cac` | `urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2` | **[NO VERIFICADO]** — prefijo confirmado por las rutas (`cac:Party`, `cac:Shipment`, …) |
| `cbc` | `urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2` | **[NO VERIFICADO]** — prefijo confirmado por las rutas (`cbc:ID`, `cbc:Name`, …) |
| `ext` | `urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2` | **[NO VERIFICADO]** — prefijo confirmado por la ruta de firma digital |
| `ds` | `http://www.w3.org/2000/09/xmldsig#` | **[NO VERIFICADO]** — prefijo confirmado por la ruta `ds:Signature` |

Atributos de catálogos usados en el documento (valores literales exigidos por la fuente):

| Atributo | Valor exigido |
|---|---|
| `@listAgencyName` | `PE:SUNAT` |
| `@listName` | `Tipo de Documento` (cat. 01), `Documento relacionado al transporte` (cat. 61), `Propiedad del item` (cat. 55), `Modalidad de traslado` (cat. 18, solo Remitente), `Motivo de traslado` (cat. N.° 20 / D05, solo Remitente) |
| `@listURI` | `urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo01`, `…:catalogo61`, `…:catalogo55`, `…:catalogo06` |
| `@schemeName` | `Documento de Identidad` |
| `@schemeAgencyName` | `PE:SUNAT` (documentos), `PE:INEI` (ubigeos) |
| `@schemeURI` | `urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06` |
| `@schemeID` | `6` (RUC), `1` (DNI), `GTIN-8` / `GTIN-12` / `GTIN-13` / `GTIN-14` |
| `@unitCode` | Catálogo N.° 03 / Catálogo N.° 65 |
| `@unitCodeListID` | `UN/ECE rec 20` |
| `@unitCodeListAgencyName` | `United Nations Economic Commission for Europe` |
| `@listID` (código producto SUNAT) | `UNSPSC` |
| `@listAgencyName` (código producto SUNAT) | `GS1 US` |

---

## 3. Estructura del documento (árbol)

Orden de lectura según los bloques de la guía oficial. Los elementos repetibles se marcan con `[0..n]`.
**La secuencia XSD no fue entregada en las fuentes** → el orden global de los hijos de `DespatchAdvice`
se indica como **[NO VERIFICADO]**; el ejemplo de la sección 6 sigue la secuencia canónica UBL 2.1.

```
DespatchAdvice
├── ext:UBLExtensions                                  [0..1]
│   └── ext:UBLExtension
│       └── ext:ExtensionContent
│           └── ds:Signature                            (campo 64 – Firma digital)
├── cbc:UBLVersionID                                    (campo 1  – 2.1)
├── cbc:CustomizationID                                 (campo 2  – 2.0)
├── cbc:ID                                              (campo 3  – V###-NNNNNNNN)
├── cbc:IssueDate                                       (campo 4  – YYYY-MM-DD)
├── cbc:IssueTime                                       (campo 5  – hh:mm:ss)
├── cbc:DespatchAdviceTypeCode                          (campo 6  – "31")
├── cbc:Note                                            (campo 7  – observaciones)
├── cac:DespatchSupplierParty                           (campos 8-11 – Transportista)
│   └── cac:Party
│       ├── cac:PartyIdentification
│       │   └── cbc:ID[@schemeID|@schemeName|@schemeAgencyName|@schemeURI]
│       └── cac:PartyLegalEntity
│           └── cbc:RegistrationName
├── cac:DeliveryCustomerParty                           (campos 18-19 – Destinatario)
│   └── cac:Party
│       ├── cac:PartyIdentification
│       │   └── cbc:ID[@schemeID …]
│       └── cac:PartyLegalEntity
│           └── cbc:RegistrationName
├── cac:OriginatorCustomerParty                         (campos 62-63 – quien paga el servicio)
│   └── cac:Party
│       ├── cac:PartyIdentification
│       │   └── cbc:ID[@schemeID …]
│       └── cac:PartyLegalEntity
│           └── cbc:RegistrationName
├── cac:Shipment
│   ├── cbc:ID                                          (campo 49 – SUNAT_Envio)
│   ├── cbc:GrossWeightMeasure[@unitCode]               (campo 52 – peso bruto total)
│   ├── cbc:SpecialInstructions                         (campos 53-58 – indicadores SUNAT_Envio_*)
│   ├── cac:ShipmentStage
│   │   ├── cac:CarrierParty                            (campos 10-11 – Registro MTC, entidad autorizadora)
│   │   │   └── cac:PartyLegalEntity
│   │   │       ├── cbc:CompanyID
│   │   │       └── cac:AgentParty
│   │   │           └── cac:PartyLegalEntity
│   │   │               └── cbc:CompanyID[@schemeID]
│   │   ├── cac:DriverPerson                            (campos 41-44 – conductor principal)
│   │   │   ├── cbc:JobTitle                            ('Principal')
│   │   │   ├── cbc:ID[@schemeID …]
│   │   │   ├── cbc:FirstName
│   │   │   ├── cbc:FamilyName
│   │   │   └── cac:IdentityDocumentReference
│   │   │       └── cbc:ID                              (licencia de conducir)
│   │   ├── cac:TransitPeriod
│   │   │   └── cbc:StartDate                           (campo 50 – fecha inicio de traslado)
│   │   └── cac:TransportEvent                          (campo 59 – no debe enviarse)
│   │       └── cbc:TransportEventTypeCode
│   ├── cac:Delivery
│   │   ├── cac:Despatch                                (campos 16-17, 29-31 – Remitente y punto de partida)
│   │   │   ├── cac:DespatchParty
│   │   │   │   ├── cac:PartyIdentification
│   │   │   │   │   └── cbc:ID[@schemeID …]
│   │   │   │   └── cac:PartyLegalEntity
│   │   │   │       └── cbc:RegistrationName
│   │   │   └── cac:DespatchAddress
│   │   │       ├── cbc:ID                              (ubigeo de partida)
│   │   │       ├── cac:AddressLine
│   │   │       │   └── cbc:Line
│   │   │       └── cac:LocationCoordinate
│   │   │           ├── cbc:LatitudeDegreesMeasure[@unitCode='DD']
│   │   │           └── cbc:LongitudeDegreesMeasure[@unitCode='DD']
│   │   └── cac:DeliveryAddress                         (campos 32-34 – punto de llegada)
│   │       ├── cbc:ID
│   │       ├── cac:AddressLine
│   │       │   └── cbc:Line
│   │       └── cac:LocationCoordinate
│   │           ├── cbc:LatitudeDegreesMeasure[@unitCode='DD']
│   │           └── cbc:LongitudeDegreesMeasure[@unitCode='DD']
│   ├── cac:TransportHandlingUnit                       (campos 35-40 – vehículo principal y secundarios)
│   │   └── cac:TransportEquipment
│   │       ├── cbc:ID                                  (placa)
│   │       ├── cac:ApplicableTransportMeans
│   │       │   └── cbc:RegistrationNationalityID       (TUC / Constancia de Inscripción Vehicular)
│   │       ├── cac:ShipmentDocumentReference
│   │       │   └── cbc:ID[@schemeID]                   (autorización especial del vehículo)
│   │       └── cac:AttachedTransportEquipment          [0..2]
│   │           ├── cbc:ID
│   │           ├── cac:ApplicableTransportMeans
│   │           │   └── cbc:RegistrationNationalityID
│   │           └── cac:ShipmentDocumentReference
│   │               └── cbc:ID[@schemeID]
│   └── cac:Consignment                                 (campos 60-61 – subcontratación)
│       ├── cbc:ID                                      (identificador obligatorio)
│       └── cac:LogisticsOperatorParty
│           ├── cac:PartyIdentification
│           │   └── cbc:ID[@schemeID]
│           └── cac:PartyLegalEntity
│               └── cbc:RegistrationName
├── cac:AdditionalDocumentReference                     [1..n] (campos 12-15 – documento relacionado)
│   ├── cbc:DocumentType
│   ├── cbc:DocumentTypeCode[@listAgencyName|@listName|@listURI]
│   ├── cbc:ID
│   └── cac:IssuerParty
│       └── cac:PartyIdentification
│           └── cbc:ID[@schemeID …]
└── cac:DespatchLine                                   [1..n] (campos 20-28 – bienes a transportar)
    ├── cbc:ID                                          (número de orden del ítem)
    ├── cac:OrderLineReference
    │   └── cbc:LineID
    ├── cbc:DeliveredQuantity[@unitCode|@unitCodeListID|@unitCodeListAgencyName]
    └── cac:Item
        ├── cbc:Description
        ├── cbc:SellersItemIdentification
        │   └── cbc:ID
        ├── cbc:CommodityClassification
        │   └── cbc:ItemClassificationCode[@listID|@listAgencyName|@listName]
        ├── cac:StandardItemIdentification
        │   └── cbc:ID[@schemeID]
        └── cac:AdditionalItemProperty
            ├── cbc:Name
            ├── cbc:NameCode[@listName|@listAgencyName|@listURI]
            └── cbc:Value
```

---

## 4. Tabla campo por campo

**Convenciones**

- **Nivel**: `Global` (una vez por documento) o `Ítem` (una vez por `cac:DespatchLine`).
- **Cond.**: `M` = obligatorio (Mandatory), `C` = condicional (Conditional), según la columna
  "CONDICIÓN INFORMÁTICA" de la fuente.
- **Tipo y longitud**: notación SUNAT: `an..N` = alfanumérico de hasta N caracteres, `n..N` = numérico,
  `n(12,10)` = numérico con 12 enteros y hasta 10 decimales, `an.50` = longitud fija.
- **Formato**: valores literales exigidos; `|` separa valores alternativos; los `|` dentro de la celda
  se escapan como `\|` para no romper la tabla.
- **Tag UBL**: ruta completa desde `/DespatchAdvice`; las filas de continuación del mismo campo se
  acumulan en la misma celda separadas por `<br>`.
- **Condiciones de validación**: texto literal de la columna "VALIDACIÓN / CONDICIÓN" de la fuente.
- **Tipo de retorno**: `ERROR`, `OBSERV`, `ERROR / OBSERV` o `-` cuando la fuente indica
  `<<< SIN VALIDACIÓN >>>`.
- **Códigos de retorno**: código (4 dígitos) + severidad + mensaje literal.

**Datos de guía de remisión**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Version del UBL | Global | M | an..3 | 2.1 | /DespatchAdvice/cbc:UBLVersionID | No existe el Tag UBL o es vacío<br>El valor del Tag UBL es diferente a "2.1" | ERROR | 2111 (ERROR) El XML no contiene el tag o no existe informacion de UBLVersionID<br>2110 (ERROR) UBLVersionID - La versión del UBL no es correcta |
| 2 | Version de la estructura del documento | Global | M | an..3 | 2.0 | /DespatchAdvice/cbc:CustomizationID | No existe el Tag UBL o es vacío<br>El valor del Tag UBL es diferente a "2.0" | ERROR | 2113 (ERROR) El XML no contiene el tag o no existe informacion de CustomizationID<br>2112 (ERROR) CustomizationID - La version del documento no es correcta |
| 3 | Numeracion, conformada por serie y Número correlativo | Global | M | an..13 | V###-NNNNNNNN | /DespatchAdvice/cbc:ID | El formato del Tag UBL es diferente a: <br> [V][A-Z0-9]{3}-[0-9]{1,8}<br>El número de serie del Tag UBL es diferente al número de serie del archivo<br>El número de documento del Tag UBL es diferente al número de documento del archivo<br>El valor del Tag UBL se encuentra en el listado con indicador de estado igual a '1'<br>El valor del Tag UBL se encuentra en el listado con indicador de estado igual a '2' | ERROR | 1001 (ERROR) ID - El dato SERIE-CORRELATIVO no cumple con el formato de acuerdo al tipo de comprobante<br>1035 (ERROR) Numero de Serie del nombre del archivo no coincide con el consignado en el contenido del archivo XML<br>1036 (ERROR) Número de documento en el nombre del archivo no coincide con el consignado en el contenido del XML<br>1033 (ERROR) El comprobante fue registrado previamente con otros datos<br>1032 (ERROR) El comprobante ya esta informado y se encuentra con estado anulado o rechazado |
| 4 | Fecha de emisión | Global | M | an10 | YYYY-MM-DD | /DespatchAdvice/cbc:IssueDate | El valor del Tag UBL no cumple con el formato establecido<br>La diferencia entre la fecha de recepción del XML y el valor del Tag UBL es mayor al límite del listado (1 día)<br>El valor del Tag UBL es mayor a la fecha de recepción (TODAY) o es 12/07/2022 | ERROR | 3436 (ERROR) El campo de fecha de emision no cumple con el formato establecido<br>2108 (ERROR) Presentacion fuera de fecha<br>2329 (ERROR) La fecha de emision se encuentra fuera del limite permitido |
| 5 | Hora de emisión | Global | M | an8 | hh:mm:ss | /DespatchAdvice/cbc:IssueTime | No existe el Tag UBL<br>El valor del Tag UBL no cumple con el formato establecido | ERROR | 3437 (ERROR) No existe informacion en el campo de hora de emision<br>3438 (ERROR) El campo de hora de emision no cumple con el formato establecido |
| 6 | Tipo de documento (Guia) | Global | M | an..2 | "31" \| "PE:SUNAT" \| "Tipo de Documento" \| "urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo01" | /DespatchAdvice/cbc:DespatchAdviceTypeCode<br>@listAgencyName<br>@listName<br>@listURI | No existe el Tag UBL o es vacío<br>El valor del Tag UBL es diferente a "31"<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT'<br>Si existe el atributo, el valor ingresado es diferente a 'Tipo de Documento'<br>Si existe el atributo, el valor ingresado es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo01' | ERROR / OBSERV | 1050 (ERROR) El XML no contiene informacion en el tag DespatchAdviceTypeCode.<br>1051 (ERROR) DespatchAdviceTypeCode - El valor del tipo de guía es inválido.<br>4251 (OBSERV) El dato ingresado como atributo @listAgencyName es incorrecto.<br>4252 (OBSERV) El dato ingresado como atributo @listName es incorrecto.<br>4253 (OBSERV) El dato ingresado como atributo @listURI es incorrecto. |
| 7 | Observaciones (Texto) | Global | C | an..250 |  | /DespatchAdvice/cbc:Note | Si el Tag UBL existe, el formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres o es vacío (se considera cualquier carácter incluido espacio, no se permite ningún otro "whitespace character": salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | OBSERV | 4186 (OBSERV) cbc:Note - El campo observaciones supera la cantidad maxima especificada (250 carácteres). |

**Datos del Transportista**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 8 | Tipo y número de documento de identidad del transportista | Global | M | an1 \| an11 | "6" \| "Documento de Identidad" \| "PE:SUNAT" \| "urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06" \| n11 | /DespatchAdvice/cac:DespatchSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID@schemeID<br>@schemeName<br>@schemeAgencyName<br>@schemeURI<br>/DespatchAdvice/cac:DespatchSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID | No existe el Tag UBL o es vacío<br>El valor del Tag UBL es diferente a "6"<br>Si existe el atributo, el valor ingresado es diferente a 'Documento de Identidad'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT'<br>Si existe el atributo, el valor ingresado es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06'<br>El Tag UBL es diferente al RUC del nombre del XML<br>El valor del Tag UBL tiene un ind_estado diferente "00" en el listado<br>El valor del Tag UBL tiene un ind_condicion diferente a "00" en el listado<br>Se encuentra afecto exclusivamente solo a los tributos Renta de primera categoría, Renta de segunda categoría o Renta de cuarta categoría (no está afecto a ningún otro tributo distinto de los indicados) <br> | ERROR / OBSERV | 2678 (ERROR) El XML no contiene el atributo o no existe información del tipo de documento del emisor<br>2511 (ERROR) El tipo de documento no es aceptado.<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto.<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto.<br>4257 (OBSERV) El dato ingresado como atributo @schemeURI es incorrecto.<br>1034 (ERROR) Número de RUC del nombre del archivo no coincide con el consignado en el contenido del archivo XML<br>3349 (ERROR) El RUC del Transportista no esta activo<br>3350 (ERROR) El RUC del Transportista no esta habido<br>3439 (ERROR) El RUC no esta afecto a los tributos autorizados |
| 9 | Apellidos y Nombres o denominacion o razón social del transportista | Global | M | an..250 |  | /DespatchAdvice/cac:DespatchSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName | No existe el Tag UBL o es vacío<br>El formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro "whitespace character": salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | ERROR / OBSERV | 1037 (ERROR) El XML no contiene el tag o no existe informacion de RegistrationName del emisor del documento<br>4338 (OBSERV) RegistrationName - El nombre o razon social del emisor no cumple con el estandar |
| 10 | Número de Registro MTC | Global | C | an..20 |  | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:PartyLegalEntity/cbc:CompanyID | No existe el Tag UBL o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 20 caracteres (solo se permite letras mayúsculas y números, no se permite ningún otro 'whitespace character': espacio, salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco o solamente ceros)<br>Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no se encuentra en el listado <br><br>Si el Tag UBL existe y es diferente de vacío, existe otro cac:PartyLegalEntity | OBSERV / ERROR | 4391 (OBSERV) El Numero de Registro MTC del transportista no existe o no contiene informacion<br>4392 (OBSERV) El Numero de Registro MTC del transportista no cumple con el formato establecido<br>4393 (OBSERV) El Numero de Registro MTC del transportista no se encuentra en las bases consultadas<br>3353 (ERROR) Solo debe consignar un Numero de autorizacion |
| 11 | Entidad emisora de la autorización especial - Transportista <br> <br> Número de autorización especial emitido por la entidad - Transportista | Global | C | an..50 \| an2 | \| (Catálogo N° D-37) \| "Entidad Autorizadora" \| "PE:SUNAT" | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:AgentParty/cac:PartyLegalEntity/cbc:CompanyID (Número de autorización)<br>/DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:AgentParty/cac:PartyLegalEntity/cbc:CompanyID@schemeID (Código de entidad autorizadora)<br>@schemeName<br>@schemeAgencyName | Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 3 a 50 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro 'whitespace character': salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco)<br>Si el Tag UBL existe y es diferente de vacío, existe otro cac:PartyLegalEntity<br>Si existe 'Número de autorización' con valor diferente de vacío, no existe el atributo o es vacío<br>Si existe el atributo, el valor ingresado no se encuentra en el listado y es diferente de vacío<br>Si existe el atributo y es diferente de vacío, el 'Número de autorización' es vacío<br>Si existe el atributo, el valor ingresado es diferente a 'Entidad Autorizadora'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT' | OBSERV / ERROR | 4396 (OBSERV) El Numero de autorizacion del transportista no cumple con el formato establecido<br>3353 (ERROR) Solo debe consignar un Numero de autorizacion<br>4394 (OBSERV) Debe indicar la entidad autorizadora de la autorizacion especial<br>4395 (OBSERV) El Codigo de entidad autorizadora es invalido<br>4397 (OBSERV) SI existe el Codigo de entidad autorizadora, debe consignar el Numero de autorizacion especial de la empresa<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto.<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto. |

**Documento relacionado - solo se permite un documento, excepto hasta 2 si se ingresa alguno de los códigos "31", "65", "66", "67", "68", "69", o más de 2 si se ingresa código "09" con serie electrónica**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 12 | Tipo de documento (Descripción) | Global | C | an..120 |  | /DespatchAdvice/cac:AdditionalDocumentReference/cbc:DocumentType | Si existe 'Código del tipo de documento relacionado' y es diferente de vacío, y no existe el Tag UBL o es vacío<br>Si el Tag UBL existe y es diferente de vacío, no existe 'Código del tipo de documento relacionado' o es vacío<br>El formato es diferente a alfanumérico de hasta 120 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro "whitespace character": salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | OBSERV | 4371 (OBSERV) El XML no contiene el tag o no existe informacion de la descripcion del tipo de documento relacionado<br>4410 (OBSERV) Si existe descripcion del documento relacionado debe existir informacion del codigo y numero del documento relacionado<br>4372 (OBSERV) La descripcion del tipo de documento relacionado no cumple con el estandar |
| 13 | Código del tipo de documento relacionado | Global | C | an2 | (Catálogo N° 61) \| "PE:SUNAT" \| "Documento relacionado al transporte" \| "urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo61" | /DespatchAdvice/cac:AdditionalDocumentReference/cbc:DocumentTypeCode<br>@listAgencyName<br>@listName<br>@listURI | Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no está en el listado de acuerdo al tipo de guía<br>Si el Tag UBL existe y es diferente de vacío, el 'Número de documento relacionado' es vacío<br>Si existe un 'Documento relacionado' con 'Código del tipo de documento relacionado' igual a "31", "65", "66", "67", "68", "69", existe más de dos 'Documento relacionado' (cac:AdditionalDocumentReference con cbc:DocumentTypeCode diferente de vacío)<br>Si no existe un 'Documento relacionado' con 'Código del tipo de documento relacionado' igual a "31", "65", "66", "67", "68", "69", y no existe un 'Documento relacionado' con 'Código del tipo de documento relacionado' igual a "09" con número de serie que no empieza con número, existe más de un 'Documento relacionado' (cac:AdditionalDocumentReference con cbc:DocumentTypeCode diferente de vacío)<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT'<br>Si existe el atributo, el valor ingresado es diferente a 'Documento relacionado al transporte'<br>Si existe el atributo, el valor ingresado es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo61' | ERROR / OBSERV | 2692 (ERROR) El tipo de documento relacionado no es válido<br>3403 (ERROR) El XML no contiene el tag o no existe informacion del numero de documento relacionado<br>3345 (ERROR) Solo corresponde ingresar hasta dos 2 documentos relacionados si uno de los tipos de documentos relacionados es "31", "65", "66", "67", "68" o "69"<br>3346 (ERROR) Solo corresponde ingresar mas de 1 documento relacionado si uno de los tipos de documentos relacionados es "31", "65", "66", "67", "68" o "69", o "09" (solo guia electronica)<br>4251 (OBSERV) El dato ingresado como atributo @listAgencyName es incorrecto.<br>4252 (OBSERV) El dato ingresado como atributo @listName es incorrecto.<br>4253 (OBSERV) El dato ingresado como atributo @listURI es incorrecto. |
| 14 | Número de documento relacionado | Global | C | an..100 |  | /DespatchAdvice/cac:AdditionalDocumentReference/cbc:ID | Si el valor del Tag UBL es diferente de vacío, el 'Código del tipo de documento relacionado' concatenado con el valor del Tag UBL se repite en el /DespatchAdvice<br>Si el valor del Tag UBL es diferente de vacío, el 'Código del tipo de documento relacionado' no existe o es vacío<br>Si el 'Código del tipo de documento relacionado' es '01', el formato del Tag UBL es diferente a <Serie>-<Numero> con la siguiente estructura: <br> - [F][A-Z0-9]{3}-[0-9]{1,8} <br> - (E001)-[0-9]{1,8} <br> - [0-9]{1,4}-[0-9]{1,8} <br> El campo <Numero> debe ser mayor a cero<br>Si el 'Código del tipo de documento relacionado' es '03', el formato del Tag UBL es diferente a <Serie>-<Numero> con la siguiente estructura: <br> - [B][A-Z0-9]{3}-[0-9]{1,8} <br> - (EB01)-[0-9]{1,8} <br> - [0-9]{1,4}-[0-9]{1,8} <br> El campo <Numero> debe ser mayor a cero<br>Si el 'Código del tipo de documento relacionado' es '04', el formato del Tag UBL es diferente a <Serie>-<Numero> con la siguiente estructura: <br> - [L][A-Z0-9]{3}-[0-9]{1,8} <br> - (E001)-[0-9]{1,8} <br> - [0-9]{1,4}-[0-9]{1,8} <br> El campo <Numero> debe ser mayor a cero<br>Si el 'Código del tipo de documento relacionado' es '50' o '52', el formato del Tag UBL es diferente de: <br> - [0-9]{3}-[0-9]{4}-[0-9]{2}-[0-9]{1,6} <br> donde: [0-9]{3}: Código de la Aduana, [0-9]{4}: Año, [0-9]{2} Régimen aduanero, [0-9]{1,6} Correlativo<br>Si el 'Código del tipo de documento relacionado' es '80', el formato del Tag UBL es diferente a numérico hasta 15 dígitos (debe ser mayor a cero)<br>Si el 'Código del tipo de documento relacionado' es '12', el formato del Tag UBL es diferente a <Serie>-<Numero> con la siguiente estructura: <br> - [a-zA-Z0-9-]{1,20}-[a-zA-Z0-9-]{1,20}<br>Si el 'Código del tipo de documento relacionado' es '48', el formato del Tag UBL es diferente a <Serie>-<Numero> con la siguiente estructura: <br> - [0-9]{1,4}-[0-9]{1,7} <br> El campo <Numero> debe ser mayor a cero<br>Si el 'Código del tipo de documento relacionado' es '09', el formato del Tag UBL es diferente a <Serie>-<Numero> con la siguiente estructura: <br> - [T][A-Z0-9]{3}-[0-9]{1,8} <br> - (EG07)-[0-9]{1,8} <br> - (EG02)-[0-9]{1,8} <br> - [0-9]{1,4}-[0-9]{1,8} <br> El campo <Numero> debe ser mayor a cero<br>Si el 'Código del tipo de documento relacionado' es '31', el formato del Tag UBL es diferente a <Serie>-<Numero> con la siguiente estructura: <br> - [V][A-Z0-9]{3}-[0-9]{1,8} <br> - (EG03)-[0-9]{1,8} <br> - (EG04)-[0-9]{1,8} <br> El campo <Numero> debe ser mayor a cero<br>Si el 'Código del tipo de documento relacionado' es '82', '65', '66', '67', '68' o '69', el formato del Tag UBL es diferente a alfanumérico de hasta 100 caracteres (se considera cualquier carácter, no permite 'whitespace character': espacio, salto de línea, fin de línea, tab, etc.)<br>Si el 'Código de tipo de documento' es '50' o '52', el valor del Tag UBL no se encuentra en el listado <br><br>Si el 'Código de tipo de documento' es '31', el valor del Tag UBL no se encuentra en el listado con estado de "Baja - por cambio de destinatario" <br><br>Si el 'Código de tipo de documento' es '80' y se encuentra referenciada en otras GRE - remitente o GRE - transportista según el listado en los últimos 15 días calendario contados a partir de la fecha de recepción de la GRE<br>Si existe el Tag UBL y el código de tipo de documento es '80', el valor del Tag UBL no se encuentra en el listado en los últimos 15 días calendario contados a partir de la fecha de recepción de la GRE <br><br>Si el 'Código de tipo de documento' es '09' y la serie no empieza con número, el valor del Tag UBL no se encuentra en el listado para el emisor de documento<br>Si el 'Código de tipo de documento' es '01', '03' o '04' y la serie no empieza con número, el valor del Tag UBL no se encuentra en el listado para el emisor de documento<br>Si el 'Código de tipo de documento' es '01', '03', '04', '09' o '48' y la serie empieza con número, el valor del Tag UBL no se encuentra en el listado para el emisor de documento | ERROR / OBSERV | 3340 (ERROR) El documento relacionado al traslado de mercancias se encuentra duplicado<br>3376 (ERROR) El XML no contiene el tag o no existe informacion del Codigo de tipo de documento relacionado<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3441 (ERROR) El numero de documento relacionado al traslado de mercancias no cumple con el formato establecido<br>3341 (ERROR) No existe el numero de Declaracion Aduanera de Mercancias (DAM) o el Declaracion Simplificada (DS)<br>2536 (ERROR) La serie y numero de la GRE consignada como documento relacionado no se encuentra registrado con baja por cambio de destinatario<br>3342 (ERROR) La Constancia de Deposito ingresada ya ha sido referenciada en otra GRE emitida<br>4404 (OBSERV) La SUNAT no cuenta con la informacion en los ultimos 15 dias de la Constancia de Deposito en sus sistemas<br>3433 (ERROR) La GRE remitente no existe<br>4382 (OBSERV) El numero de comprobante consignado como documento relacionado no existe<br>3207 (ERROR) Comprobante físico no se encuentra autorizado |
| 15 | Número de RUC del emisor del documento relacionado | Global | C | an11 \| an1 | n11 \| "6" \| "Documento de Identidad" \| "PE:SUNAT" \| "urn:pe:gob:sunat:cpe:see:gem:catalogos: <br> catalogo06" | /DespatchAdvice/cac:AdditionalDocumentReference/cac:IssuerParty/cac:PartyIdentification/cbc:ID (Número de RUC del emisor del documento)<br>/DespatchAdvice/cac:AdditionalDocumentReference/cac:IssuerParty/cac:PartyIdentification/cbc:ID@schemeID (Tipo de documento de identidad del emisor del documento relacionado)<br>@schemeName<br>@schemeAgencyName<br>@schemeURI | Si el 'Código del tipo de documento relacionado' es '01', '03', '04', '12', '48', '09', '31' no existe el Tag UBL o es vacío<br>Si el Tag UBL existe y es diferente de vacío y el 'Código del tipo de documento relacionado' es '09', y la serie no empieza con número, el valor del Tag UBL es diferente al RUC del remitente<br>Si el Tag UBL existe y es diferente de vacío y el 'Código del tipo de documento relacionado' es '31', y la serie no empieza con número, el valor del Tag UBL es diferente al RUC del transportista<br>Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no existe en el listado<br>Si el 'Código del tipo de documento relacionado' es '01', '03', '04', '12', '48', '09', '31', el atributo del atributo no existe o es diferente a 6 (RUC)<br>Si existe el atributo, el valor ingresado es diferente a 'Documento de Identidad'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT'<br>Si existe el atributo, el valor ingresado es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06' | ERROR / OBSERV | 3380 (ERROR) Debe consignar el RUC del emisor del documento relacionado<br>3381 (ERROR) El RUC del emisor del documento relacionado no corresponde<br>3381 (ERROR) El RUC del emisor del documento relacionado no corresponde<br>3408 (ERROR) El Numero de RUC del emisor del documento relacionado no existe<br>3382 (ERROR) El Tipo de documento del emisor del documento relacionado debe ser Numero de RUC<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto.<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto.<br>4257 (OBSERV) El dato ingresado como atributo @schemeURI es incorrecto. |

**Datos del Remitente**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 16 | Tipo y número de documento de identidad del remitente | Global | M | an1 \| an..15 | (Catálogo N° 06) \| "Documento de Identidad" \| "PE:SUNAT" \| "urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06" | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty/cac:PartyIdentification/cbc:ID@schemeID (Tipo de documento de identidad)<br>@schemeName<br>@schemeAgencyName<br>@schemeURI<br>/DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty/cac:PartyIdentification/cbc:ID (Número de documento de identidad) | No existe el atributo o es vacío<br>El valor del atributo no está en el listado<br>Si existe el atributo, el valor ingresado es diferente a 'Documento de Identidad'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT'<br>Si existe el atributo, el valor ingresado es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06'<br>No existe el Tag UBL o es vacío<br>Si el tipo y número de documento de identidad del tranportista es el mismo que el remitente<br>Si "Tipo de documento de identidad del remitente" es "6" y el formato del Tag UBL es diferente a numérico de 11 dígitos<br>Si "Tipo de documento de identidad del remitente" es "6", el valor del Tag UBL no se encuentra en el listado<br>Si 'Tipo de documento de identidad del remitente' es "6" (RUC), el valor del Tag UBL tiene un ind_estado diferente "00" en el listado<br>Si 'Tipo de documento de identidad del remitente' es "6" (RUC), el valor del Tag UBL tiene un ind_condicion diferente a "00" en el listado<br>Si 'Tipo de documento de identidad del destinatario' es '1' y el fomato del Tag UBL es diferente a numérico de 8 dígitos<br>Si 'Tipo de documento de identidad del remitente' es '1', el valor del Tag UBL no se encuentra en base RENIEC <br><br>Si 'Tipo de documento de identidad del remitente' es diferente a '1' y diferente de '6', el formato del Tag UBL es diferente a alfanumérico de hasta 15 caracteres (se considera cualquier carácter, no permite 'whitespace character': espacio, salto de línea, fin de línea, tab, etc.) | ERROR / OBSERV | 2541 (ERROR) El XML no contiene el tag o no existe informacion del tipo de documento identidad del remitente.<br>2542 (ERROR) cac:DespatchSupplierParty/cbc:CustomerAssignedAccountID@schemeID - El valor ingresado como tipo de documento identidad del remitente es incorrecta.<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto.<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto.<br>4257 (OBSERV) El dato ingresado como atributo @schemeURI es incorrecto.<br>3383 (ERROR) Debe consignar el Numero de documento de identidad del Remitente<br>2560 (ERROR) Transportista no debe ser igual al remitente<br>3384 (ERROR) El Numero de documento de identidad del Remitente no cumple con el formato establecido<br>3385 (ERROR) El Numero de RUC del Remitente no existe<br>4420 (OBSERV) El Numero de RUC del Remitente no esta activo<br>4421 (OBSERV) El Numero de RUC del Remitente no esta habido<br>3384 (ERROR) El Numero de documento de identidad del Remitente no cumple con el formato establecido<br>3386 (ERROR) El Numero de DNI del Remitente no existe<br>3384 (ERROR) El Numero de documento de identidad del Remitente no cumple con el formato establecido |
| 17 | Apellidos y nombres, denominacion o razón social del remitente | Global | M | an..250 |  | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty/cac:PartyLegalEntity/cbc:RegistrationName | No existe el Tag UBL o es vacío<br>El formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro "whitespace character": salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | ERROR / OBSERV | 3387 (ERROR) Debe consignar el Nombre o razon social del Remitente<br>4422 (OBSERV) El nombre o razon social del Remitente no cumple con el formato establecido |

**Datos del Destinatario**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 18 | Tipo y número de documento de identidad del destinatario | Global | M | an1 \| an..15 | (Catálogo N° 06) \| "Documento de Identidad" \| "PE:SUNAT" \| "urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06" | /DespatchAdvice/cac:DeliveryCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID@schemeID (Tipo de documento de identidad)<br>@schemeName<br>@schemeAgencyName<br>@schemeURI<br>/DespatchAdvice/cac:DeliveryCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID (Número de documento de identidad) | Si el 'Número de documento del destinatario' es diferente de vacío, no existe el atributo o es vacío<br>Si el atributo existe y es diferente de vacío, el valor del atributo no está en el listado<br>Si existe el atributo, el valor ingresado es diferente a 'Documento de Identidad'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT'<br>Si existe el atributo, el valor ingresado es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06'<br>Si no existe el Tag UBL o es vacío<br>Si 'Tipo de documento de identidad del destinatario' es "6", el formato del Tag UBL es diferente a numérico de 11 dígitos<br>Si 'Tipo de documento de identidad del destinatario' es "6", el valor del Tag UBL no se encuentra en el listado<br>Si 'Tipo de documento de identidad del destinatario' es "6" (RUC), el valor del Tag UBL tiene un ind_estado diferente "00" en el listado<br>Si 'Tipo de documento de identidad del destinatario' es "6" (RUC), el valor del Tag UBL tiene un ind_condicion diferente a "00" en el listado<br>Si "Tipo de documento de identidad del destinatario" es "1" y el formato del Tag UBL es diferente a numérico de 8 dígitos<br>Si 'Tipo de documento de identidad del destinatario' es "1", el valor del Tag UBL no se encuentra en el listado<br>Si 'Tipo de documento de identidad del destinatario' es diferente a '1' y diferente de '6', el formato del Tag UBL es diferente a alfanumérico de hasta 15 caracteres (se considera cualquier carácter, no permite 'whitespace character': espacio, salto de línea, fin de línea, tab, etc.)<br>Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09' y la serie no empieza con número y consigna 'Motivo de traslado' diferente a '18', el valor del Tag UBL es diferente al 'tipo y número de documento de identidad del destinatario' en dicho documento | ERROR / OBSERV | 2759 (ERROR) El XML no contiene el atributo o no existe información del tipo de documento del destinatario.<br>2760 (ERROR) El valor ingresado como tipo de documento del destinatario es incorrecto.<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto.<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto.<br>4257 (OBSERV) El dato ingresado como atributo @schemeURI es incorrecto.<br>2757 (ERROR) El XML no contiene el tag o no existe información del número de documento de identidad del destinatario.<br>2758 (ERROR) El valor ingresado como numero de documento de identidad del destinatario no cumple con el estandar.<br>3443 (ERROR) El Numero de RUC del Destinatario no existe<br>4373 (OBSERV) El RUC del Destinatario no esta activo<br>4374 (OBSERV) El RUC del destinatario no esta habido<br>2758 (ERROR) El valor ingresado como numero de documento de identidad del destinatario no cumple con el estandar.<br>3444 (ERROR) El Numero de DNI del Destinatario no existe<br>2758 (ERROR) El valor ingresado como numero de documento de identidad del destinatario no cumple con el estandar.<br>3434 (ERROR) Si existe una GRE Remitente como documento relacionado, el tipo y numero de documento de identidad del Destinatario deben ser los mismos que del destinatario consignado en la GRE Remitente |
| 19 | Apellidos y nombres, denominacion o razón social del destinatario | Global | M | an..250 |  | /DespatchAdvice/cac:DeliveryCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName | Si no existe el Tag UBL o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro "whitespace character": salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | ERROR / OBSERV | 2761 (ERROR) El XML no contiene el atributo o no existe información del nombre o razon social del destinatario.<br>4152 (OBSERV) Nombre o razon social del destinatario no cumple con un formato válido |

**Bienes a transportar**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 20 | Número de orden del item | Ítem | C | n..4 |  | /DespatchAdvice/cac:DespatchLine/cbc:ID<br>/DespatchAdvice/cac:DespatchLine/cac:OrderLineReference/cbc:LineID | (Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' y no existe el 'Indicador de traslado total de bienes') <br> o <br> (no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' o '09' o '82'), no existe al menos un Bien a transportar con 'Cantidad' mayor a cero (cac:DespatchLine con cbc:DeliveredQuantity mayor a cero) <br><br>(Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09', con serie que no empieza con número) <br> o <br> (si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01' o '04' con serie que no empieza con número y existe el 'Indicador de traslado total de bienes') <br> o <br> (si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '50' o '52' y existe el 'Indicador de traslado total de bienes'), y existe algún Bien a transportar con 'Cantidad' mayor a cero (cac:DespatchLine con cbc:DeliveredQuantity mayor a cero) <br><br>El formato del Tag UBL es diferente de numérico de hasta 4 dígitos<br>(Si no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09' con serie que empieza con número) <br> Y <br> (Si no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '82') <br> Y <br> (Si no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01' o '04' con serie que empieza con número o no existe 'Indicador de traslado total de bienes') <br> Y <br> (Si no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '03', '12' o '48' o no existe 'Indicador de traslado total de bienes'), <br> el Tag UBL existe con valor "0"<br>El valor del Tag UBL se repite en el /DespatchAdvice<br><<< SIN VALIDACIÓN >>> | ERROR / OBSERV / - | 3435 (ERROR) Debe consignar el detalle de los bienes a transportar<br>4434 (OBSERV) No corresponde consignar el detalle de los bienes a transportar<br>2023 (ERROR) El Numero de orden del item no cumple con el formato establecido<br>3458 (ERROR) El numero de item debe ser mayor a cero. Solo se permite colocar cero para consignar una anotacion opcional sobre los bienes a transportar.<br>2752 (ERROR) El número de ítem no puede estar duplicado.<br>(sin validacion) <<< SIN VALIDACIÓN >>> |
| 21 | Cantidad | Ítem | C | an..23 | n(12,10) | /DespatchAdvice/cac:DespatchLine/cbc:DeliveredQuantity | (Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' y no existe el 'Indicador de traslado total de bienes') <br> o <br> (no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' o '09' o '82'), el Tag UBL no existe<br>Si el Tag UBL existe, el formato del Tag UBL es diferente de decimal positivo de 12 enteros y hasta 10 decimales | ERROR | 2580 (ERROR) El XML No contiene el tag o no existe informacion de la cantidad del item.<br>2780 (ERROR) El valor ingresado en cantidad de items no cumple con el estandar |
| 22 | Unidad de medida | Ítem | C | an..3 | (Catálogo N.° 03) <br> (Catálogo N.° 65) \| "UN/ECE rec 20" \| '"United Nations Economic Commission for Europe" | /DespatchAdvice/cac:DespatchLine/cbc:DeliveredQuantity@unitCode<br>@unitCodeListID<br>@unitCodeListAgencyName | (Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' y no existe el 'Indicador de traslado total de bienes') <br> o <br> (no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' o '09' o '82'), el atributo no existe o está vacío<br>Si existe el atributo, el 'Código del tipo de documento relacionado' es diferente a '50' y '52', el valor del atributo no se encuentra en el listado Catálogo 03<br>Si existe el atributo, el 'Código del tipo de documento relacionado' es igual a '50' o '52', el valor del atributo no se encuentra en el listado Catálogo 65<br>Si existe el atributo, el valor ingresado es diferente a 'UN/ECE rec 20'<br>Si existe el atributo, el valor ingresado es diferente a 'United Nations Economic Commission for Europe' | ERROR / OBSERV | 2883 (ERROR) Es obligatorio indicar la unidad de medida del item<br>4320 (OBSERV) El dato ingresado como unidad de medida no corresponde al valor esperado<br>4320 (OBSERV) El dato ingresado como unidad de medida no corresponde al valor esperado<br>4258 (OBSERV) El dato ingresado como atributo @unitCodeListID es incorrecto.<br>4259 (OBSERV) El dato ingresado como atributo @unitCodeListAgencyName es incorrecto. |
| 23 | Descripción detallada | Item | C | an..500 |  | /DespatchAdvice/cac:DespatchLine/cac:Item/cbc:Description | (Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' y no existe el 'Indicador de traslado total de bienes') <br> o <br> (no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' o '09' o '82'), el Tag UBL no existe o está vacío<br>Si existe el Tag UBL, el formato del Tag UBL es diferente a alfanumérico de 3 a 500 caracteres (se considera cualquier carácter, permite "whitespace character": espacio, salto de línea, fin de línea, tab, etc. Tampoco se permite solo espacios en blanco) | ERROR / OBSERV | 2781 (ERROR) El XML no contiene el atributo o no existe informacion de descripcion del items<br>4084 (OBSERV) Descripción del Ítem - El dato ingresado no cumple con el formato establecido. |
| 24 | Código del bien | Ítem | C | an..30 |  | /DespatchAdvice/cac:DespatchLine/cac:Item/cac:SellersItemIdentification/cbc:ID | Si el Tag UBL existe, el formato del Tag UBL es diferente a alfanumérico de hasta 30 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro 'whitespace character': salto de línea, tab, fin de línea, etc.) | OBSERV | 4085 (OBSERV) Código del Ítem - El dato ingresado no cumple con el formato establecido. |
| 25 | Código producto SUNAT | Ítem | C | n..8 | (Catálogo N.° 25) | /DespatchAdvice/cac:DespatchLine/cac:Item/cac:CommodityClassification/ <br> cbc:ItemClassificationCode | (Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' y no existe el 'Indicador de traslado total de bienes' y existe el 'Indicador de bien normalizado' con valor '1') <br> o <br> (no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' o '09' o '82' y existe el 'Indicador de bien normalizado' con valor '1'), no existe el Tag UBL o es vacío<br>Si existe el Tag UBL y es diferente de vacío, el valor del Tag UBL es diferente a numérico de hasta 8 caracteres (no acepta solamente ceros)<br>Si existe el Tag UBL y es diferente de vacío, el valor del Tag UBL no está en el listado<br>(Si existe el Tag UBL y es diferente de vacío, y existe el 'Indicador de bien normalizado' con valor '1'), el valor del Tag UBL no está en el listado de bienes normalizados | ERROR | 3372 (ERROR) Si el bien es regulado por SUNAT (existe Indicador de bien normalizado), debe indicar el Codigo de producto SUNAT<br>3002 (ERROR) El Código producto de SUNAT no es válido<br>3373 (ERROR) El Codigo de Producto SUNAT no se encuentra en el listado<br>3425 (ERROR) El Codigo de producto SUNAT no esta en el listado de bienes normalizados |
| 26 | Código GTIN | Ítem | C | an..14 |  | /DespatchAdvice/cac:DespatchLine/cac:Item/cac:StandardItemIdentification/cbc:ID (Código de producto GTIN)<br>@schemeID (Tipo de estructura GTIN) | Si existe el Tag UBL y es diferente de vacío, el valor del Tag UBL es diferente a alfanumérico de hasta 14 caracteres<br>Si existe el atributo, el valor ingresado es diferente a 'GTIN-8', 'GTIN-12', 'GTIN-13' y 'GTIN-14' | ERROR / OBSERV | 3375 (ERROR) El Codigo de Producto GTIN no cumple con el formato establecido<br>4335 (OBSERV) El tipo de estructura GS1 no tiene un valor permitido |
| 27 <br> <br> 28 | Partida arancelaria <br> <br> Indicador de bien normalizado | Ítem | C | an..100 \| an4 \| an10 \| an1 | \| (Catálogo N.° 55) \| "Propiedad del item" \| "PE:SUNAT" \| "urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo55" \| n10 \| "0" False <br> "1" True | /DespatchAdvice/cac:DespatchLine/cac:Item/cac:AdditionalItemProperty/cbc:Name (Nombre del concepto)<br>/DespatchAdvice/cac:DespatchLine/cac:Item/cac:AdditionalItemProperty/cbc:NameCode (Código del concepto)<br>@listName<br>@listAgencyName<br>@listURI<br>/DespatchAdvice/cac:DespatchLine/cac:Item/cac:AdditionalItemProperty/cbc:Value (Partida arancelaria)<br>/DespatchAdvice/cac:DespatchLine/cac:Item/cac:AdditionalItemProperty/cbc:Value (Indicador de bien normalizado) | Si existe el Tag UBL y es vacío<br>(Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' y no existe el 'Indicador de traslado total de bienes' y existe el 'Indicador de bien normalizado' con valor '1') <br> o <br> (no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' o '09' o '82' y existe el 'Indicador de bien normalizado' con valor '1'), no existe el Tag con código '7020' (Partida arancelaria)<br>(Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' y no existe el 'Indicador de traslado total de bienes') <br> o <br> (no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50' o '52' o '09' o '82'), y existe 'Documento relacionado' con 'Código de tipo de documento' igual a '80', no existe al menos un bien (cac:DespatchAdviceLine) que contenga el código '7020' (Partida arancelaria)<br>Si existe el atributo, el valor ingresado es diferente a 'Propiedad del item'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT'<br>Si existe el atributo, el valor ingresado es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo55'<br>De existir 'Código del concepto' igual a '7020' o '7022', no existe el tag o es vacío.<br>Si el 'Código del concepto' es '7020', el valor del Tag UBL es diferente a numérico de hasta 10 caracteres (no acepta solamente ceros)<br>Si el 'Código de concepto' es '7020', el valor del Tag UBL no se encuentra en el listado<br>Si el 'Código del concepto' es '7020', y el 'Indicador de bien normalizado' es '1', el valor del Tag UBL no está en el listado de bienes normalizados<br>Si el 'Código de concepto' es '7022', el valor del Tag UBL es diferente de '0' y diferente de '1'<br>Si el 'Código de concepto' es '7022' y el valor del Tag UBL es '1', y no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01', '03', '04', '12', '48', '50', '52', '09' u '82', no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '80' | OBSERV / ERROR | 4235 (OBSERV) No existe información en el nombre del concepto.<br>3426 (ERROR) Si se trata de un bien normalizado por SUNAT, debe indicarse la Partida arancelaria<br>3352 (ERROR) Si ingreso un documento relacionado tipo "49" (solo en caso de GRE-Remitente) u "80", debe existir al menos un item con Partida arancelaria<br>4252 (OBSERV) El dato ingresado como atributo @listName es incorrecto.<br>4251 (OBSERV) El dato ingresado como atributo @listAgencyName es incorrecto.<br>4253 (OBSERV) El dato ingresado como atributo @listURI es incorrecto.<br>3064 (ERROR) El XML no contiene tag o no existe informacion del valor del concepto por linea.<br>3377 (ERROR) La Partida arancelaria no cumple con el formato establecido<br>3378 (ERROR) La Partida arancelaria no se encuentra en el listado<br>3429 (ERROR) La Partida arancelaria no esta en el listado de bienes normalizados<br>3396 (ERROR) El Indicador de bien normalizado es invalido<br>3379 (ERROR) Debe consignar un documento relacionado igual a '49' (solo en caso de GRE-Remitente) u '80' para un traslado que comprende un bien normalizado |

**Dirección del punto de partida**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 29 | Ubigeo de partida | Global | M | n6 | (Catálogo N° 13) \| "PE:INEI" \| "Ubigeos" | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cbc:ID<br>@schemeAgencyName<br>@schemeName | No existe el Tag UBL o es vacío<br>El formato del Tag UBL es diferente a numérico de 6 dígitos<br>El valor del Tag UBL no está en el listado<br>Si existe el atributo, el valor ingresado es diferente a 'PE:INEI'<br>Si existe el atributo, el valor ingresado es diferente a 'Ubigeos' | ERROR / OBSERV | 2775 (ERROR) El XML no contiene el atributo o no existe informacion del codigo de ubigeo.<br>2776 (ERROR) El valor ingresado como codigo de ubigeo no cumple con el estandar.<br>3363 (ERROR) El codigo de ubigeo del punto de partida no existe<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto.<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto. |
| 30 | Direccion completa y detallada de partida | Global | C | an..500 |  | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cac:AddressLine/cbc:Line | Si no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09' con serie que no empieza con número, y no existe 'Indicador de transbordo programado', no existe el Tag UBL o es vacío<br>Si el Tag UBL existe, el formato del Tag UBL es diferente a alfanumérico de 3 a 500 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro "whitespace character": salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | ERROR / OBSERV | 2577 (ERROR) El XML no contiene el tag o no existe informacion de direccion detallada de punto de partida.<br>4076 (OBSERV) Direccion de punto de partida - El dato ingresado no cumple con el formato establecido. |
| 31 | Punto de georreferencia de partida | Global | C | an..13 \| an2 \| an..13 \| an2 | n(3,8) \| 'DD' \| n(3,8) \| 'DD' | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cac:LocationCoordinate/cbc:LatitudeDegreesMeasure (Longitud) <br> <br><br>@unitCode<br>/DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cac:LocationCoordinate/cbc:LongitudeDegreesMeasure (Latitud)<br>@unitCode | El formato del Tag UBL es diferente de númerico de hasta 3 enteros y hasta 8 decimales (acepta valores positivos o negativos)<br><<< SIN VALIDACIÓN >>><br>El formato del Tag UBL es diferente de númerico de hasta 3 enteros y hasta 8 decimales (acepta valores positivos o negativos)<br><<< SIN VALIDACIÓN >>> | ERROR / - | 3413 (ERROR) El valor ingresado como Punto de georreferencia no cumple con el formato establecido<br>(sin validacion) <<< SIN VALIDACIÓN >>><br>3413 (ERROR) El valor ingresado como Punto de georreferencia no cumple con el formato establecido<br>(sin validacion) <<< SIN VALIDACIÓN >>> |

**Dirección del punto de llegada**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 32 | Ubigeo de llegada | Global | C | n6 | (Catálogo N° 13) \| "PE:INEI" \| "Ubigeos" | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cbc:ID<br>@schemeAgencyName<br>@schemeName | Si no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09' con serie que no empieza con número y con 'Motivo de traslado' igual a '18' , y no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09' con serie que empieza con número, no existe el Tag UBL o es vacío<br>Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09' con serie que empieza con número, no existe el Tag UBL o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a numérico de 6 dígitos<br>Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no está en el listado<br>Si existe el atributo, el valor ingresado es diferente a 'PE:INEI'<br>Si existe el atributo, el valor ingresado es diferente a 'Ubigeos' | ERROR / OBSERV | 2775 (ERROR) El XML no contiene el atributo o no existe informacion del codigo de ubigeo.<br>4431 (OBSERV) Si en caso el motivo de traslado de la guia remision remitente es Traslado emisor itinerante CP no corresponde informar codigo de ubigeo del punto de llegada<br>2776 (ERROR) El valor ingresado como codigo de ubigeo no cumple con el estandar.<br>3368 (ERROR) El codigo de ubigeo del punto de llegada no existe<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto.<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto. |
| 33 | Direccion completa y detallada de llegada | Global | C | an..500 |  | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cac:AddressLine/cbc:Line | Si no existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09', y no existe el 'Indicador de transbordo programado', no existe el Tag UBL o es vacío<br>Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09' con serie que empieza con número, y no existe el 'Indicador de transbordo programado', no existe el Tag UBL o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 3 a 500 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro "whitespace character": salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | ERROR / OBSERV | 2574 (ERROR) El XML no contiene el tag o no existe informacion de direccion detallada de punto de llegada.<br>4178 (OBSERV) cac:DeliveryAddress: Debe consignar Dirección del punto de llegada (cbc:StreetName).<br>4068 (OBSERV) Direccion de punto de llegada - El dato ingresado no cumple con el formato establecido. |
| 34 | Punto de georreferencia de llegada | Global | C | an..13 \| an2 \| an..13 \| an2 | n(3,8) \| 'DD' \| n(3,8) \| 'DD' | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cac:LocationCoordinate/cbc:LatitudeDegreesMeasure (Longitud)<br>@unitCode<br>/DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cac:LocationCoordinate/cbc:LongitudeDegreesMeasure (Latitud)<br>@unitCode | El formato del Tag UBL es diferente de númerico de hasta 3 enteros y hasta 8 decimales (acepta valores positivos o negativos)<br><<< SIN VALIDACIÓN >>><br>El formato del Tag UBL es diferente de númerico de hasta 3 enteros y hasta 8 decimales (acepta valores positivos o negativos)<br><<< SIN VALIDACIÓN >>> | ERROR / - | 3413 (ERROR) El valor ingresado como Punto de georreferencia no cumple con el formato establecido<br>(sin validacion) <<< SIN VALIDACIÓN >>><br>3413 (ERROR) El valor ingresado como Punto de georreferencia no cumple con el formato establecido<br>(sin validacion) <<< SIN VALIDACIÓN >>> |

**Vehículo principal**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 35 | Número de placa del vehiculo | Global | M | an..8 |  | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cbc:ID | El Tag UBL no existe o está vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 6 a 8 caracteres (solo se permiten letras mayúsculas y números, no espacios en blanco ni guion, tampoco se permite solamente ceros)<br>Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no se encuentra en el listado | ERROR / OBSERV | 2566 (ERROR) El XML no contiene el tag o no existe informacion del Numero de placa del vehículo.<br>2567 (ERROR) Numero de placa del vehículo - El dato ingresado no cumple con el formato establecido.<br>4398 (OBSERV) El Numero de placa no se encuentra en las bases consultadas |
| 36 | Tarjeta Única de Circulación Electrónica o Certificado de Habilitación vehicular | Global | C | an..15 |  | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:ApplicableTransportMeans/cbc:RegistrationNationalityID (Número de documento) <br> | El Tag UBL no existe o está vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 10 a 15 caracteres (solo se permite letras mayúsculas y números, no se permite ningún otro "whitespace character": espacio, salto de línea, tab, fin de línea, etc., tampoco se permite solamente ceros)<br>Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no se encuentra en el listado <br> | OBSERV / ERROR | 4399 (OBSERV) No ha consignado el Numero de Constancia de Inscripcion Vehicular o Certificado de Habilitacion Vehicular o la TUC (fisica o electronica)<br>3355 (ERROR) El Numero de Constancia de Inscripcion Vehicular o Certificado de Habilitacion Vehicular o la TUC (fisica o electronica) no cumple con el formato establecido<br>4400 (OBSERV) El Numero de Constancia de Inscripcion Vehicular o Certificado de Habilitacion Vehicular o la TUC (fisica o electronica) no se encuentra en las bases consultadas |
| 37 | Entidad emisora de la autorización - vehículo principal <br> <br> Número de autorización del vehículo emitido por la entidad - vehículo principal | Global | C | an..50 \| an2 | \| (Catálogo N° D-37) \| "Entidad Autorizadora" \| "PE:SUNAT" | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:ShipmentDocumentReference/cbc:ID (Número de autorización)<br>/DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:ShipmentDocumentReference/cbc:ID@schemeID (Entidad que emite el documento)<br>@schemeName<br>@schemeAgencyName | Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 3 a 50 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro 'whitespace character': salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco)<br>Si el Tag UBL existe y es diferente de vacío, existe otro cac:ShipmentDocumentReference<br>Si existe 'Número de autorización' con valor diferente de vacío, no existe el atributo o es vacío<br>Si existe el atributo y es diferente de vacío, el valor ingresado no se encuentra en el listado<br>Si existe el atributo y es diferente de vacío, el 'Número de autorización' es vacío<br>Si existe el atributo, el valor ingresado es diferente a 'Entidad Autorizadora'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT' | OBSERV / ERROR | 4406 (OBSERV) El Numero de autorizacion del vehiculo no cumple con el formato establecido<br>3356 (ERROR) Solo debe consignar un Numero de autorizacion del vehiculo<br>4403 (OBSERV) Debe indicar la entidad autorizadora del vehiculo<br>4407 (OBSERV) El Codigo de entidad autorizadora del vehiculo no corresponde a un valor esperado<br>4405 (OBSERV) Si existe el Codigo de entidad autorizadora, debe consignar el Numero de autorizacion especial del vehiculo<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto.<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto. |

**Vehiculos Secundarios - Incremental hasta 2**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 38 | Número de placa del vehículo | Global | C | an..8 |  | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:AttachedTransportEquipment/cbc:ID | Existe más de dos vehículos secundarios (cac:AttachedTransportEquipment)<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 6 a 8 caracteres (solo se permiten letras mayúsculas y números, no espacios en blanco ni guion, tampoco se permite solamente ceros)<br>Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no se encuentra en el listado | OBSERV / ERROR | 4389 (OBSERV) Solo corresponde consignar hasta dos vehiculos secundarios<br>2567 (ERROR) Numero de placa del vehículo - El dato ingresado no cumple con el formato establecido.<br>4398 (OBSERV) El Numero de placa no se encuentra en las bases consultadas |
| 39 | Tarjeta Única de Circulación Electrónica o Certificado de Habilitación vehicular | Global | C | an..15 |  | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:AttachedTransportEquipment/cac:ApplicableTransportMeans/cbc:RegistrationNationalityID (Número de documento) <br> | Si existe 'Número de placa del vehículo secundario' diferente de vacío, el Tag UBL no existe o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 10 a 15 caracteres (solo se permite letras mayúsculas y números, no se permite ningún otro "whitespace character": espacio, salto de línea, tab, fin de línea, etc., tampoco se permite solamente ceros)<br>Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no se encuentra en el listado <br> | OBSERV / ERROR | 4399 (OBSERV) No ha consignado el Numero de Constancia de Inscripcion Vehicular o Certificado de Habilitacion Vehicular o la TUC (fisica o electronica)<br>3355 (ERROR) El Numero de Constancia de Inscripcion Vehicular o Certificado de Habilitacion Vehicular o la TUC (fisica o electronica) no cumple con el formato establecido<br>4400 (OBSERV) El Numero de Constancia de Inscripcion Vehicular o Certificado de Habilitacion Vehicular o la TUC (fisica o electronica) no se encuentra en las bases consultadas |
| 40 | Entidad emisora de la autorización - vehículo secundario <br> <br> Número de autorización del vehículo emitido por la entidad - vehículo secundario | Global | C | an..50 \| an2 | \| (Catálogo N° D-37) \| "Entidad Autorizadora" \| "PE:SUNAT" | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:AttachedTransportEquipment/cac:ShipmentDocumentReference/cbc:ID (Número de autorización) <br><br>/DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:AttachedTransportEquipment/cac:ShipmentDocumentReference/cbc:ID@schemeID (Entidad que emite el documento)<br>@schemeName<br>@schemeAgencyName | Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 3 hasta 50 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro 'whitespace character': salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco)<br>Si el Tag UBL existe y es diferente de vacío, existe otro cac:ShipmentDocumentReference<br>Si existe 'Número de autorización' con valor diferente de vacío, no existe el atributo o es vacío<br>Si existe el atributo y es diferente de vacío, el valor ingresado no se encuentra en el listado<br>Si existe el atributo y es diferente de vacío, el 'Número de autorización' es vacío<br>Si existe el atributo, el valor ingresado es diferente a 'Entidad Autorizadora'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT' | OBSERV / ERROR | 4406 (OBSERV) El Numero de autorizacion del vehiculo no cumple con el formato establecido<br>3356 (ERROR) Solo debe consignar un Numero de autorizacion del vehiculo<br>4403 (OBSERV) Debe indicar la entidad autorizadora del vehiculo<br>4407 (OBSERV) El Codigo de entidad autorizadora del vehiculo no corresponde a un valor esperado<br>4405 (OBSERV) Si existe el Codigo de entidad autorizadora, debe consignar el Numero de autorizacion especial del vehiculo<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto.<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto. |

**Conductor principal**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 41 | Tipo de Conductor | Global | M | an9 | 'Principal' | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:JobTitle | No existe un tag cac:DriverPerson con 'Tipo de Conductor' igual a 'Principal'<br>Existe más de un tag cac:DriverPerson con 'Tipo de Conductor' igual a 'Principal' | ERROR | 3357 (ERROR) Debe consignar informacion del conductor principal<br>3358 (ERROR) Solo debe existir informacion de un conductor principal |
| 42 | Tipo y número de documento de identidad del conductor | Global | M | an1 \| an..15 | (Catálogo N° 06) \| "Documento de Identidad" \| "PE:SUNAT" \| "urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06" | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:ID@schemeID (Tipo de documento de identidad)<br>@schemeName<br>@schemeAgencyName<br>@schemeURI<br>/DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:ID (Número de documento de identidad) | Si el 'Tipo de Conductor' es igual a 'Principal', el atributo no existe o es vacío<br>Si el 'Tipo de Conductor' es igual a 'Principal', el valor del atributo es distinto al Catálogo N° 06 o es igual a '6'<br>Si existe el atributo, el valor ingresado es diferente a 'Documento de Identidad'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT'<br>Si existe el atributo, el valor ingresado es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06'<br>Si el 'Tipo de Conductor' es igual a 'Principal', el Tag UBL no existe o es vacío<br>Si el 'Tipo de documento de identidad del conductor' es "1", el fomato del Tag UBL es diferente a numérico de 8 dígitos<br>Si el 'Tipo de documento de identidad del conductor' es "1", el valor del Tag UBL no se encuentra en el listado <br><br>Si el 'Tipo de documento de identidad del conductor' es diferente a '1', el formato del Tag UBL es diferente a alfanumérico de hasta 15 caracteres (se considera cualquier carácter, no se permite ningún otro 'whitespace character': espacio, salto de línea, tab, fin de línea, etc.) | ERROR / OBSERV | 2570 (ERROR) El XML no contiene el tag o no existe informacion del tipo de documento identidad del conductor.<br>2571 (ERROR) cac:DriverPerson/ID@schemeID - El valor ingresado de tipo de documento identidad de conductor es incorrecto.<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto.<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto.<br>4257 (OBSERV) El dato ingresado como atributo @schemeURI es incorrecto.<br>2568 (ERROR) El XML no contiene el tag o no existe informacion en el Numero de documento de identidad del conductor.<br>2569 (ERROR) Documento identidad del conductor - El dato ingresado no cumple con el formato establecido.<br>3359 (ERROR) El numero de DNI del conductor no existe<br>2569 (ERROR) Documento identidad del conductor - El dato ingresado no cumple con el formato establecido. |
| 43 | Apellidos y nombres del conductor | Global | M | an..250 |  | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:FirstName (Nombres)<br>/DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:FamilyName (Apellidos) | Si el 'Tipo de Conductor' es igual a 'Principal', el Tag UBL no existe o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro 'whitespace character': salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco)<br>Si el 'Tipo de Conductor' es igual a 'Principal', el Tag UBL no existe o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro 'whitespace character': salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | ERROR / OBSERV | 3360 (ERROR) Debe consignar informacion de los nombres del conductor<br>4409 (OBSERV) Los nombres y/o apellidos del conductor no cumplen con el formato establecido<br>3361 (ERROR) Debe consignar informacion de los apellidos del conductor<br>4409 (OBSERV) Los nombres y/o apellidos del conductor no cumplen con el formato establecido |
| 44 | Número de licencia de conducir | Global | M | an..10 |  | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cac:IdentityDocumentReference/cbc:ID | Si el 'Tipo de Conductor' es igual a 'Principal', el Tag UBL no existe o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 9 a 10 caracteres (solo se permiten letras mayúsculas y números, no se permite solamente ceros)<br>Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no se encuentra en el listado <br> | ERROR / OBSERV | 2572 (ERROR) El XML no contiene el tag o no existe informacion del Numero de licencia del conductor.<br>2573 (ERROR) Numero de licencia del conductor - El dato ingresado no cumple con el formato establecido.<br>4412 (OBSERV) El Numero de licencia de conducir no se encuentra en las bases consultadas |

**Conductores Secundarios - Incremental hasta 2**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 45 | Tipo de Conductor | Global | C | an10 | 'Secundario' | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:JobTitle | Existe más de dos tag cac:DriverPerson con 'Tipo de Conductor' igual a 'Secundario'<br>Existe más de un tag cac:DriverPerson con 'Tipo de Conductor' igual a 'Secundario' y 'Número de licencia de conducir' secundario iguales (no se considera los 'Números de licencia de conducir' con valor vacío) | OBSERV / ERROR | 4376 (OBSERV) Solo debe consignar informacion de hasta dos conductores secundarios<br>3362 (ERROR) No debe repetirse la informacion de conductores |
| 46 | Tipo y número de documento de identidad del conductor | Global | C | an1 \| an..15 | (Catálogo N° 06) \| "Documento de Identidad" \| "PE:SUNAT" \| "urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06" | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:ID@schemeID<br>@schemeName<br>@schemeAgencyName<br>@schemeURI<br>/DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:ID | Si el 'Tipo de Conductor' es igual a 'Secundario' y existe 'Número de documento de identidad' de conductor secundario con valor diferente de vacío, no existe el atributo o es vacío<br>Si existe el atributo y es diferente de vacío, y el 'Tipo de Conductor' es igual a 'Secundario', el valor del atributo es distinto al Catálogo N° 06 o es igual a '6'<br>Si existe el atributo, el valor ingresado es diferente a 'Documento de Identidad'<br>Si existe el atributo, el valor ingresado es diferente a 'PE:SUNAT'<br>Si existe el atributo, el valor ingresado es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06'<br>Si el 'Tipo de Conductor' es igual a 'Secundario' y existe 'Tipo de documento de identidad' de conductor secundario con valor diferente de vacío, el valor del Tag UBL es vacío<br>Si el 'Tipo de Conductor' es igual a 'Secundario' y existe 'Número de licencia de conducir' de conductor secundario con valor diferente de vacío, no existe el Tag UBL o es vacío<br>Si el 'Tipo de documento de identidad del conductor' es "1", el fomato del Tag UBL es diferente a numérico de 8 dígitos<br>Si el 'Tipo de documento de identidad del conductor' es "1", el valor del Tag UBL no se encuentra en el listado <br><br>Si el 'Tipo de documento de identidad del conductor' es diferente a '1' y diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 15 caracteres (se considera cualquier carácter, no se permite ningún otro 'whitespace character': espacio, salto de línea, tab, fin de línea, etc.) | ERROR / OBSERV | 2570 (ERROR) El XML no contiene el tag o no existe informacion del tipo de documento identidad del conductor.<br>2571 (ERROR) cac:DriverPerson/ID@schemeID - El valor ingresado de tipo de documento identidad de conductor es incorrecto.<br>4255 (OBSERV) El dato ingresado como atributo @schemeName es incorrecto.<br>4256 (OBSERV) El dato ingresado como atributo @schemeAgencyName es incorrecto.<br>4257 (OBSERV) El dato ingresado como atributo @schemeURI es incorrecto.<br>2568 (ERROR) El XML no contiene el tag o no existe informacion en el Numero de documento de identidad del conductor.<br>2568 (ERROR) El XML no contiene el tag o no existe informacion en el Numero de documento de identidad del conductor.<br>2569 (ERROR) Documento identidad del conductor - El dato ingresado no cumple con el formato establecido.<br>3359 (ERROR) El numero de DNI del conductor no existe<br>2569 (ERROR) Documento identidad del conductor - El dato ingresado no cumple con el formato establecido. |
| 47 | Apellidos y nombres del conductor | Global | C | an..250 \| an..250 |  | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:FirstName (Nombres)<br>/DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:FamilyName (Apellidos) | Si el 'Tipo de Conductor' es igual a 'Secundario' y existe 'Número de licencia de conducir' de conductor secundario con valor diferente de vacío, no existe el Tag UBL o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro 'whitespace character': salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco)<br>Si el 'Tipo de Conductor' es igual a 'Secundario' y existe 'Número de licencia de conducir' de conductor secundario con valor diferente de vacío, no existe el Tag UBL o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro 'whitespace character': salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | ERROR / OBSERV | 3360 (ERROR) Debe consignar informacion de los nombres del conductor<br>4409 (OBSERV) Los nombres y/o apellidos del conductor no cumplen con el formato establecido<br>3361 (ERROR) Debe consignar informacion de los apellidos del conductor<br>4409 (OBSERV) Los nombres y/o apellidos del conductor no cumplen con el formato establecido |
| 48 | Número de licencia de conducir | Global | C | an..10 |  | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cac:IdentityDocumentReference/cbc:ID | Si el 'Tipo de Conductor' es igual a 'Secundario' y existe 'Número de documento de identidad' de conductor secundario con valor diferente de vacío, no existe el Tag UBL o es vacío<br>Si el Tag UBL existe y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 9 a 10 caracteres (no se permite solamente ceros)<br>Si el Tag UBL existe y es diferente de vacío, el valor del Tag UBL no se encuentra en el listado <br> | ERROR / OBSERV | 2572 (ERROR) El XML no contiene el tag o no existe informacion del Numero de licencia del conductor.<br>2573 (ERROR) Numero de licencia del conductor - El dato ingresado no cumple con el formato establecido.<br>4412 (OBSERV) El Numero de licencia de conducir no se encuentra en las bases consultadas |

**Datos del traslado**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 49 | Identificador del traslado | Global | M | an11 | "SUNAT_Envio" | /DespatchAdvice/cac:Shipment/cbc:ID | <<< SIN VALIDACIÓN >>> | - | (sin validacion) <<< SIN VALIDACIÓN >>> |
| 50 | Fecha Inicio de traslado | Global | M | an..10 | YYYY-MM-DD | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:TransitPeriod/cbc:StartDate | No existe el Tag UBL<br>Si existe y el valor del Tag UBL es menor a la 'Fecha de emisión' de la guía (cbc:IssueDate)<br>Si existe y el Tag UBL no cumple con el formato | ERROR | 3406 (ERROR) El XML no contiene el tag o no existe informacion de la fecha de inicio de traslado o fecha de entrega del bien al Transportista<br>3343 (ERROR) La Fecha de inicio del traslado debe ser mayor o igual a la Fecha de emision del documento<br>3407 (ERROR) El valor ingresado como fecha de inicio o fecha de entrega al Transportista no cumple con el estandar (YYYY-MM-DD) |
| 51 | Anotación opcional sobre los bienes a transportar <br> | Global | C | n1 \| an..500 | "0" | /DespatchAdvice/cac:DespatchLine/cbc:ID<br>/DespatchAdvice/cac:DespatchLine/cac:OrderLineReference/cbc:LineID<br>/DespatchAdvice/cac:DespatchLine/cac:Item/cbc:Description | <<< SIN VALIDACIÓN >>><br><<< SIN VALIDACIÓN >>><br>(Si existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '09' con serie que empieza con número) <br> o <br> (existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '82'), no existe un cac:DespatchLine con cbc:ID con valor cero y que contenga el campo cac:Item/cbc:Description diferente de vacío<br>(Si existe 'Indicador de traslado total de bienes' y existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '01' o '04', con serie que empieza con número) <br> o <br> (existe 'Indicador de traslado total de bienes' y existe 'Documento relacionado' con 'Código de tipo de documento relacionado' igual a '03', '12' o '48'), no existe un cac:DespatchLine con cbc:ID con valor cero y que contenga el campo cac:Item/cbc:Description diferente de vacío<br>Si existe el Tag UBL y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de 3 a 500 caracteres (se considera cualquier carácter, permite "whitespace character": espacio, salto de línea, fin de línea, tab, etc. Tampoco se permite solo espacios en blanco) | - / OBSERV | (sin validacion) <<< SIN VALIDACIÓN >>><br>(sin validacion) <<< SIN VALIDACIÓN >>><br>4429 (OBSERV) La anotacion opcional sobre los bienes a transportar no existe o no contiene informacion<br>4429 (OBSERV) La anotacion opcional sobre los bienes a transportar no existe o no contiene informacion<br>4430 (OBSERV) La anotacion opcional sobre los bienes a transportar no cumple con el formato establecido |
| 52 | Peso bruto total de la carga <br> <br> Unidad de medida del peso bruto | Global | M | an..16 \| an3 | n(12,3) \| "KGM" Kilogramos <br> "TNE" Toneladas | /DespatchAdvice/cac:Shipment/cbc:GrossWeightMeasure<br>/DespatchAdvice/cac:Shipment/cbc:GrossWeightMeasure@unitCode | No existe el Tag UBL<br>El formato del Tag UBL es diferente a decimal positivo de 12 enteros y 3 decimales<br>El valor del atributo es vacío<br>El valor del atributo es diferente 'KGM' y diferente de 'TNE' | ERROR | 2880 (ERROR) Es obligatorio ingresar el peso bruto total de la guía<br>2523 (ERROR) GrossWeightMeasure - El dato ingresado no cumple con el formato establecido.<br>2881 (ERROR) Es obligatorio indicar la unidad de medida del Peso Total de la guía<br>2523 (ERROR) GrossWeightMeasure - El dato ingresado no cumple con el formato establecido. |
| 53 | Indicador de traslado total de bienes | Global | C | an…50 | "SUNAT_Envio_IndicadorTrasladoTotal" | /DespatchAdvice/cac:Shipment/cbc:SpecialInstructions | Existe más de un Tag UBL con valor "SUNAT_Envio_IndicadorTrasladoTotal" | ERROR | 3344 (ERROR) El indicador no debe repetirse |
| 54 | Indicador de retorno de vehículo con envases o embalajes vacíos | Global | C | an..50 | "SUNAT_Envio_IndicadorRetornoVehiculoEnvaseVacio" | /DespatchAdvice/cac:Shipment/cbc:SpecialInstructions | Si existe el Tag UBL y las 6 primeras posiciones es "SUNAT_", el valor del Tag UBL es diferente de alguno de los siguientes valores: <br> SUNAT_Envio_IndicadorTransbordoProgramado <br> SUNAT_Envio_IndicadorRetornoVehiculoEnvaseVacio <br> SUNAT_Envio_IndicadorRetornoVehiculoVacio <br> SUNAT_Envio_IndicadorTrasporteSubcontratado <br> SUNAT_Envio_IndicadorPagadorFlete_Remitente <br> SUNAT_Envio_IndicadorPagadorFlete_Subcontratador <br> SUNAT_Envio_IndicadorPagadorFlete_Tercero <br> SUNAT_Envio_IndicadorTrasladoTotal<br>Existe más de un Tag UBL con valor "SUNAT_Envio_IndicadorRetornoVehiculoEnvaseVacio" | ERROR | 3388 (ERROR) El indicador no cumple con el formato establecido<br>3344 (ERROR) El indicador no debe repetirse |
| 55 | Indicador de retorno de vehículo vacío | Global | C | an…50 | "SUNAT_Envio_IndicadorRetornoVehiculoVacio" | /DespatchAdvice/cac:Shipment/cbc:SpecialInstructions | Existe más de un Tag UBL con valor "SUNAT_Envio_IndicadorRetornoVehiculoVacio" | ERROR | 3344 (ERROR) El indicador no debe repetirse |
| 56 | Indicador de Transbordo Programado | Global | C | an..50 | "SUNAT_Envio_IndicadorTransbordoProgramado" | /DespatchAdvice/cac:Shipment/cbc:SpecialInstructions | Existe más de un Tag UBL con valor "SUNAT_Envio_IndicadorTransbordoProgramado" | ERROR | 3344 (ERROR) El indicador no debe repetirse |
| 57 | Indicador de transporte subcontratado | Global | C | an..50 | "SUNAT_Envio_IndicadorTrasporteSubcontratado" | /DespatchAdvice/cac:Shipment/cbc:SpecialInstructions | Existe más de un Tag UBL con valor "SUNAT_Envio_IndicadorTrasporteSubcontratado" | ERROR | 3344 (ERROR) El indicador no debe repetirse |
| 58 | Indicador de pagador de flete | Global | C | an..50 | "SUNAT_Envio_IndicadorPagadorFlete_Remitente" <br> "SUNAT_Envio_IndicadorPagadorFlete_Subcontratador" <br> "SUNAT_Envio_IndicadorPagadorFlete_Tercero" | /DespatchAdvice/cac:Shipment/cbc:SpecialInstructions | No existe un Tag UBL con valor "SUNAT_Envio_IndicadorPagadorFlete_Remitente" o <br> "SUNAT_Envio_IndicadorPagadorFlete_Subcontratador" o <br> "SUNAT_Envio_IndicadorPagadorFlete_Tercero"<br>Existe más de un Tag UBL con las primeras 34 posiciones con valor "SUNAT_Envio_IndicadorPagadorFlete_" | OBSERV / ERROR | 4388 (OBSERV) Debe consignar el Indicador de pagador de flete<br>3344 (ERROR) El indicador no debe repetirse |
| 59 | Tipo de evento | Global | C | an1 | 1-Transbordo no programado <br> 2-Imposibilidad arribo <br> 3-Imposibilidad entrega | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:TransportEvent/cbc:TransportEventTypeCode | Si existe el Tag UBL y es diferente de vacío <br> Nota: Este Tag es de uso exclusivo cuando se trata de una 'Guía por eventos', y no debe ser enviado | ERROR | 3374 (ERROR) No debe consignar el el Tipo de evento |

**Datos de empresa que subcontrata**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 60 | Tipo y número de documento de identidad del subcontratador | Global | C | an11 \| an1 \| an11 | "SUNAT_Envio" \| "6" \| n11 | /DespatchAdvice/cac:Shipment/cac:Consignment/cbc:ID (Identificador obligatorio)<br>/DespatchAdvice/cac:Shipment/cac:Consignment/cac:LogisticsOperatorParty/cac:PartyIdentification/cbc:ID@schemeID (Tipo de documento)<br>/DespatchAdvice/cac:Shipment/cac:Consignment/cac:LogisticsOperatorParty/cac:PartyIdentification/cbc:ID (Número de RUC) | <<< SIN VALIDACIÓN >>><br>Si existe el 'Indicador de transporte subcontratado', no existe el atributo o es vacío<br>Si existe el atributo y el valor del atributo es diferente a '6'<br>Si existe el 'Indicador de transporte subcontratado', no existe el Tag UBL o es vacío<br>Si existe el Tag UBL, el valor del Tag UBL es el mismo que el 'Número de RUC transportista'<br>Si existe el Tag UBL y es diferente de vacío, el valor del Tag UBL no se encuentra en el listado | - / OBSERV / ERROR | (sin validacion) <<< SIN VALIDACIÓN >>><br>4425 (OBSERV) El XML no contiene el atributo o no existe informacion del tipo de documento de identidad del subcontratador<br>3391 (ERROR) El Tipo de documento de identidad del subcontratador debe ser 6-Numero de RUC<br>4424 (OBSERV) El XML no contiene el tag o no existe informacion del numero de documento de identidad del subcontratador<br>3390 (ERROR) El tipo y numero de documento de identidad del subcontratador es el mismo que del Transportista<br>3389 (ERROR) El Numero de RUC del subcontratador no existe |
| 61 | Apellidos y nombres, denominacion o razón social del subcontratador | Global | C | an..250 |  | /DespatchAdvice/cac:Shipment/cac:Consignment/cac:LogisticsOperatorParty/cac:PartyLegalEntity/cbc:RegistrationName | Si existe el 'Indicador de transporte subcontratado', no existe el Tag UBL o es vacío<br>Si existe el Tag UBL y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro "whitespace character": salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | OBSERV | 4426 (OBSERV) El XML no contiene el atributo o no existe informacion del nombre o razon social del subcontratador<br>4427 (OBSERV) El nombre/razon social del subcontratador no cumple con el formato establecido |

**Datos quien paga el servicio de transporte - solo se completa cuando paga un tercero**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 62 | Tipo y número de documento de identidad de quien paga el servicio | Global | C | an1 \| an..15 | (Catálogo N° 06) | /DespatchAdvice/cac:OriginatorCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID@schemeID (Tipo de documento de identidad)<br>/DespatchAdvice/cac:OriginatorCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID (Número de documento de identidad) | Si existe el 'Indicador de pagador de flete' con valor 'SUNAT_Envio_IndicadorPagadorFlete_Tercero', no existe el atributo o es vacío<br>Si existe el 'Indicador de pagador de flete' con valor 'SUNAT_Envio_IndicadorPagadorFlete_Tercero', el valor del atributo no está en el listado<br>Si existe el 'Indicador de pagador de flete' con valor 'SUNAT_Envio_IndicadorPagadorFlete_Tercero', no existe el Tag UBL o es vacío<br>Si 'Tipo de documento de identidad de quien paga el servicio' es '1' y el fomato del Tag UBL es diferente a numérico de 8 dígitos<br>Si 'Tipo de documento de identidad de quien paga el servicio' es '1', el valor del Tag UBL no se encuentra en base RENIEC <br><br>Si 'Tipo de documento de identidad de quien paga el servicio' es '6', el formato del Tag UBL es diferente a numérico de 11 dígitos<br>Si 'Tipo de documento de identidad de quien paga el servicio' es '6', el valor del Tag UBL no se encuentra en el listado<br>Si existe 'Tipo de documento de identidad de quien paga el servicio' y es diferente a '1', '6' y vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 15 caracteres (se considera cualquier carácter, no permite 'whitespace character': espacio, salto de línea, fin de línea, tab, etc.) | OBSERV / ERROR | 4401 (OBSERV) El XML no contiene el tag o no existe informacion del tipo de documento de identidad de quien paga el servicio<br>3399 (ERROR) El Tipo de documento de identidad de quien paga el servicio es invalido<br>4402 (OBSERV) El XML no contiene el atributo o no existe informacion del numero de documento de identidad de quien paga el servicio<br>3400 (ERROR) El Numero de documento de identidad de quien paga el servicio no cumple con el formato establecido<br>3401 (ERROR) El Numero de DNI de quien paga el servicio no existe<br>3400 (ERROR) El Numero de documento de identidad de quien paga el servicio no cumple con el formato establecido<br>3402 (ERROR) El Numero de RUC de quien paga el servicio no existe<br>3400 (ERROR) El Numero de documento de identidad de quien paga el servicio no cumple con el formato establecido |
| 63 | Apellidos y nombres, denominacion o razón social de quien paga el servicio | Global | C | an..250 |  | /DespatchAdvice/cac:OriginatorCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName | Si existe el 'Indicador de pagador de flete' con valor 'SUNAT_Envio_IndicadorPagadorFlete_Tercero', no existe el Tag UBL o es vacío<br>Si existe el Tag UBL y es diferente de vacío, el formato del Tag UBL es diferente a alfanumérico de hasta 250 caracteres (se considera cualquier carácter incluido espacio, no se permite ningún otro 'whitespace character': salto de línea, tab, fin de línea, etc. Tampoco se permite solo espacios en blanco) | OBSERV | 4370 (OBSERV) El XML no contiene el tag o no existe informacion del nombre o razon social de quien paga el servicio<br>4423 (OBSERV) El nombre o razon social de quien paga el servicio no cumple con el formato establecido |

**Firma Digital**

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato / valor esperado | Tag UBL | Condiciones de validacion | Tipo de retorno | Codigos de retorno |
|---|---|---|---|---|---|---|---|---|---|
| 64 | Firma digital | Global | M | an..3000 |  |  | La firma digital no existe<br>La firma digital no cumple los requisitos de vigencia. | ERROR | 1059 (ERROR) El XML no contiene firma digital.<br>2336 (ERROR) Ocurrió un error en el proceso de validación de la firma digital |

---

## 5. Reglas de cardinalidad y condicionales

### 5.1 Cardinalidades

| Elemento / bloque | Cardinalidad | Regla |
|---|---|---|
| `cac:DespatchLine` (bienes a transportar) | 1..n | Ítem incremental; cada ítem se numera con `cbc:ID` (campo 20) |
| `cac:AdditionalDocumentReference` (documento relacionado) | 1..n con límites | Solo se permite **1** documento; hasta **2** si algún tipo es `31`, `65`, `66`, `67`, `68` o `69`; **más de 2** si se ingresa el tipo `09` con serie electrónica (título del bloque en la fuente) |
| `cac:AttachedTransportEquipment` (vehículos secundarios) | 0..2 | "Solo corresponde consignar hasta dos vehículos secundarios" (4389) |
| `cac:DriverPerson` con `cbc:JobTitle='Secundario'` | 0..2 | "Solo debe consignar informacion de hasta dos conductores secundarios" (4376) |
| `cac:DriverPerson` con `cbc:JobTitle='Principal'` | 0..1 | "Solo debe existir informacion de un conductor principal" (3358) |
| `cac:SpecialInstructions` (indicadores) | 0..n, sin repetir | "El indicador no debe repetirse" (3344); un indicador por tag y con formato `SUNAT_Envio_*` (3388) |
| `cac:OriginatorCustomerParty` (quien paga) | 0..1 | Solo se completa cuando paga un tercero (título del bloque) |
| `cac:Consignment` (subcontratación) | 0..1 | Solo si existe el indicador de transporte subcontratado |
| `ds:Signature` | 1..1 | Firma digital obligatoria (1059 / 2336) |

### 5.2 Reglas de negocio y condicionales (con código de retorno)

| Regla | Códigos de retorno |
|---|---|
| El transportista no debe ser igual al remitente | 2560 |
| El documento relacionado no puede estar duplicado (`Código de tipo de documento relacionado` + `cbc:ID` no se repiten) | 3340 |
| Si se ingresa un documento relacionado tipo `49` (solo GRE-Remitente) u `80`, debe existir al menos un ítem con partida arancelaria | 3352 |
| La fecha de inicio del traslado debe ser mayor o igual a la fecha de emisión del documento | 3343 |
| Solo corresponde ingresar hasta 2 documentos relacionados si uno de los tipos es `31`, `65`, `66`, `67`, `68` o `69` | 3345 |
| Solo corresponde ingresar más de 1 documento relacionado si algún tipo es `31`, `65`, `66`, `67`, `68`, `69` o `09` (solo guía electrónica) | 3346 |
| El RUC del transportista debe estar activo / habido | 3349 / 3350 |
| Debe existir información del conductor principal | 3357 |
| Solo debe existir información de un conductor principal | 3358 |
| No debe repetirse la información de conductores | 3362 |
| El código de ubigeo de partida / llegada debe existir | 3363 / 3368 |
| No debe consignar el Tipo de evento | 3374 |
| El indicador no debe repetirse / no cumple el formato | 3344 / 3388 |
| El RUC del subcontratador no existe; debe ser `6`; no puede ser igual al transportista | 3389 / 3391 / 3390 |
| Si existe una GRE-Remitente como documento relacionado, debe existir; el destinatario debe coincidir con el de la GRE-Remitente | 3433 / 3434 |
| Debe consignarse el detalle de los bienes a transportar (3435) y, en el caso contrario, no debe consignarse (4434) | 3435 / 4434 |
| El número de ítem debe ser mayor a cero; solo se permite `0` para consignar anotación opcional sobre los bienes | 3458 |
| Si existe descripción del documento relacionado debe existir información del código y número del documento relacionado | 4410 |
| Debe consignarse el indicador de pagador de flete | 4388 |
| Tipo de evento: `1-Transbordo no programado`, `2-Imposibilidad arribo`, `3-Imposibilidad entrega` | 3374 (prohíbe su uso al enviar el XML) |
| Coherencia de la numeración con el nombre del archivo (RUC, serie y correlativo) | 1034 / 1035 / 1036 |
| Vigencia del comprobante y fecha de presentación | 1032 / 1033 / 2108 / 2329 |

### 5.3 Reglas normativas de la R.S. N.° 123-2022/SUNAT

| Regla | Fuente |
|---|---|
| La GRE-Remitente debe emitirse **antes** de la entrega de los bienes al transportista | R.S. 123-2022-SUNAT, Art. 4.2.a |
| El transportista puede emitir **una sola GRE-Transportista** para bienes asociados a **más de una GRE-Remitente**, siempre que el destinatario, el punto de partida y el punto de llegada sean los mismos | R.S. 123-2022-SUNAT, Art. 3.3.b |
| El transportista puede consignar **más de una GRE-Remitente** en una GRE-Transportista, siempre que el remitente, el destinatario, el punto de partida y el punto de llegada sean los mismos | R.S. 123-2022-SUNAT, Art. 3.3.b |
| No se exige GRE-Transportista cuando el traslado se realiza en vehículos de categorías **L** y **M1** (RNV) y se ha emitido la GRE-Remitente | R.S. 123-2022-SUNAT, Art. 21.2.2 |
| En el traslado desde la ZOFRATACNA hasta la Zona Comercial de Tacna, la guía del remitente puede ser sustituida por el documento del Art. 21.1.2.f), pero el transportista **permanece obligado a emitir la guía de remisión del transportista** | R.S. 123-2022-SUNAT, Art. 21.2.1 |
| El indicador `SUNAT_Envio_IndicadorTrasladoVehiculoM1L` (traslado en vehículos M1 o L) existe en el estándar **Remitente**, no en el estándar **Transportista** → en la GRE-Transportista su uso es **[NO VERIFICADO]** | Hoja "Guía-Remitente2_0" (campo 36) vs. hoja "Guía-Transportista2_0" |
| La denominación "GRE-Transportista simplificada" **no aparece** en las fuentes entregadas → **[NO VERIFICADO]**; lo verificable es el supuesto del Art. 3.3.b (una GRE-Transportista que agrupa varias GRE-Remitente) | Búsqueda sobre `RS_123-2022_GRE.txt`, `R-123-2022-SUNAT.txt` y hojas XLSX |

---

## 6. Ejemplo XML mínimo válido

Valores de ejemplo con marcadores `<...>` donde la fuente exige un dato dependiente del contribuyente.
La firma digital se representa con el marcador `<!-- ds:Signature -->` (contenido de
`ext:UBLExtensions/ext:UBLExtension/ext:ExtensionContent`).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<DespatchAdvice xmlns="urn:oasis:names:specification:ubl:schema:xsd:DespatchAdvice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2"
                xmlns:ext="urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2"
                xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  <ext:UBLExtensions>
    <ext:UBLExtension>
      <ext:ExtensionContent>
        <!-- ds:Signature -->
      </ext:ExtensionContent>
    </ext:UBLExtension>
  </ext:UBLExtensions>

  <cbc:UBLVersionID>2.1</cbc:UBLVersionID>
  <cbc:CustomizationID>2.0</cbc:CustomizationID>
  <cbc:ID>V001-00000001</cbc:ID>
  <cbc:IssueDate>2025-04-21</cbc:IssueDate>
  <cbc:IssueTime>10:30:00</cbc:IssueTime>
  <cbc:DespatchAdviceTypeCode listAgencyName="PE:SUNAT"
                              listName="Tipo de Documento"
                              listURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo01">31</cbc:DespatchAdviceTypeCode>
  <cbc:Note>Traslado de mercaderia</cbc:Note>

  <cac:DespatchSupplierParty>
    <cac:Party>
      <cac:PartyIdentification>
        <cbc:ID schemeID="6"
                schemeName="Documento de Identidad"
                schemeAgencyName="PE:SUNAT"
                schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06">20123456789</cbc:ID>
      </cac:PartyIdentification>
      <cac:PartyLegalEntity>
        <cbc:RegistrationName>TRANSPORTES DEL SUR SAC</cbc:RegistrationName>
      </cac:PartyLegalEntity>
    </cac:Party>
  </cac:DespatchSupplierParty>

  <cac:DeliveryCustomerParty>
    <cac:Party>
      <cac:PartyIdentification>
        <cbc:ID schemeID="6"
                schemeName="Documento de Identidad"
                schemeAgencyName="PE:SUNAT"
                schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06">20999999999</cbc:ID>
      </cac:PartyIdentification>
      <cac:PartyLegalEntity>
        <cbc:RegistrationName>DESTINATARIO FINAL SAC</cbc:RegistrationName>
      </cac:PartyLegalEntity>
    </cac:Party>
  </cac:DeliveryCustomerParty>

  <cac:Shipment>
    <cbc:ID>SUNAT_Envio</cbc:ID>
    <cbc:GrossWeightMeasure unitCode="KGM">1000.000</cbc:GrossWeightMeasure>
    <cbc:SpecialInstructions>SUNAT_Envio_IndicadorTrasladoTotal</cbc:SpecialInstructions>

    <cac:ShipmentStage>
      <cac:CarrierParty>
        <cac:PartyLegalEntity>
          <cbc:CompanyID>AB0123</cbc:CompanyID>
        </cac:PartyLegalEntity>
      </cac:CarrierParty>

      <cac:DriverPerson>
        <cbc:JobTitle>Principal</cbc:JobTitle>
        <cbc:ID schemeID="6"
                schemeName="Documento de Identidad"
                schemeAgencyName="PE:SUNAT"
                schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06">45678912</cbc:ID>
        <cbc:FirstName>JUAN CARLOS</cbc:FirstName>
        <cbc:FamilyName>PEREZ RAMOS</cbc:FamilyName>
        <cac:IdentityDocumentReference>
          <cbc:ID>Q12345678</cbc:ID>
        </cac:IdentityDocumentReference>
      </cac:DriverPerson>

      <cac:TransitPeriod>
        <cbc:StartDate>2025-04-22</cbc:StartDate>
      </cac:TransitPeriod>
    </cac:ShipmentStage>

    <cac:Delivery>
      <cac:Despatch>
        <cac:DespatchParty>
          <cac:PartyIdentification>
            <cbc:ID schemeID="6"
                    schemeName="Documento de Identidad"
                    schemeAgencyName="PE:SUNAT"
                    schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06">20111111111</cbc:ID>
          </cac:PartyIdentification>
          <cac:PartyLegalEntity>
            <cbc:RegistrationName>REMITENTE DE MERCADERIA SAC</cbc:RegistrationName>
          </cac:PartyLegalEntity>
        </cac:DespatchParty>
        <cac:DespatchAddress>
          <cbc:ID schemeAgencyName="PE:INEI" schemeName="Ubigeos">150131</cbc:ID>
          <cac:AddressLine>
            <cbc:Line>AV. PRINCIPIAL 123 - LIMA</cbc:Line>
          </cac:AddressLine>
        </cac:DespatchAddress>
      </cac:Despatch>
      <cac:DeliveryAddress>
        <cbc:ID schemeAgencyName="PE:INEI" schemeName="Ubigeos">150131</cbc:ID>
        <cac:AddressLine>
          <cbc:Line>AV. DESTINO 456 - LIMA</cbc:Line>
        </cac:AddressLine>
      </cac:DeliveryAddress>
    </cac:Delivery>

    <cac:TransportHandlingUnit>
      <cac:TransportEquipment>
        <cbc:ID>ABC-123</cbc:ID>
      </cac:TransportEquipment>
    </cac:TransportHandlingUnit>
  </cac:Shipment>

  <cac:AdditionalDocumentReference>
    <cbc:DocumentType>Guia de Remision Remitente</cbc:DocumentType>
    <cbc:DocumentTypeCode listAgencyName="PE:SUNAT"
                         listName="Documento relacionado al transporte"
                         listURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo61">09</cbc:DocumentTypeCode>
    <cbc:ID>T001-00000001</cbc:ID>
    <cac:IssuerParty>
      <cac:PartyIdentification>
        <cbc:ID schemeID="6"
                schemeName="Documento de Identidad"
                schemeAgencyName="PE:SUNAT"
                schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06">20111111111</cbc:ID>
      </cac:PartyIdentification>
    </cac:IssuerParty>
  </cac:AdditionalDocumentReference>

  <cac:DespatchLine>
    <cbc:ID>0</cbc:ID>
    <cac:OrderLineReference>
      <cbc:LineID>1</cbc:LineID>
    </cac:OrderLineReference>
    <cac:Item>
      <cbc:Description>MERCANCIA VARIADA - 1 PALLET</cbc:Description>
    </cac:Item>
  </cac:DespatchLine>
</DespatchAdvice>
```

**Notas sobre el ejemplo**

- `cbc:DespatchLine/cbc:ID` = `0` solo es admisible para consignar la anotación opcional sobre los bienes
  a transportar (campo 51 / código 3458). Si se envía el detalle de bienes con cantidad mayor a cero, el
  `cbc:ID` debe ser mayor que cero y `cbc:DeliveredQuantity` es obligatorio.
- El `cac:AdditionalDocumentReference` con código `09` y **serie electrónica** es el caso que habilita más de
  un documento relacionado (3346) y exige que el detalle de bienes no se consigne cuando la serie no empieza
  con número (4434).
- `cac:TransportEvent/cbc:TransportEventTypeCode` se omite: el tipo de evento no debe enviarse en el XML de la
  guía (3374).
- `cac:Consignment` (subcontratación) y `cac:OriginatorCustomerParty` (quien paga) se omiten porque son
  condicionales.
- Los URI de namespace y la secuencia exacta de elementos son **[NO VERIFICADO]** (las fuentes no incluyen el
  XSD); el ejemplo sigue la secuencia canónica de UBL 2.1 para `DespatchAdvice`.

---

## 7. Comparativa: GRE-Remitente vs GRE-Transportista

Base: hoja "Guía-Remitente2_0" y hoja "Guía-Transportista2_0" del `ValidacionesGREv20250421_0.xlsx`
(misma versión) y Anexo VII a) / b) de la R.S. 123-2022.

| Aspecto | GRE-Remitente (tipo `09`) | GRE-Transportista (tipo `31`) |
|---|---|---|
| Tipo de documento (Catálogo N.° 01) | `09` | `31` |
| Formato de la serie | `T###-NNNNNNNN` → `[T][A-Z0-9]{3}-[0-9]{1,8}` | `V###-NNNNNNNN` → `[V][A-Z0-9]{3}-[0-9]{1,8}` |
| Emisor del documento (`cac:DespatchSupplierParty`) | Remitente (campos 8-9) | Transportista (campos 8-9) |
| Identidad del remitente | En `cac:DespatchSupplierParty` (campos 8-9) | En `cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty` (campos 16-17) |
| Campos del estándar | 90 (81 filas; 82–90 comparten fila) | 64 (63 filas; 27–28 comparten fila) |
| Bloque "Datos del Transportista" | Remitente conoce al transportista: campos 43-46, todos `C` | El transportista es el emisor: campos 8-11, con RUC y razón social `M` |
| Vehículo principal (placa) | Condicional (campo 47) | **Obligatorio** (campo 35, `M`) |
| Conductor principal | Condicional (campos 53-56) | **Obligatorio** (campos 41-44, `M`) |
| Motivo de traslado | **Obligatorio** (campo 22 → `cac:Shipment/cbc:HandlingCode`, Catálogo N.° 20 / D05) | **No existe** en el estándar Transportista |
| Modalidad de traslado | **Obligatorio** (campo 32 → `cac:Shipment/cac:ShipmentStage/cbc:TransportModeCode`, Catálogo N.° 18) | **No existe** en el estándar Transportista |
| Indicador de traslado en vehículos M1 o L | Sí (campo 36 → `SUNAT_Envio_IndicadorTrasladoVehiculoM1L`) | **No consta** → **[NO VERIFICADO]** |
| Fecha de entrega de bienes al transportista | Sí (campo 34) | No consta (solo fecha de inicio del traslado, campo 50) |
| Peso bruto de los ítems seleccionados | Sí (campo 24) | No consta (solo peso bruto total de la carga, campo 52) |
| Proveedor / comprador | Sí (`cac:SellerSupplierParty` campos 17-18, `cac:BuyerCustomerParty` campos 19-20) | **No existen** |
| Puerto / aeropuerto de embarque | Sí (campos 71-73 → `cac:FirstArrivalPortLocation`) | **No existe** |
| Documentos relacionados | "Documento relacionado - incremental" (campos 11-14) | Máximo 1; hasta 2 con `31`, `65`, `66`, `67`, `68`, `69`; más de 2 con `09` serie electrónica (campos 12-15) |
| Datos de quien paga el servicio | **No existen** | Sí (campos 62-63 → `cac:OriginatorCustomerParty`) |
| Datos de empresa que subcontrata | **No existen** | Sí (campos 60-61 → `cac:Shipment/cac:Consignment`) |
| Indicador de pagador de flete | No consta | Sí (campo 58 → `SUNAT_Envio_IndicadorPagadorFlete_Remitente` / `_Subcontratador` / `_Tercero`) |
| Indicador de registro de vehículos y conductores del transportista | Sí (campo 40) | No consta |
| Indicador de traslado en contenedor del Manifiesto de Carga | Sí (campo 41) | No consta |
| Firma digital | Campo 84, firma del remitente | Campo 64, firma del transportista |

### Diferencias de tag entre el XLSX 2025 y el Anexo VII b) de la R.S. 123-2022 (Transportista)

| Dato | Guía oficial `ValidacionesGREv20250421_0.xlsx` | Anexo VII b) R.S. 123-2022 |
|---|---|---|
| TUC / Constancia de Inscripción Vehicular del vehículo principal (campo 36) | `cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:ApplicableTransportMeans/cbc:RegistrationNationalityID` | `cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:ShipmentDocumentReference/cbc:ID` |
| Identificador del subcontratador (campo 60) | `cac:Shipment/cac:Consignment/cac:LogisticsOperatorParty/cac:PartyIdentification/cbc:ID` | `cac:Shipment/cac:Consignment/cac:LogisticsOperatorParty/cbc:CustomerAssignedAccountID` |
| Número de orden del ítem (campo 20) | `cac:DespatchLine/cbc:ID` | `cac:DespatchLine/cbc:ID` + `cac:DespatchLine/cac:OrderLineReference/cbc:LineID` |

> En caso de implementarse contra la R.S. 123-2022 (no contra el XLSX 2025), debe prevailecer el tag del XLSX
> vigente publicado por la SUNAT, ya que es el que valida el servicio de recepción.

---

## 8. Referencias

### Fuentes utilizadas (archivos locales)

| Archivo | Contenido |
|---|---|
| `sunat/xlsx/Gu_a_Transportista2_0.txt` | Hoja "Guía-Transportista2_0" del XLSX oficial (64 campos, tags, validaciones, códigos de retorno) |
| `sunat/xlsx/Gu_a_Remitente2_0.txt` | Hoja "Guía-Remitente2_0" del mismo XLSX (90 campos) |
| `sunat/xlsx/Cat_logos.txt` | Catálogos oficiales (Catálogo N.° 01, 03, 06, 13, 20, 25, 55, 61, 65, D05, …) |
| `sunat/xlsx/C_digosRetorno.txt` | Catálogo de códigos de retorno (incluye la ruta `ds:Signature`) |
| `sunat/xlsx/Control_de_Cambios.txt` | Control de cambios de las guías (vigencia de la versión 2025-04-21) |
| `sunat/xlsx/General.txt` | Consideraciones generales del XLSX |
| `sunat/xlsx/Palabras_Clave.txt` | Palabras clave de la guía |
| `sunat/R-123-2022-SUNAT.txt` | R.S. N.° 123-2022/SUNAT, Anexo VII a) Remitente y b) Transportista |
| `sunat/RS_123-2022_GRE.txt` | Texto íntegro de la R.S. 123-2022/SUNAT (Arts. 3.3.b, 4.2.a, 18, 20, 21.2.2) |
| `sunat/anexo12-2017.txt` | Anexo 12 (estructura UBL de la GRE, ruta de firma digital) |
| `sunat/ValidacionesGREv20250421_0.xlsx` | Archivo XLSX oficial de validaciones (origen de los `.txt` anteriores) |

### Referencias normativas y-catálogos

| Referencia | Ubicación |
|---|---|
| R.S. N.° 123-2022/SUNAT (Reglamento de la Guía de Remisión Electrónica) | El Peruano, 12/07/2022 — `www.sunat.gob.pe` (citado en el propio resolutivo) |
| Guías y manuales de consulta de SUNAT | `www.sunat.gob.pe` |
| Catálogos SUNAT (`catalogo01`, `catalogo06`, `catalogo13`, `catalogo25`, `catalogo55`, `catalogo61`, `catalogo65`, `catalogoD05`) | `urn:pe:gob:sunat:cpe:see:gem:catalogos:<catálogo>` |
| UN/CEFACT Rec 20 — códigos de unidades de medida | `unece.org` |
| Código GTIN — estructura GS1 | `gtin.org` / `gs1.org` |
| Ubigeos (INEI) | `inei.gob.pe` |
| Catálogo de bienes normalizados / productos SUNAT | Portal de SUNAT (consulta por RUC del contribuyente) |

> Los URI de los catálogos SUNAT que aparecen en las fuentes son `urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogoXX`;
> las URLs públicas de descarga de cada catálogo **[NO VERIFICADO]** (no constan en los archivos entregados).
