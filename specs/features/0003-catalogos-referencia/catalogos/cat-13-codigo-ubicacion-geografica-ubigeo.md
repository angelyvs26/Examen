# Catálogo 13 — Código de ubicación geográfica (UBIGEO)

| Metadato | Valor |
|---|---|
| Código de catálogo | 13 |
| Nombre oficial | Código de ubicación geográfica (UBIGEO) |
| Columnas | `Código`, `Descripción` |
| Número de filas | 0 (el catálogo remite a una lista externa) |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [76] a [77] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-13-codigo-ubicacion-geografica-ubigeo.csv` |

## Uso en la GRE

Ubigeo de 6 dígitos del punto de partida y del punto de llegada de la GRE. La fuente no incluye la tabla de ubigeos: el catálogo remite al catálogo de ubigeos del INEI.

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchAddress/cbc:ID` — Ubigeo del punto de partida. Atributos: `@schemeAgencyName="PE:INEI"`, `@schemeName="Ubigeos"`.
- `/DespatchAdvice/cac:Shipment/cac:Delivery/cac:DeliveryAddress/cbc:ID` — Ubigeo del punto de llegada. Atributos: `@schemeAgencyName="PE:INEI"`, `@schemeName="Ubigeos"`.

## Tabla

| Código | Descripción |
|---|---|

## Filas de referencia de la fuente (sin código de catálogo)

- [76] Catálogo de ubigeos del INEI
- [77] https://www.datosabiertos.gob.pe/dataset/c%C3%B3digo-de-ubicaci%C3%B3n-geogr%C3%A1fica-en-el-per%C3%BA-instituto-nacional-de-estad%C3%ADstica-e-inform%C3%A1tica

## CSV

`../datos/cat-13-codigo-ubicacion-geografica-ubigeo.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

