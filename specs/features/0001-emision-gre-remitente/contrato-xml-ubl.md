# Contrato XML UBL 2.1 - Guía de Remisión Remitente (GRE)

## 1. Identificación del documento

| Campo | Valor | Observación |
|---|---|---|
| Nombre del estándar | UBL (Universal Business Language) | Oasis |
| Versión UBL | 2.1 | Confirmado en fuente (/DespatchAdvice/cbc:UBLVersionID = 2.1) |
| CustomizationID | 2.0 | Confirmado en fuente (/DespatchAdvice/cbc:CustomizationID = 2.0) |
| Tipo de documento (código) | 09 | Guía de remisión remitente. Catálogo 01. |
| Prefijo de serie | T | Para GRE Remitente. Para GRE Transportista es V. |
| Formato de ID | T###-NNNNNNNN | Regex exacta: [T][A-Z0-9]{3}-[0-9]{1,8}. Longitud máxima n..13. |
| Encoding del documento | ISO-8859-1 / Latin-1, sin BOM | [NO VERIFICADO] - No se encuentra explícitamente indicado en los archivos fuente. Se recomienda tratar como ISO-8859-1/Latin-1 sin BOM por convención SUNAT para XMLs CPE; debe confirmarse con esquema/XSL. |

## 2. Namespaces del XML

| Prefijo | Namespace | Uso |
|---|---|---|
| (default) | urn:oasis:names:specification:ubl:schema:xsd:DespatchAdvice-2 | Elemento raíz DespatchAdvice |
| cbc | urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2 | Componentes básicos |
| cac | urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2 | Componentes agregados |
| ext | urn:oasis:names:specification:ubl:schema:xsd:CommonExtensionComponents-2 | Extensiones UBL |
| ds | http://www.w3.org/2000/09/xmldsig# | Firma digital XMLDSig |
| SUNAT extensiones | [NO VERIFICADO] | No se identifica namespace específico de extensiones SUNAT en las fuentes. Se espera ext:UBLExtensions/UBLExtension/ExtensionContent para alojar ds:Signature según estructura UBL. |

## 3. Estructura del documento (tags UBL usados)

