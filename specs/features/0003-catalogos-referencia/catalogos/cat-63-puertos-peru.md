# Catálogo 63 — Puertos del Perú

| Metadato | Valor |
|---|---|
| Código de catálogo | 63 |
| Nombre oficial | Puertos del Perú |
| Columnas | `Código`, `Nombre`, `Ubigeo` |
| Número de filas | 21 |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [349] a [369] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-63-puertos-peru.csv` |

## Uso en la GRE

Códigos de los puertos del Perú, con su ubigeo. Se usa cuando el traslado tiene como origen o destino un puerto (por ejemplo, motivo "08" Importación o "19" Traslado de mercancía extranjera).

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:Shipment/cac:FirstArrivalPortLocation/cbc:ID` — Código del puerto. Atributos: `@schemeAgencyName="PE:SUNAT"`, `@schemeName="Puertos"`, `@schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo63"`.
- `/DespatchAdvice/cac:Shipment/cac:FirstArrivalPortLocation/cbc:LocationTypeCode` — Tipo de locación: "1" Puerto (catálogo 63) o "2" Aeropuerto (catálogo 64).

## Tabla

| Código | Nombre | Ubigeo |
|---|---|---|
| PUB | Bayóvar | 200801 |
| CLL | Callao | 070101 |
| CON | Conchán | 150119 |
| CHY | Chancay | 150605 |
| CHM | Chimbote | 021801 |
| EEN | Eten | 140113 |
| HCO | Huacho | 150801 |
| HUY | Huarmey | 021101 |
| ILQ | Ilo | 180301 |
| IQT | Iquitos | 160101 |
| MRI | Matarani | 040701 |
| PAI | Paita | 200501 |
| PIO | Pisco | 110505 |
| PCL | Pucallpa | 250101 |
| PUN | Puno | 210101 |
| SVY | Salaverry | 130109 |
| SNX | San Nicolas | 110304 |
| SUP | Supe | 150204 |
| TYL | Talara | 200701 |
| YMS | Yurimaguas | 160201 |
| ZOR | Zorritos | 240103 |

## CSV

`../datos/cat-63-puertos-peru.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

