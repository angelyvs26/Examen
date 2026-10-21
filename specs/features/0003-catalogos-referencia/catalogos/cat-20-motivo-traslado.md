# Catálogo 20 — Código de motivo de traslado

| Metadato | Valor |
|---|---|
| Código de catálogo | 20 |
| Nombre oficial | Código de motivo de traslado |
| Columnas | `Código`, `Descripción` |
| Número de filas | 14 |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [90] a [103] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-20-motivo-traslado.csv` |

## Uso en la GRE

Motivo del traslado de la mercancía. Determina los datos exigidos (por ejemplo, comprador para el motivo "03", proveedor para los motivos "02" y "07", DAM/DS para los motivos "08" y "09", puerto o aeropuerto para el motivo "19") y si el motivo "13" (Otros) exige describir el motivo en el texto de observaciones.

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:Shipment/cbc:HandlingCode` — Motivo del traslado. Atributos: `@listName="Motivo de traslado"`, `@listAgencyName="PE:SUNAT"`, `@listURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogoD05"`.
- `/DespatchAdvice/cac:Shipment/cbc:HandlingInstructions` — Descripción del motivo de traslado (texto libre de hasta 100 caracteres; se usa con el motivo "13" Otros y es la base del listado de palabras clave para la detección del motivo).

## Notas

- El código 19 (Traslado de mercancía extranjera) se incorporó con vigencia 2024-11-14 (hoja «Control de Cambios» del XLSX oficial).
- La R.S. 123-2022-SUNAT declara para este campo el `@listURI` `...catalogos:catalogoD05`, identificador que la propia norma asigna al catálogo N.° 20 de motivo de traslado.

## Tabla

| Código | Descripción |
|---|---|
| 01 | Venta |
| 02 | Compra |
| 03 | Venta con entrega a terceros |
| 04 | Traslado entre establecimientos de la misma empresa |
| 05 | Consignación |
| 06 | Devolución |
| 07 | Recojo de bienes transformados |
| 08 | Importación |
| 09 | Exportación |
| 13 | Otros |
| 14 | Venta sujeta a confirmación del comprador    |
| 17 | Traslado de bienes para transformación |
| 18 | Traslado emisor itinerante CP |
| 19 | Traslado de mercancía extranjera |

## CSV

`../datos/cat-20-motivo-traslado.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