`	ext
DespatchAdvice (default namespace)
├── ext:UBLExtensions
│   └── ext:UBLExtension
│       └── ext:ExtensionContent
│           └── ds:Signature  (firma digital - ver sección 7)
├── cbc:UBLVersionID (2.1)
├── cbc:CustomizationID (2.0)
├── cbc:ID (T###-NNNNNNNN)
├── cbc:IssueDate (YYYY-MM-DD)
├── cbc:IssueTime (hh:mm:ss)
├── cbc:DespatchAdviceTypeCode (09) [@listAgencyName, @listName, @listURI]
├── cbc:Note (an..250, opcional)
├── cac:DespatchSupplierParty
│   └── cac:Party
│       ├── cac:PartyIdentification
│       │   └── cbc:ID [@schemeID='6', @schemeName, @schemeAgencyName, @schemeURI]
│       └── cac:PartyLegalEntity
│           └── cbc:RegistrationName (an..250)
├── cac:AdditionalDocumentReference (incremental, opcional)
│   ├── cbc:DocumentType (an..120)
│   ├── cbc:DocumentTypeCode (Catálogo 61) [atributos list*]
│   ├── cbc:ID (an..100)
│   └── cac:IssuerParty
│       └── cac:PartyIdentification
│           └── cbc:ID [@schemeID='6', atributos scheme*]
├── cac:DeliveryCustomerParty
│   └── cac:Party
│       ├── cac:PartyIdentification
│       │   └── cbc:ID [@schemeID Catálogo 06, atributos scheme*]
│       └── cac:PartyLegalEntity
│           └── cbc:RegistrationName (an..250)
├── cac:SellerSupplierParty (condicional por motivo)
│   └── cac:Party
│       ├── cac:PartyIdentification
│       │   └── cbc:ID [@schemeID, atributos scheme*]
│       └── cac:PartyLegalEntity
│           └── cbc:RegistrationName (an..250)
├── cac:BuyerCustomerParty (condicional por motivo)
│   └── cac:Party
│       ├── cac:PartyIdentification
│       │   └── cbc:ID [@schemeID, atributos scheme*]
│       └── cac:PartyLegalEntity
│           └── cbc:RegistrationName (an..250)
└── cac:Shipment
    ├── cbc:ID ("SUNAT_Envio")
    ├── cbc:HandlingCode (Catálogo 20, motivo) [atributos list*]
    ├── cbc:HandlingInstructions (an..100, condicional motivo 13)
    ├── cbc:NetWeightMeasure [@unitCode='KGM'] (condicional import/export/19)
    ├── cbc:Information (an..250, sustento diferencia peso)
    ├── cbc:GrossWeightMeasure [@unitCode 'KGM'/'TNE'] (obligatorio)
    ├── cbc:TotalTransportHandlingUnitQuantity (n..13, condicional)
    ├── cbc:SpecialInstructions (indicadores SUNAT_Envio_*)
    ├── cac:ShipmentStage
    │   ├── cbc:TransportModeCode (Catálogo 18) [atributos list*]
    │   ├── cac:TransitPeriod
    │   │   ├── cbc:StartDate (YYYY-MM-DD, fecha inicio traslado / fecha entrega al transportista)
    │   │   └── (según modalidad/indicadores)
    │   ├── cac:CarrierParty (condicional modalidad 01 sin indicador M1/L)
    │   │   ├── cac:PartyIdentification
    │   │   │   └── cbc:ID [@schemeID='6']
    │   │   ├── cac:PartyLegalEntity
    │   │   │   ├── cbc:RegistrationName
    │   │   │   └── cbc:CompanyID (N° Registro MTC) / (autorización transportista con @schemeID entidad)
    │   │   └── cac:AgentParty
    │   │       └── cac:PartyLegalEntity
    │   │           └── cbc:CompanyID [@schemeID entidad autorizadora] (autorización especial transportista)
    │   ├── cac:TransportEvent
    │   │   └── cbc:TransportEventTypeCode (1,2,3) - uso exclusivo comunicación eventos [NO DEBE ENVIARSE]
    │   └── cac:DriverPerson (incremental hasta 2, principal/secundario según reglas)
    │       ├── cbc:JobTitle ('Principal'/'Secundario')
    │       ├── cbc:ID [@schemeID Catálogo 06]
    │       ├── cbc:FirstName
    │       ├── cbc:FamilyName
    │       └── cac:IdentityDocumentReference
    │           └── cbc:ID (licencia)
    ├── cac:Delivery
    │   ├── cac:Despatch
    │   │   ├── cac:DespatchParty
    │   │   │   └── cac:AgentParty
    │   │   │       └── cac:PartyLegalEntity
    │   │   │           └── cbc:CompanyID [@schemeID entidad] (autorización especial remitente - ítem 10)
    │   │   └── cac:DespatchAddress
    │   │       ├── cbc:ID [@schemeAgencyName='PE:INEI', @schemeName='Ubigeos'] (ubigeo partida)
    │   │       ├── cbc:AddressTypeCode [@listID RUC asociado, @listAgencyName='PE:SUNAT', @listName='Establecimientos anexos'] (código establecimiento)
    │   │       ├── cac:AddressLine
    │   │       │   └── cbc:Line
    │   │       └── cac:LocationCoordinate
    │   │           ├── cbc:LatitudeDegreesMeasure [@unitCode='DD']
    │   │           └── cbc:LongitudeDegreesMeasure [@unitCode='DD']
    │   └── cac:DeliveryAddress
    │       ├── cbc:ID [@schemeAgencyName='PE:INEI', @schemeName='Ubigeos'] (ubigeo llegada)
    │       ├── cbc:AddressTypeCode [@listID RUC asociado, @listAgencyName='PE:SUNAT', @listName='Establecimientos anexos']
    │       ├── cac:AddressLine
    │       │   └── cbc:Line
    │       └── cac:LocationCoordinate
    │           ├── cbc:LatitudeDegreesMeasure [@unitCode='DD']
    │           └── cbc:LongitudeDegreesMeasure [@unitCode='DD']
    ├── cac:TransportHandlingUnit
    │   ├── cac:Package
    │   │   ├── cbc:ID (número contenedor)
    │   │   └── cbc:TraceID (número precinto)
    │   └── cac:TransportEquipment
    │       ├── cbc:ID (placa vehículo principal)
    │       ├── cac:ApplicableTransportMeans
    │       │   └── cbc:RegistrationNationalityID (TUC/CCH)
    │       ├── cac:ShipmentDocumentReference
    │       │   └── cbc:ID [@schemeID entidad autorizadora] (autorización vehículo principal)
    │       └── cac:AttachedTransportEquipment (incremental hasta 2)
    │           ├── cbc:ID (placa)
    │           ├── cac:ApplicableTransportMeans
    │           │   └── cbc:RegistrationNationalityID
    │           └── cac:ShipmentDocumentReference
    │               └── cbc:ID [@schemeID entidad autorizadora]
    └── cac:FirstArrivalPortLocation
        ├── cbc:ID [@schemeAgencyName='PE:SUNAT', @schemeName, @schemeURI] (puerto/aeropuerto)
        ├── cbc:LocationTypeCode ('1'=Puerto, '2'=Aeropuerto)
        └── cbc:Name
└── cac:DespatchLine (incremental)
    ├── cbc:ID (número orden item)
    ├── cbc:DeliveredQuantity [@unitCode]
    ├── cac:OrderLineReference
    │   └── cbc:LineID
    └── cac:Item
        ├── cbc:Description
        ├── cac:SellersItemIdentification
        │   └── cbc:ID
        ├── cac:CommodityClassification
        │   └── cbc:ItemClassificationCode [@listID='UNSPSC', @listAgencyName='GS1 US', @listName='Item Classification']
        ├── cac:StandardItemIdentification
        │   └── cbc:ID [@schemeID 'GTIN-*']
        └── cac:AdditionalItemProperty (múltiples)
            ├── cbc:Name
            ├── cbc:NameCode (Catálogo 55) [atributos list*]
            └── cbc:Value (según concepto: partida arancelaria, indicador bien normalizado, DAM/DS, MC, contenedor, precinto, etc.)
## 4. Tabla de datos campo por campo (validaciones y códigos de retorno)

> Nota: Se recorren las filas de la fuente \Gu_a_Remitente2_0.txt\ (hoja "Guía-Remitente2_0", header línea 2). Las "continuaciones" (atributos @list*/@scheme*) se han unido a su fila madre. Cuando hay múltiples validaciones para el mismo dato se separan con <br>. Los códigos de retorno son los más relevantes.

### Datos de guía de remisión

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Versión del UBL | Global | M | an..3 | 2.1 | /DespatchAdvice/cbc:UBLVersionID | No existe el Tag UBL o es vacío <br> El valor del Tag UBL es diferente a '2.1' | ERROR | 2111<br>2110 |
| 2 | Versión de la estructura del documento | Global | M | an..3 | 2.0 | /DespatchAdvice/cbc:CustomizationID | No existe el Tag UBL o es vacío <br> El valor del Tag UBL es diferente a '2.0' | ERROR | 2113<br>2112 |
| 3 | Numeracion, conformada por serie y Número correlativo | Global | M | an..13 | T###-NNNNNNNN | /DespatchAdvice/cbc:ID | El formato del Tag UBL es diferente a: [T][A-Z0-9]{3}-[0-9]{1,8} <br> El número de serie del Tag UBL es diferente al número de serie del archivo <br> El número de documento del Tag UBL es diferente al número de documento del archivo <br> El valor del Tag UBL se encuentra en el listado con indicador de estado igual a 1 <br> El valor del Tag UBL se encuentra en el listado con indicador de estado igual a 2 | ERROR | 1001<br>1035<br>1036<br>1033<br>1032 |
| 4 | Fecha de emisión | Global | M | an10 | YYYY-MM-DD | /DespatchAdvice/cbc:IssueDate | El valor del Tag UBL no cumple con el formato establecido <br> La diferencia entre la fecha de recepción del XML y el valor del Tag UBL es mayor al límite del listado (1 día) <br> El valor del Tag UBL es mayor a la fecha de recepción (TODAY) o es 12/07/2022 | ERROR | 3436<br>2108<br>2329 |
| 5 | Hora de emisión | Global | M | an8 | hh:mm:ss | /DespatchAdvice/cbc:IssueTime | No existe el Tag UBL <br> El valor del Tag UBL no cumple con el formato establecido | ERROR | 3437<br>3438 |
| 6 | Tipo de documento (Guia) | Global | M | an2 | "09" / @listAgencyName="PE:SUNAT" / @listName="Tipo de Documento" / @listURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo01" | /DespatchAdvice/cbc:DespatchAdviceTypeCode | No existe el Tag UBL o es vacío <br> El valor del Tag UBL es diferente a '09' <br> Si existe atributo @listAgencyName y es diferente a 'PE:SUNAT' <br> Si existe atributo @listName y es diferente a 'Tipo de Documento' <br> Si existe atributo @listURI y es diferente a 'urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo01' | ERROR/OBSERV | 1050<br>1051<br>4251<br>4252<br>4253 |
| 7 | Observaciones (Texto) | Global | C | an..250 | Alfanumérico hasta 250, sin whitespace distinto a espacio (no \n,\t,\r), no solo espacios | /DespatchAdvice/cbc:Note | Si existe y formato inválido/vacío | OBSERV | 4186 |

### Datos del Remitente

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 8 | Tipo y número de documento de identidad del remitente | Global | M | an1 / an11 | "6" / n11 / schemeID="6" (@schemeName/@schemeAgencyName/@schemeURI) | /DespatchAdvice/cac:DespatchSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID] y cbc:ID | No existe atributo o vacío <br> schemeID diferente a '6' <br> Atributos scheme* incorrectos <br> Tag diferente al RUC del nombre del XML <br> ind_estado != '00' <br> ind_condicion != '00' <br> Solo afecto a rentas primera/segunda/cuarta categoría | ERROR/OBSERV | 2678<br>2511<br>4255/4256/4257<br>1034<br>2010<br>2011<br>3439 |
| 9 | Apellidos y nombres, denominación o razón social del remitente | Global | M | an..250 | Alfanumérico hasta 250, sin whitespace distinto a espacio | /DespatchAdvice/cac:DespatchSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName | No existe o vacío <br> Formato inválido | ERROR/OBSERV | 1037<br>4338 |
| 10 | Entidad emisora / Número de autorización especial - remitente | Global | C | an..50 / an2 | Alfanumérico 3-50 / Catálogo D-37 (@schemeID/@schemeName/@schemeAgencyName) | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty/cac:AgentParty/cac:PartyLegalEntity/cbc:CompanyID[@schemeID] | Formato inválido <br> Más de un PartyLegalEntity <br> schemeID no está en listado o vacío cuando hay número <br> Falta schemeID cuando hay número <br> Número vacío cuando hay schemeID <br> Atributos incorrectos | OBSERV/ERROR | 4369<br>3353<br>4395<br>4394<br>4397<br>4255/4256 |
### Documento relacionado - incremental

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 11 | Tipo de documento (Descripción) | Global | C | an..120 | Alfanumérico hasta 120 | /DespatchAdvice/cac:AdditionalDocumentReference/cbc:DocumentType | Falta si existe código <br> Falta código si existe descripción <br> Formato inválido | OBSERV | 4371<br>4410<br>4372 |
| 12 | Código del tipo de documento relacionado | Global | C | an2 | Catálogo 61 (@listAgencyName/@listName/@listURI) | /DespatchAdvice/cac:AdditionalDocumentReference/cbc:DocumentTypeCode | No válido para tipo guía <br> Falta número si hay código <br> Reglas por motivo (08/09/19/13) y códigos permitidos (50,52,91,09,49,80) <br> Más de uno 91 cuando motivo 19 <br> 91 con 50/52 simultáneos <br> 49/80 con ítem sin partida arancelaria (7020) <br> Atributos list* incorrectos | ERROR | 2692<br>3403<br>3440/3445/3493/3494/3463/3352<br>4251/4252/4253 |
| 13 | Número de documento relacionado | Global | C | an..100 | Según código (ver formatos) | /DespatchAdvice/cac:AdditionalDocumentReference/cbc:ID | Duplicado (código + número) <br> Falta código si hay número <br> Formatos por código: 01(F/[A-Z0-9]{3}-\d{1,8}|E001-\d{1,8}|\d{1,4}-\d{1,8}), 03, 04, 09, 12, 48, 49(num15), 50/52(DAM/DS según régimen/motivo), 80(num15), 81, 91(Manifiesto según tipo/vía), 71-78 <br> Serie empieza con número y debe estar autorizado (catálogo físico) <br> Serie empieza con letra y debe existir para emisor <br> GRE-Remitente con serie numérica debe estar en estado 'Baja - por cambio de destinatario' | ERROR | 3340<br>3376<br>3441 (múltiples casos)<br>3207<br>4382<br>2536 |
| 14 | Número de RUC del emisor del documento relacionado | Global | C | an11 / an1 | n11 / schemeID='6' (@scheme* ) | /DespatchAdvice/cac:AdditionalDocumentReference/cac:IssuerParty/cac:PartyIdentification/cbc:ID[@schemeID] | Obligatorio para 01,03,04,09,12,48 <br> Debe coincidir con remitente/destinatario según motivo/código <br> Formato numérico 11 <br> Debe existir en listado <br> schemeID debe ser 6 | ERROR/OBSERV | 3380<br>3381<br>3409<br>3408<br>3382<br>4255/4256/4257 |

### Datos del Destinatario

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 15 | Tipo y número de documento de identidad del destinatario | Global | M | an1 / an..15 | Catálogo 06 (@scheme*) / según tipo | /DespatchAdvice/cac:DeliveryCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID] | Atributo obligatorio si hay número <br> Valor en listado <br> Motivo 06/17/19 debe ser schemeID='6' <br> No igual a remitente según motivo <br> Debe ser igual a remitente si motivo 02/04/07 <br> Para motivo 08 debe coincidir con importador en DAM/DS <br> DNI 8 dígitos debe existir en RENIEC <br> RUC 11 dígitos activo/habido <br> Otros tipos alfanuméricos hasta 15 | ERROR/OBSERV | 2759<br>2760<br>3417<br>2555<br>2554<br>4435<br>2758<br>3444<br>3443<br>4373<br>4374<br>4255/4256/4257 |
| 16 | Apellidos y nombres, denominación o razón social del destinatario | Global | M | an..250 | Alfanumérico hasta 250, sin whitespace distinto a espacio | /DespatchAdvice/cac:DeliveryCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName | No existe o vacío <br> Formato inválido | ERROR/OBSERV | 2761<br>4152 |

### Datos del Proveedor / Comprador

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 17 | Tipo y número de documento de identidad del proveedor | Global | C | an1 / an..15 | Según motivo (02/07/13) / Catálogo 06 | /DespatchAdvice/cac:SellerSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID] | Obligatorio si motivo 02/07 y falta <br> Valores permitidos según motivo <br> No debe existir si motivo distinto a 02/07/13 <br> Si motivo 02/07 debe existir SellerSupplierParty con ID <br> Debe coincidir con emisor de doc relacionado 01/03/12 si aplica <br> No puede ser igual a remitente (motivo 02/07) <br> RUC/DNI/otros según formato y existencia/estado | ERROR/OBSERV | 2552<br>3447<br>4054<br>4375<br>3442<br>3448<br>2724<br>2621<br>2450<br>4051<br>4052<br>4255/4256/4257 |
| 18 | Razón social / nombres del proveedor | Global | C | an..250 | Alfanumérico hasta 250 | /DespatchAdvice/cac:SellerSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName | Obligatorio si hay ID proveedor (motivo 02/07/13) <br> Formato inválido <br> No debe existir si motivo distinto | ERROR/OBSERV | 3449<br>4106<br>4054 |
| 19 | Tipo y número de documento de identidad del comprador | Global | C | an1 / an..15 | Según motivo (03/13) / Catálogo 06 | /DespatchAdvice/cac:BuyerCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID] | Obligatorio si motivo 03 <br> Valores válidos <br> No debe existir si motivo distinto a 03/13 <br> No puede ser igual a remitente/destinatario (motivo 03) <br> RUC/DNI/otros según formato y existencia/estado | ERROR/OBSERV | 3331<br>3332<br>4377<br>4378<br>3333<br>3334<br>3335<br>3337<br>3336<br>3338<br>4379<br>4380<br>4255/4256/4257 |
| 20 | Razón social / nombres del comprador | Global | C | an..250 | Alfanumérico hasta 250 | /DespatchAdvice/cac:BuyerCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName | Obligatorio si hay ID comprador (motivo 03) <br> Formato inválido <br> No debe existir si motivo distinto | ERROR/OBSERV | 3339<br>4381<br>4377 |
### Datos del envío

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 21 | Identificador del traslado | Global | M | an11 | "SUNAT_Envio" | /DespatchAdvice/cac:Shipment/cbc:ID | SIN VALIDACIÓN (según fuente) | - | - |
| 22 | Motivo del traslado | Global | M | an2 | Catálogo 20 (@listAgencyName/@listName/@listURI) | /DespatchAdvice/cac:Shipment/cbc:HandlingCode | No existe o vacío <br> No está en listado <br> Atributos list* incorrectos | ERROR/OBSERV | 3404<br>3405<br>4251/4252/4253 |
| 23 | Descripción de motivo de traslado | Global | C | an..100 | Alfanumérico 3-100 (min 3 alfabéticos), no solo palabras clave, máximo 1 tag | /DespatchAdvice/cac:Shipment/cbc:HandlingInstructions | Obligatorio si motivo '13' <br> Formato inválido <br> Sin suficientes caracteres alfabéticos <br> Solo palabras clave <br> Más de un tag | ERROR/OBSERV/REST | 3457<br>4190<br>4190<br>4190<br>4440 |
| 24 | Peso bruto total de los ítems seleccionados (en KGM) / Unidad | Global | C | an..16 / an3 | n(12,3) / 'KGM' | /DespatchAdvice/cac:Shipment/cbc:NetWeightMeasure[@unitCode] | Obligatorio si motivo 08/09/19 y NO indicador traslado total DAM/DS <br> No debe existir si motivo distinto a 08/09/19 <br> Formato decimal positivo 12,3 <br> unitCode debe ser 'KGM' | ERROR/OBSERV | 4383<br>3395<br>3397<br>3398 |
| 25 | Sustento de la diferencia del peso bruto total de la carga respecto al peso de los ítems seleccionados | Global | C | an..250 | Alfanumérico hasta 250, sin whitespace distinto a espacio | /DespatchAdvice/cac:Shipment/cbc:Information | Obligatorio si motivo 08/09/19 y NO indicador traslado total DAM/DS <br> No debe existir si motivo distinto <br> Formato inválido | ERROR/OBSERV | 4387<br>3418<br>4428 |
| 26 | Peso bruto total de la carga / Unidad | Global | M | an..16 / an3 | n(12,3) / Catálogo 03 ('KGM','TNE') | /DespatchAdvice/cac:Shipment/cbc:GrossWeightMeasure[@unitCode] | No existe <br> Formato inválido <br> unitCode vacío o inválido | ERROR | 2880<br>2523<br>2881<br>2523 |
| 27 | Número de Bultos o Pallets | Global | C | an..13 | n..13 | /DespatchAdvice/cac:Shipment/cbc:TotalTransportHandlingUnitQuantity | Obligatorio si motivo 08/09/19 y NO existe ningún contenedor <br> Formato inválido | ERROR | 3419<br>3489 |
| 28 | Número de contenedor 1 | Global | C | an..17 | Alfanumérico hasta 17 (mayúsculas, '-' y '/') | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:Package/cbc:ID | Máximo 2 contenedores <br> No repetir contenedor <br> Formato inválido <br> Obligatorio si existe indicador contenedor en Manifiesto (True) <br> Debe coincidir con detalle y contar con documentos de transporte según listado | ERROR/OBSERV/REST | 3420<br>3421<br>4071<br>3487<br>4442 |
| 29 | Número de precinto 1 | Global | C | an..100 | Alfanumérico hasta 100 (mayúsculas/números, permite coma, no solo ceros) | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:Package/cbc:TraceID | Obligatorio si hay contenedor 1 y NO indicador contenedor en Manifiesto (Falso) <br> Obligatorio si motivo 19, hay contenedor 1, indicador contenedor True, y línea con indicador contenedor vacío '0' <br> No repetir precinto <br> Formato inválido | ERROR/OBSERV | 3422<br>3495<br>3423<br>4074 |
| 30 | Número de contenedor 2 | Global | C | an..17 | Igual que contenedor 1 | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:Package/cbc:ID | Formato inválido <br> Debe coincidir con detalle según listado si indicador contenedor True | OBSERV/REST | 4071<br>4442 |
| 31 | Número de precinto 2 | Global | C | an..100 | Igual que precinto 1 | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:Package/cbc:TraceID | Obligatorio bajo condiciones similares a contenedor 1 <br> No repetir <br> Formato inválido | ERROR/OBSERV | 3422<br>3495<br>3423<br>4074 |
| 32 | Modalidad de traslado | Global | M | an2 | Catálogo 18 (@listName/@listAgencyName/@listURI) | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cbc:TransportModeCode | No existe o vacío <br> No está en listado <br> Atributos incorrectos | ERROR/OBSERV | 2532<br>2773<br>4252/4251/4253 |
| 33 | Fecha Inicio de traslado | Global | C | an10 | YYYY-MM-DD | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:TransitPeriod/cbc:StartDate | Obligatorio según modalidad/indicadores <br> Debe ser >= fecha emisión <br> Formato inválido | ERROR | 3406<br>3343<br>3407 |
| 34 | Fecha de entrega de bienes al transportista | Global | C | an10 | YYYY-MM-DD | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:TransitPeriod/cbc:StartDate | Obligatorio en ciertos casos modalidad 01 <br> Debe ser >= fecha emisión <br> Formato inválido | OBSERV/ERROR | 4385<br>4386<br>3407 |
| 35-41 | Indicadores SUNAT_Envio_* (Transbordo programado, traslado M1/L, retorno envases, retorno vacío, traslado total DAM/DS, registro vehículos/conductores, traslado en contenedor Manifiesto) | Global | C | an..60 | Valores específicos con prefijo SUNAT_Envio_ | /DespatchAdvice/cac:Shipment/cbc:SpecialInstructions | Reglas por motivo/modalidad/documento relacionado <br> No repetir <br> Valores permitidos | ERROR | 3388<br>3344<br>3392<br>3485<br>3450<br>3451<br>3478 |
| 42 | Tipo de evento | Global | C | an1 | 1/2/3 | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:TransportEvent/cbc:TransportEventTypeCode | NO DEBE SER ENVIADO (uso exclusivo comunicación eventos) | ERROR | 3374 |
### Datos del Transportista / Vehículo / Conductor

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 43 | Número de RUC transportista (tipo/doc) | Global | C | an1 / an11 | '6' / n11 (@scheme*) | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:PartyIdentification/cbc:ID[@schemeID] | Obligatorio si modalidad 01 y sin indicador M1/L <br> No debe existir si modalidad 02 <br> schemeID debe ser '6' <br> Debe existir/activo/habido/afecto a tributos autorizados <br> No igual a remitente | ERROR/OBSERV | 2561<br>3347<br>2485<br>3348<br>3349<br>3350<br>3351<br>2560<br>4255/4256/4257 |
| 44 | Razón social transportista | Global | C | an..250 | Alfanumérico 3-250, sin whitespace distinto a espacio | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:PartyLegalEntity/cbc:RegistrationName | Obligatorio bajo condiciones modalidad 01 <br> No debe existir si modalidad 02 <br> Formato inválido | ERROR/OBSERV | 2563<br>3347<br>4165 |
| 45 | Número de Registro MTC | Global | C | an..20 | Alfanumérico hasta 20 (mayúsculas/números, sin espacios, no solo ceros) | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:PartyLegalEntity/cbc:CompanyID | Obligatorio según condiciones <br> No debe existir si modalidad 02 <br> Formato inválido <br> Debe existir en listado MTC <br> No más de un PartyLegalEntity | OBSERV/ERROR/REST | 4391<br>3347<br>4392<br>4393<br>3353 |
| 46 | Autorización especial transportista (entidad + número) | Global | C | an..50 / an2 | Alfanumérico 3-50 / Catálogo D-37 | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:AgentParty/cac:PartyLegalEntity/cbc:CompanyID[@schemeID] | Formato inválido <br> Requiere schemeID si hay número <br> schemeID no válido <br> Falta número si hay schemeID <br> Solo un PartyLegalEntity | OBSERV/ERROR | 4396<br>4394<br>4395<br>4397<br>3353<br>4255/4256 |
| 47 | Número de placa del vehículo principal | Global | C | an..8 | Alfanumérico 6-8 (mayúsculas/números, sin espacios/guión, no solo ceros) | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cbc:ID | Obligatorio si modalidad 02 y sin indicador M1/L <br> Obligatorio si modalidad 01 con indicador registro vehículos/conductores <br> No debe existir si indicador M1/L o condiciones prohibidas <br> Formato inválido <br> Debe existir en listado MTC | ERROR/OBSERV/REST | 2566<br>3354<br>2567<br>4398 |
| 48 | TUC / Certificado de Habilitación Vehicular | Global | C | an..15 | Alfanumérico 10-15 (mayúsculas/números, sin whitespace, no solo ceros) | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:ApplicableTransportMeans/cbc:RegistrationNationalityID | Obligatorio si indicador registro vehículos/conductores y hay placa <br> No debe existir si indicador registro no existe <br> Formato inválido <br> Debe existir en listado MTC | OBSERV/ERROR/REST | 4399<br>3452<br>3355<br>4400 |
| 49 | Autorización especial vehículo principal (entidad + número) | Global | C | an..50 / an2 | Alfanumérico 3-50 / Catálogo D-37 | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:ShipmentDocumentReference/cbc:ID[@schemeID] | No debe existir si indicador M1/L o condiciones prohibidas <br> Formato inválido <br> Solo un ShipmentDocumentReference <br> Requiere schemeID si hay número <br> schemeID no válido/falta número | OBSERV/ERROR | 3452<br>4406<br>3356<br>4403<br>4407<br>4405<br>4255/4256 |
| 50 | Placa vehículo secundario | Global | C | an..8 | Igual que principal | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:AttachedTransportEquipment/cbc:ID | Máximo 2 <br> No debe existir si indicador M1/L o sin indicador registro <br> Formato inválido <br> Debe existir en listado MTC | OBSERV/ERROR/REST | 4389<br>3453<br>2567<br>4398 |
| 51 | TUC vehículo secundario | Global | C | an..15 | Igual que principal | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:AttachedTransportEquipment/cac:ApplicableTransportMeans/cbc:RegistrationNationalityID | Obligatorio si hay placa secundaria y indicador registro <br> No debe existir si sin indicador registro <br> Formato inválido <br> Debe existir en listado MTC | OBSERV/ERROR/REST | 4399<br>3454<br>3355<br>4400 |
| 52 | Autorización especial vehículo secundario (entidad + número) | Global | C | an..50 / an2 | Igual que principal | /DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:AttachedTransportEquipment/cac:ShipmentDocumentReference/cbc:ID[@schemeID] | No debe existir bajo condiciones prohibidas <br> Formato inválido <br> Solo uno <br> Requiere schemeID/falta número | OBSERV/ERROR | 3454<br>4406<br>3356<br>4403<br>4407<br>4405 |
| 53 | Tipo de conductor (Principal) | Global | C | an9 | 'Principal' | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:JobTitle | Obligatorio bajo condiciones (modalidad 02 sin M1/L o modalidad 01 con indicador registro) <br> Solo 1 principal <br> No debe existir si indicador M1/L o sin condiciones | ERROR | 3357<br>3358<br>3455 |
| 54 | Tipo y número de documento de identidad conductor principal | Global | C | an1 / an..15 | Catálogo 06 (@scheme*) | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:ID[@schemeID] | Obligatorio bajo condiciones <br> No debe existir si indicador M1/L o sin condiciones <br> schemeID permitido distinto a '6' según listado? (catálogo 06) - debe ser distinto a '6' en validación para conductor principal <br> DNI 8 dígitos debe existir en RENIEC <br> Formato inválido | ERROR/OBSERV/REST | 2570<br>3455<br>2571<br>2569<br>3359<br>4255/4256/4257 |
| 55 | Apellidos y nombres conductor principal | Global | C | an..250 | Alfanumérico 3-250, sin whitespace distinto a espacio | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:FirstName / cbc:FamilyName | Obligatorio bajo condiciones <br> No debe existir si indicador M1/L o sin condiciones <br> Formato inválido | ERROR/OBSERV | 3360<br>3361<br>3455<br>4409 |
| 56 | Número de licencia conductor principal | Global | C | an..10 | Alfanumérico 9-10 (mayúsculas/números, no solo ceros) | /DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cac:IdentityDocumentReference/cbc:ID | Obligatorio bajo condiciones <br> No debe existir si indicador M1/L o sin condiciones <br> Formato inválido <br> Debe existir en listado MTC | ERROR/OBSERV/REST | 2572<br>3455<br>2573<br>4412 |
| 57-60 | Conductor secundario (hasta 2) - reglas similares con JobTitle 'Secundario' | Global | C | Ver 53-56 | Ver 53-56 | Mismos tags con JobTitle 'Secundario' | Máximo 2 <br> No deben repetirse por licencia <br> No debe existir si indicador M1/L o sin condiciones <br> Requiere licencia si hay otros datos <br> Formato/existencia según corresponda | ERROR/OBSERV/REST | 3362<br>3456<br>2570<br>2571<br>2568<br>3360<br>3361<br>4409<br>2572<br>2573<br>4412 |
### Dirección de partida / llegada, puerto/aeropuerto

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 61 | Ubigeo de partida | Global | M | an6 | Numérico 6 / Catálogo 13 (@schemeAgencyName='PE:INEI', @schemeName='Ubigeos') | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cbc:ID | No existe o vacío <br> Formato inválido <br> No existe en listado <br> Si motivo 08/19 con puerto/aeropuerto debe coincidir con ubigeo del puerto/aeropuerto <br> Si motivo 19 debe existir puerto/aeropuerto | ERROR | 2775<br>2776<br>3363<br>3364<br>3483<br>4256/4255 |
| 62 | Dirección completa y detallada de partida | Global | M | an..500 | Alfanumérico 3-500, sin whitespace distinto a espacio | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cac:AddressLine/cbc:Line | No existe o vacío <br> Formato inválido | ERROR/OBSERV | 2577<br>4076 |
| 63 | Número de RUC asociado al punto de partida | Global | C | an11 | n11 / @listID | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cbc:AddressTypeCode@listID | Obligatorio si hay código establecimiento <br> No debe ser igual a remitente si motivo 02/07/08 <br> Debe ser igual a remitente si motivo 04 <br> Si motivo 08 sin puerto/aeropuerto debe coincidir con depósito temporal en DAM/DS <br> Formato numérico 11 <br> Debe existir en listado | ERROR | 3410<br>3411<br>3414<br>3424<br>3409<br>3412 |
| 64 | Código de establecimiento de punto de partida | Global | C | an4 | n4 / Catálogo establecimientos (@listAgencyName/@listName) | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cbc:AddressTypeCode | Obligatorio si hay RUC asociado / si motivo 04 / si motivo 08 sin puerto/aeropuerto <br> Si RUC asociado es del remitente y código distinto a '0000' debe estar declarado <br> Si RUC asociado distinto a remitente debe estar declarado para ese RUC <br> Debe coincidir con ubigeo de partida <br> Atributos list* incorrectos | ERROR/OBSERV | 3365<br>3366<br>3367<br>4251/4252 |
| 65 | Punto de georreferencia de partida | Global | C | an..13 / an2 | n(3,8) / 'DD' | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cac:LocationCoordinate/cbc:LatitudeDegreesMeasure[@unitCode] / cbc:LongitudeDegreesMeasure[@unitCode] | Formato numérico hasta 3 enteros 8 decimales (positivo/negativo) <br> @unitCode 'DD' sin validación explícita | ERROR | 3413 |
| 66 | Ubigeo de llegada | Global | C | an6 | Numérico 6 / Catálogo 13 | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cbc:ID | Obligatorio si motivo distinto a '18' <br> Formato inválido <br> No existe en listado <br> Si motivo 09 con puerto debe coincidir con ubigeo puerto/aeropuerto | ERROR/OBSERV | 2775<br>2776<br>3368<br>3364<br>4256/4255 |
| 67 | Dirección completa y detallada de llegada | Global | C | an..500 | Alfanumérico 3-500, sin whitespace distinto a espacio | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cac:AddressLine/cbc:Line | Obligatorio si motivo distinto a '18' <br> Formato inválido | ERROR/OBSERV | 2574<br>4068 |
| 68 | Número de RUC asociado al punto de llegada | Global | C | an11 | n11 / @listID | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cbc:AddressTypeCode@listID | Obligatorio si hay código establecimiento <br> No debe ser igual a remitente si motivo 01/03/05/06/14/17 <br> Debe ser igual a remitente si motivo 04 <br> Debe ser igual a destinatario si motivo 19 <br> Formato numérico 11 <br> Debe existir en listado | ERROR | 3410<br>3411<br>3414<br>3488<br>3409<br>3412 |
| 69 | Código de establecimiento de punto de llegada | Global | C | an4 | n4 | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cbc:AddressTypeCode | Obligatorio si hay RUC asociado / si motivo 04 o 19 / si motivo 09 sin puerto <br> No debe existir si motivo '18' <br> Debe estar declarado según RUC asociado/remitente <br> Debe coincidir con ubigeo llegada | ERROR/OBSERV | 3369<br>3416<br>3370<br>3371<br>4251/4252 |
| 70 | Punto de georreferencia de llegada | Global | C | an..13 / an2 | n(3,8) / 'DD' | /DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cac:LocationCoordinate/cbc:LatitudeDegreesMeasure[@unitCode] / cbc:LongitudeDegreesMeasure[@unitCode] | Formato numérico hasta 3 enteros 8 decimales <br> @unitCode 'DD' sin validación explícita | ERROR | 3413 |
| 71 | Código del Puerto / Tipo de locación (Puerto) | Global | C | an3 / an1 | Catálogo 63 / '1' (@scheme* ) | /DespatchAdvice/cac:Shipment/cac:FirstArrivalPortLocation/cbc:ID[@schemeAgencyName/@schemeName/@schemeURI] / cbc:LocationTypeCode | Obligatorio si motivo 08/09/19 y tipo locación '1' (con mayor exigencia para 19) <br> Si hay código puerto debe haber tipo locación <br> Tipo locación debe ser '1' o '2' <br> Código debe existir en listado | ERROR/OBSERV | 4413<br>3484<br>4415<br>4416<br>3459<br>4256/4255/4257 |
| 72 | Código del Aeropuerto / Tipo de locación (Aeropuerto) | Global | C | an3 / an1 | Catálogo 64 / '2' | /DespatchAdvice/cac:Shipment/cac:FirstArrivalPortLocation/cbc:ID / cbc:LocationTypeCode | Obligatorio si motivo 08/19 y tipo '2' (19 más exigente) <br> Si hay código aeropuerto debe haber tipo locación <br> Código debe existir en listado | ERROR/OBSERV | 4413<br>3484<br>4415<br>4416<br>3460<br>4256/4255/4257 |
| 73 | Nombre del puerto, aeropuerto | Global | C | an..200 | Alfanumérico | /DespatchAdvice/cac:Shipment/cac:FirstArrivalPortLocation/cbc:Name | Obligatorio si hay código puerto/aeropuerto | OBSERV | 4418 |
### Bienes a trasladar - Incremental

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 74 | Número de orden del item | Ítem | C | n..4 | Numérico hasta 4 | /DespatchAdvice/cac:DespatchLine/cbc:ID / cac:OrderLineReference/cbc:LineID | Formato inválido <br> Duplicado | ERROR | 2023<br>2752 |
| 75 | Cantidad del bien | Ítem | C | an..23 | n(12,10) (positivo) o entero positivo según doc relacionado | /DespatchAdvice/cac:DespatchLine/cbc:DeliveredQuantity | Obligatorio si NO indicador traslado total DAM/DS y NO indicador contenedor Manifiesto <br> Formato decimal positivo 12,10 si doc relacionado distinto a '91' <br> Formato entero positivo hasta 10 dígitos si doc relacionado '91' y sin indicador contenedor <br> Si indicador traslado total DAM/DS debe coincidir con cantidad/saldo según DAM/DS | ERROR/OBSERV/REST | 2580<br>2780<br>4436 |
| 76 | Unidad de medida del bien | Ítem | C | an..3 | Catálogo 03 / Catálogo 65 (@unitCodeListID/@unitCodeListAgencyName) | /DespatchAdvice/cac:DespatchLine/cbc:DeliveredQuantity@unitCode | Obligatorio si NO indicador traslado total y NO indicador contenedor <br> Reglas por motivo y doc relacionado (DAM/DS requieren catálogo 65, Manifiesto 91 requiere 'U') <br> Atributos list* incorrectos | ERROR/OBSERV | 2883<br>4320<br>3446<br>4258<br>4259 |
| 77 | Descripción detallada del bien | Ítem | C | an..500 | Alfanumérico 3-500 (permite whitespace excepto solo espacios) | /DespatchAdvice/cac:DespatchLine/cac:Item/cbc:Description | Obligatorio si NO indicador traslado total DAM/DS <br> Formato inválido | ERROR/OBSERV | 2781<br>4084 |
| 78 | Código del bien | Ítem | C | an..30 | Alfanumérico hasta 30, sin whitespace distinto a espacio | /DespatchAdvice/cac:DespatchLine/cac:Item/cac:SellersItemIdentification/cbc:ID | Formato inválido si existe | OBSERV | 4085 |
| 79 | Código producto SUNAT | Ítem | C | n..8 | Catálogo 25 (@listID='UNSPSC', @listAgencyName='GS1 US', @listName) | /DespatchAdvice/cac:DespatchLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode | Obligatorio si NO indicador traslado total y indicador bien normalizado '1' <br> Numérico hasta 8, no solo ceros <br> Debe existir en catálogo 25 <br> Debe existir en listado bienes normalizados (catálogo 62) si aplica <br> Atributos incorrectos | ERROR/OBSERV | 3372<br>3002<br>3373<br>3425<br>4254/4251/4252 |
| 80 | Código GTIN | Ítem | C | an..14 | Alfanumérico hasta 14 / @schemeID GTIN-8/12/13/14 | /DespatchAdvice/cac:DespatchLine/cac:Item/cac:StandardItemIdentification/cbc:ID[@schemeID] | Formato inválido <br> schemeID inválido | ERROR/OBSERV | 3375<br>4335 |
| 81-90 | Propiedades adicionales (AdditionalItemProperty) - Catálogo 55 | Ítem | C | Según concepto | Ver reglas | /DespatchAdvice/cac:DespatchLine/cac:Item/cac:AdditionalItemProperty/cbc:Name / cbc:NameCode / cbc:Value | Conceptos relevantes: 7020 Partida arancelaria, 7021 NUMERACIÓN DAM/DS, 7022 Indicador bien normalizado, 7023 N° serie DAM/DS, 7024 Doc transporte MC, 7025 N° detalle MC, 7026 Contenedor MC, 7027 Precinto, 7028 Indicador contenedor vacío. <br> Reglas obligatorias según motivo/doc relacionado/indicadores <br> Formatos específicos <br> Deben existir/conciliar con documentos relacionados, contenedores, Manifiesto de carga <br> No duplicados según contexto | ERROR/OBSERV/REST | 4235<br>3426/3427/3428/3480/3490/3491/3481/3482<br>3064<br>3377/3378/3429<br>3396/3379<br>2769/3430<br>3431/3432<br>3466/3467<br>3468/3469/3472/3492<br>3470/3486/3471<br>3474/3475<br>3476/3477<br>4252/4251/4253 |

### Firma digital

| N° | Dato | Nivel | Cond. | Tipo y longitud | Formato | Tag UBL | Validación | Tipo retorno | Códigos |
|---|---|---|---|---|---|---|---|---|---|
| 84 | Firma digital | Global | M | an..3000 | XMLDSig (ds:Signature) en ext:UBLExtensions/UBLExtension/ExtensionContent | (extensiones) | No existe firma digital <br> Firma digital no cumple requisitos de vigencia / ha sido alterado | ERROR | 1059<br>2336 |

## 5. Reglas de cardinalidad derivadas

| Concepto | Regla | Referencia |
|---|---|---|
| Obligatoriedad general | M = Obligatorio, C = Condicional según reglas | Fuente Gu_a_Remitente2_0.txt |
| GRE Remitente | DespatchAdviceTypeCode = '09', serie con prefijo 'T' | Ítem 6, R.S. 123-2022 |
| Modalidad | '01' Público, '02' Privado (Catálogo 18) | Ítem 32 |
| Motivos | Catálogo 20: 01,02,03,04,05,06,07,08,09,13,14,17,18,19 | Ítem 22 |
| Proveedor | Solo si motivo '02','07','13'. Reglas de tipo/documento/razón social. No igual a remitente si motivo '02','07'. | Ítems 17-18 |
| Comprador | Solo si motivo '03','13'. Obligatorio para motivo '03'. No igual a remitente/destinatario si motivo '03'. | Ítems 19-20 |
| Destinatario | Obligatorio. Para motivo '06','17','19' debe ser RUC (schemeID '6'). Restricciones de igualdad según motivo. | Ítems 15-16 |
| Transportista | Solo modalidad '01' sin indicador traslado M1/L. No debe existir en modalidad '02'. Debe ser distinto a remitente. | Ítems 43-46 |
| Vehículos/conductores | Reglas cruzadas entre modalidad, indicadores M1/L y registro vehículos/conductores. Máximo 2 vehículos secundarios, máximo 2 conductores secundarios. | Ítems 47-60 |
| Documentos relacionados | Incremental. Reglas estrictas por motivo/código (DAM/DS para 08/09/19, Manifiesto 91 para 19, Constancias 49/80 con bienes normalizados, GRE 09 para baja por cambio destinatario, etc.) | Ítems 11-14 |
| Contenedores/precintos/Manifiesto | Si motivo 08/09/19 puede requerir contenedores/bultos. Si hay Manifiesto 91 con indicador contenedor, detalle debe tener conceptos MC y conciliar contenedores/precintos/documento transporte/número detalle. | Ítems 27-31, 75-76, 81-90 |
| Bienes normalizados | Si indicador bien normalizado '1' requiere partida arancelaria (7020) y código producto SUNAT (79); con ciertos documentos relacionados requiere constancia 49/80. | Ítems 79, 81-90, 504 |
| Peso | GrossWeightMeasure obligatorio (KGM/TNE). NetWeightMeasure/sustento obligatorios si motivo 08/09/19 y no traslado total DAM/DS. | Ítems 24-26 |
| Firma | Obligatoria. Debe validarse vigencia/correspondencia. | Ítem 84 |

## 6. Ejemplo XML mínimo válido (GRE Remitente)

`xml
<?xml version="1.0" encoding="ISO-8859-1"?>
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
  <cbc:ID>T001-00000001</cbc:ID>
  <cbc:IssueDate>2025-01-01</cbc:IssueDate>
  <cbc:IssueTime>10:00:00</cbc:IssueTime>
  <cbc:DespatchAdviceTypeCode listAgencyName="PE:SUNAT" listName="Tipo de Documento" listURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo01">09</cbc:DespatchAdviceTypeCode>
  <cac:DespatchSupplierParty>
    <cac:Party>
      <cac:PartyIdentification>
        <cbc:ID schemeID="6" schemeName="Documento de Identidad" schemeAgencyName="PE:SUNAT" schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06">20100000000</cbc:ID>
      </cac:PartyIdentification>
      <cac:PartyLegalEntity>
        <cbc:RegistrationName>RAZON SOCIAL DEL REMITENTE</cbc:RegistrationName>
      </cac:PartyLegalEntity>
    </cac:Party>
  </cac:DespatchSupplierParty>
  <cac:DeliveryCustomerParty>
    <cac:Party>
      <cac:PartyIdentification>
        <cbc:ID schemeID="6" schemeName="Documento de Identidad" schemeAgencyName="PE:SUNAT" schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06">20111111111</cbc:ID>
      </cac:PartyIdentification>
      <cac:PartyLegalEntity>
        <cbc:RegistrationName>RAZON SOCIAL DEL DESTINATARIO</cbc:RegistrationName>
      </cac:PartyLegalEntity>
    </cac:Party>
  </cac:DeliveryCustomerParty>
  <cac:Shipment>
    <cbc:ID>SUNAT_Envio</cbc:ID>
    <cbc:HandlingCode listAgencyName="PE:SUNAT" listName="Motivo de traslado" listURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo20">01</cbc:HandlingCode>
    <cbc:GrossWeightMeasure unitCode="KGM">100.000</cbc:GrossWeightMeasure>
    <cac:ShipmentStage>
      <cbc:TransportModeCode listName="Modalidad de traslado" listAgencyName="PE:SUNAT" listURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo18">02</cbc:TransportModeCode>
      <cac:TransitPeriod>
        <cbc:StartDate>2025-01-01</cbc:StartDate>
      </cac:TransitPeriod>
    </cac:ShipmentStage>
    <cac:Delivery>
      <cac:Despatch>
        <cac:DespatchAddress>
          <cbc:ID schemeAgencyName="PE:INEI" schemeName="Ubigeos">150101</cbc:ID>
          <cac:AddressLine>
            <cbc:Line>AV. EJEMPLO 123, LIMA</cbc:Line>
          </cac:AddressLine>
        </cac:DespatchAddress>
      </cac:Despatch>
      <cac:DeliveryAddress>
        <cbc:ID schemeAgencyName="PE:INEI" schemeName="Ubigeos">150104</cbc:ID>
        <cac:AddressLine>
          <cbc:Line>AV. EJEMPLO 456, LIMA</cbc:Line>
        </cac:AddressLine>
      </cac:DeliveryAddress>
    </cac:Delivery>
  </cac:Shipment>
  <cac:DespatchLine>
    <cbc:ID>1</cbc:ID>
    <cbc:DeliveredQuantity unitCode="NIU">1.0000000000</cbc:DeliveredQuantity>
    <cac:OrderLineReference>
      <cbc:LineID>1</cbc:LineID>
    </cac:OrderLineReference>
    <cac:Item>
      <cbc:Description>PRODUCTO DE EJEMPLO</cbc:Description>
    </cac:Item>
  </cac:DespatchLine>
