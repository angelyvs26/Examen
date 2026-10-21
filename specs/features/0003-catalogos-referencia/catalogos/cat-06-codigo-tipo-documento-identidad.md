# Catálogo 06 — Código de tipo de documento de identidad

| Metadato | Valor |
|---|---|
| Código de catálogo | 06 |
| Nombre oficial | Código de tipo de documento de identidad |
| Columnas | `Código`, `Descripción` |
| Número de filas | 12 |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [59] a [70] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-06-codigo-tipo-documento-identidad.csv` |

## Uso en la GRE

Tipo de documento de identidad de los actores de la GRE (remitente, destinatario, proveedor, comprador, transportista y conductores) y del emisor del documento relacionado. Se transporta en el atributo `@schemeID` del identificador de la parte.

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:DespatchSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID@schemeID` — Tipo de documento de identidad del remitente ("6" para el RUC).
- `/DespatchAdvice/cac:DeliveryCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID@schemeID` — Tipo de documento de identidad del destinatario.
- `/DespatchAdvice/cac:SellerSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID@schemeID` — Tipo de documento de identidad del proveedor.
- `/DespatchAdvice/cac:BuyerCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID@schemeID` — Tipo de documento de identidad del comprador.
- `/DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:PartyIdentification/cbc:ID@schemeID` — Tipo de documento de identidad del transportista ("6" para el RUC).
- `/DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:DriverPerson/cbc:ID@schemeID` — Tipo de documento de identidad del conductor (principal y secundarios).
- `/DespatchAdvice/cac:AdditionalDocumentReference/cac:IssuerParty/cac:PartyIdentification/cbc:ID@schemeID` — Tipo de documento de identidad del emisor del documento relacionado.

## Notas

- En todos los casos el valor va acompañado de `@schemeName="Documento de Identidad"`, `@schemeAgencyName="PE:SUNAT"` y `@schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo06"`.

## Tabla

| Código | Descripción |
|---|---|
| 0 | DOC.TRIB.NO.DOM.SIN.RUC |
| 1 | Documento Nacional de Identidad |
| 4 | Carnet de extranjería |
| 6 | Registro Unico de Contributentes |
| 7 | Pasaporte |
| A | Cédula Diplomática de identidad |
| B | DOC.IDENT.PAIS.RESIDENCIA-NO.D |
| C | Tax Identification Number - TIN – Doc Trib PP.NN |
| D | Identification Number - IN – Doc Trib PP. JJ |
| E | TAM- Tarjeta Andina de Migración  |
| F | Permiso Temporal de Permanencia - PTP |
| G | Salvoconducto |

## CSV

`../datos/cat-06-codigo-tipo-documento-identidad.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

