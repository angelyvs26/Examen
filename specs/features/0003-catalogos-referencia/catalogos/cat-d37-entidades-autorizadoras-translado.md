# Catálogo D-37 — Entidades que emiten autorizaciones especiales para el traslado

| Metadato | Valor |
|---|---|
| Código de catálogo | D-37 |
| Nombre oficial | Entidades que emiten autorizaciones especiales para el traslado |
| Columnas | `Código`, `Nombre`, `Abreviatura` |
| Número de filas | 12 |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [515] a [526] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-d37-entidades-autorizadoras-translado.csv` |

## Uso en la GRE

Entidades que emiten autorizaciones especiales para el traslado de bienes (SUCAMEC, DIGEMID, DIGESA, SENASA, SERFOR, MTC, PRODUCE, MIN. AMBIENTE, SANIPES, MML, MINSA, GR). Su código se consigna como `@schemeID` del número de autorización especial.

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty/cac:AgentParty/cac:PartyLegalEntity/cbc:CompanyID@schemeID` — Código de entidad autorizadora (remitente).
- `/DespatchAdvice/cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:AgentParty/cac:PartyLegalEntity/cbc:CompanyID@schemeID` — Código de entidad autorizadora (transportista).
- `/DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:ShipmentDocumentReference/cbc:ID@schemeID` — Código de entidad emisora (vehículo principal).
- `/DespatchAdvice/cac:Shipment/cac:TransportHandlingUnit/cac:TransportEquipment/cac:AttachedTransportEquipment/cac:ShipmentDocumentReference/cbc:ID@schemeID` — Código de entidad emisora (vehículo secundario).

## Notas

- En todos los casos el atributo lleva `@schemeName="Entidad Autorizadora"` y `@schemeAgencyName="PE:SUNAT"`.
- La R.S. 123-2022-SUNAT indica que este catálogo será publicado en el Portal de la SUNAT; en la fuente aparece bajo el bloque «Otros Listados».

## Tabla

| Código | Nombre | Abreviatura |
|---|---|---|
| 01 | Superintendencia Nacional de Control de Servicios de Seguridad, Armas, Municiones y Explosivos de Uso Civil | SUCAMEC |
| 02 | Dirección General de Medicamentos Insumos y Drogas | DIGEMID |
| 03 | Dirección General de Salud Ambiental | DIGESA |
| 04 | Servicio Nacional de Sanidad Agraria | SENASA |
| 05 | Servicio Nacional Forestal y de Fauna Silvestre | SERFOR |
| 06 | Ministerio de Transportes y Comunicaciones | MTC |
| 07 | Ministerio de la Producción | PRODUCE |
| 08 | Ministerio del Ambiente | MIN. AMBIENTE |
| 09 | Organismo Nacional de Sanidad Pesquera | SANIPES |
| 10 | Municipalidad Metropolitana de Lima | MML |
| 11 | Ministerio de Salud | MINSA |
| 12 | Gobierno Regional | GR |

## CSV

`../datos/cat-d37-entidades-autorizadoras-translado.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