</DespatchAdvice>
`

*Ejemplo mínimo orientativo. Debe ajustarse a las reglas según motivo/modalidad/documentación. Firma como placeholder (<!-- ds:Signature -->). No es un documento firmado real.*

## 7. Reglas transversales

| Regla | Código(s) | Detalle (según General.txt / fuente) |
|---|---|---|
| Nombre archivo XML | [NO VERIFICADO formato completo] | General.txt indica estructura tipo RRRRRRRRRRR-01-SSSS-NNNNNNNN.xml en descripción general; para GRE el código de tipo es  9. R.S. 123-2022 detalla: RRRRRRRRRRR-09-SSSS-NNNNNNNN.XML donde RUC (11), guion, TT='09', guion, serie TAAA/VAAA, guion, correlativo (1-8 dígitos), extensión XML/ZIP. Debe coincidir con contenido. |
| ZIP con un solo comprobante | 0158 | El archivo ZIP contiene demasiados comprobantes para este tipo de envío |
| ZIP vacío/corrupto/sin archivos | 0155/0156/0157 | Errores en empaque |
| Nombre XML no coincide con ZIP | 0161 | El nombre del archivo XML no coincide con el nombre del archivo ZIP |
| Parseo/XSD | 0306 | No se puede leer (parsear) el archivo XML |
| Correspondencia nombre archivo ↔ contenido (serie/correlativo) | 1035/1036 | Serie/número del nombre no coinciden con cbc:ID |
| Correspondencia nombre archivo ↔ contenido (comprobante registrado) | 1032/1033 | Ya informado/anulado/rechazado o registrado con otros datos |
| Correspondencia RUC archivo ↔ RUC emisor | 1034/0154 | RUC en nombre/XML vs RUC emisor; si invoca otro RUC debe existir relación PSE vigente (hasta 7mo día calendario mes siguiente a revocación) |
| Autorización usuario/PSE | 0154 | El RUC del archivo no corresponde al RUC del usuario o el proveedor no esta autorizado a enviar comprobantes del contribuyente |
| Certificado comunicado a SUNAT | 2325 | El certificado usado no es el comunicado a SUNAT |
| Firma no coincide / alterado | 2335/2336/1059 | Firma no coincide con el comprobante / no cumple requisitos / no existe firma digital |
| Vigencia relación PSE | [NO VERIFICADO] | General.txt indica vigencia hasta 7mo día calendario del mes siguiente a fecha de revocación en campo fec_fin del listado |

## Referencias

- Archivos fuente usados:
  - C:\Users\angel\AppData\Local\Temp\opencode\sunat\xlsx\Gu_a_Remitente2_0.txt (524 líneas, header en línea 2) - Validaciones GRE Remitente (hoja "Guía-Remitente2_0")
  - C:\Users\angel\AppData\Local\Temp\opencode\sunat\xlsx\General.txt - Validaciones generales de envío
  - C:\Users\angel\AppData\Local\Temp\opencode\sunat\R-123-2022-SUNAT.txt - R.S. 123-2022-SUNAT (ANEXO VII, Anexo N°14)
  - C:\Users\angel\AppData\Local\Temp\opencode\sunat\xlsx\Cat_logos.txt - Catálogos oficiales (01,03,06,13,18,20,25,55,61,62,63,64,65,D-37)
- Fecha de consulta: 2026-10-21
- Sitio oficial SUNAT: https://www.sunat.gob.pe/
