# Catálogo 25 — Código de producto SUNAT

| Metadato | Valor |
|---|---|
| Código de catálogo | 25 |
| Nombre oficial | Código de producto SUNAT |
| Columnas | `Código`, `Descripción` |
| Número de filas | 0 (el catálogo remite a una lista externa) |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [108] a [109] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-25-codigo-producto-sunat.csv` |

## Uso en la GRE

Código de producto SUNAT (UNSPSC) de cada ítem de la GRE. La tabla de códigos no está incluida en la fuente: el catálogo remite al sistema UNSPSC v14_0801 (nivel 3).

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:DespatchLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode` — Código producto SUNAT del ítem. Atributos: `@listID="UNSPSC"`, `@listAgencyName="GS1 US"`, `@listName="Item Classification"`.

## Tabla

| Código | Descripción |
|---|---|

## Filas de referencia de la fuente (sin código de catálogo)

- [108] Sistema de Codificación Común de las Naciones Unidas - UNSPSC v14_0801 (nivel 3)
- [109] https://www.unspsc.org/codeset-downloads/productid/28/createdbyuser/3?txtsearch=

## CSV

`../datos/cat-25-codigo-producto-sunat.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

