# Catálogo 03 — Código de tipo de unidad de medida comercial

| Metadato | Valor |
|---|---|
| Código de catálogo | 03 |
| Nombre oficial | Código de tipo de unidad de medida comercial |
| Columnas | `Código`, `Descripción` |
| Número de filas | 0 (el catálogo remite a una lista externa) |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [52] a [53] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-03-codigo-tipo-unidad-medida-comercial.csv` |

## Uso en la GRE

Unidad de medida de los pesos de la carga y de la cantidad de los ítems. La tabla de códigos no está incluida en la fuente: el catálogo remite al estándar UN/ECE Recommendation 20 Revision 13. Para los traslados con DAM o DS existe el catálogo 65 de esta misma fuente.

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:Shipment/cbc:NetWeightMeasure@unitCode` — Unidad de medida del peso bruto total de los ítems seleccionados ("KGM" Kilogramos).
- `/DespatchAdvice/cac:Shipment/cbc:GrossWeightMeasure@unitCode` — Unidad de medida del peso bruto total ("KGM" Kilogramos o "TNE" Toneladas).
- `/DespatchAdvice/cac:DespatchLine/cbc:DeliveredQuantity@unitCode` — Unidad de medida de la cantidad del ítem (junto con el catálogo 65). Atributo `@unitCodeListID="UN/ECE rec 20"`.

## Tabla

| Código | Descripción |
|---|---|

## Filas de referencia de la fuente (sin código de catálogo)

- [52] UN/ECE Recommendation 20 Revision 13
- [53] https://www.unece.org/fileadmin/DAM/uncefact/recommendations/rec20/rec20_Rev13e_2017.xls

## CSV

`../datos/cat-03-codigo-tipo-unidad-medida-comercial.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

