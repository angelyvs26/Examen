# Catálogo 18 — Código de modalidad de transporte

| Metadato | Valor |
|---|---|
| Código de catálogo | 18 |
| Nombre oficial | Código de modalidad de transporte |
| Columnas | `Código`, `Descripción` |
| Número de filas | 2 |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [83] a [84] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-18-modalidad-transporte.csv` |

## Uso en la GRE

Modalidad del traslado: transporte público o transporte privado. Determina si se exigen o no los datos del transportista, del vehículo y del conductor.

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:Shipment/cac:ShipmentStage/cbc:TransportModeCode` — Modalidad de traslado. Atributos: `@listName="Modalidad de traslado"`, `@listAgencyName="PE:SUNAT"`, `@listURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo18"`.

## Tabla

| Código | Descripción |
|---|---|
| 01 | Transporte público |
| 02 | Transporte privado |

## CSV

`../datos/cat-18-modalidad-transporte.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

