# Catálogos SUNAT para la GRE (Guía de Remisión Electrónica)

Transcripción de los catálogos oficiales del **Anexo N.° 8 : Catálogo de códigos** que la SUNAT exige para emitir la GRE, tal como figuran en la hoja «Catálogos» del XLSX oficial de validaciones `ValidacionesGREv20250421_0.xlsx`.

Cada catálogo tiene su ficha Markdown (`cat-<NN>-<slug>.md`) y su tabla en CSV (`../datos/cat-<NN>-<slug>.csv`) para carga como dato de referencia de la aplicación.

## Fuente de los datos

| Elemento | Detalle |
|---|---|
| Archivo de trabajo | `Cat_logos.txt` (498 líneas, formato `texto plano`, una fila por línea con prefijo `[n]`) |
| Origen | Hoja «Catálogos» del XLSX oficial de validaciones de la GRE de SUNAT (`ValidacionesGREv20250421_0.xlsx`) |
| Contexto normativo | R.S. 123-2022-SUNAT, Anexo N.° 14 (Estándar UBL 2.1 Guía de remisión remitente / transportista) |
| Estructura del archivo | Un bloque por catálogo: `No. \| <número>`, `Catálogo \| <nombre oficial>`, `<cabecera de columnas>` y las filas de datos |
| Regla de transcripción | Códigos y descripciones se reproducen literalmente (sin parafrasear, sin corregir ortografía, sin truncar); las filas que no son datos de catálogo (encabezados internos, rótulos de sección y referencias a listas externas) se conservaron en la sección «Filas de referencia de la fuente» de la ficha correspondiente |

## Índice de catálogos

### Catálogos consumidos por el XML de la GRE

| No. | Nombre oficial | Filas | Para qué se usa en la GRE | Ficha | Datos |
|---|---|---|---|---|---|
| 01 | Código de tipo de documento | 40 | Tipo de documento electrónico: solo "09" (GRE-Remitente) y "31" (GRE-Transportista). También las posiciones 13-14 (`TT`) del nombre del archivo XML/ZIP. | [cat-01-codigo-tipo-documento.md](cat-01-codigo-tipo-documento.md) | [CSV](../datos/cat-01-codigo-tipo-documento.csv) |
| 06 | Código de tipo de documento de identidad | 12 | `@schemeID` del identificador del remitente, destinatario, proveedor, comprador, transportista, conductor y del emisor del documento relacionado. | [cat-06-codigo-tipo-documento-identidad.md](cat-06-codigo-tipo-documento-identidad.md) | [CSV](../datos/cat-06-codigo-tipo-documento-identidad.csv) |
| 13 | Código de ubicación geográfica (UBIGEO) | 0 | Ubigeo del punto de partida y del punto de llegada. La tabla de ubigeos no está en la fuente (la publica el INEI). | [cat-13-codigo-ubicacion-geografica-ubigeo.md](cat-13-codigo-ubicacion-geografica-ubigeo.md) | [CSV](../datos/cat-13-codigo-ubicacion-geografica-ubigeo.csv) |
| 18 | Código de modalidad de transporte | 2 | `cbc:TransportModeCode`: transporte público (01) o privado (02). | [cat-18-modalidad-transporte.md](cat-18-modalidad-transporte.md) | [CSV](../datos/cat-18-modalidad-transporte.csv) |
| 20 | Código de motivo de traslado | 14 | `cbc:HandlingCode`: motivo del traslado (01 a 19). Condiciona los datos exigidos según el motivo. | [cat-20-motivo-traslado.md](cat-20-motivo-traslado.md) | [CSV](../datos/cat-20-motivo-traslado.csv) |
| 25 | Código de producto SUNAT | 0 | `cbc:ItemClassificationCode` del ítem (UNSPSC). La tabla no está en la fuente (remite a UNSPSC v14_0801). | [cat-25-codigo-producto-sunat.md](cat-25-codigo-producto-sunat.md) | [CSV](../datos/cat-25-codigo-producto-sunat.csv) |
| 55 | Código de identificación del concepto tributario | 120 | `cbc:NameCode` del «Propiedad del item»: partida arancelaria (7020), DAM (7021), indicador de bien normalizado (7022), serie DAM/DS (7023) y datos del Manifiesto de Carga (7024 a 7028). | [cat-55-codigo-identificacion-concepto-tributario.md](cat-55-codigo-identificacion-concepto-tributario.md) | [CSV](../datos/cat-55-codigo-identificacion-concepto-tributario.csv) |
| 61 | Documentos relacionados aplicables a las GRE (versión 2.0) | 27 | `cbc:DocumentTypeCode` del documento que sustenta el traslado (factura, DAM, DS, MC, autorizaciones varias). La columna «GRE Aplicable» indica si aplica a la GRE-Remitente, a la GRE-Transportista o a ambas. | [cat-61-documentos-relacionados-gre.md](cat-61-documentos-relacionados-gre.md) | [CSV](../datos/cat-61-documentos-relacionados-gre.csv) |
| 62 | Bienes normalizados | 52 | Partida arancelaria, descripción y código de producto SUNAT de los bienes normalizados por SUNAT (control del ítem con indicador de bien normalizado). | [cat-62-bienes-normalizados.md](cat-62-bienes-normalizados.md) | [CSV](../datos/cat-62-bienes-normalizados.csv) |
| 63 | Puertos del Perú | 21 | `cbc:FirstArrivalPortLocation/cbc:ID` con `cbc:LocationTypeCode` = "1" (Puerto) y su ubigeo. | [cat-63-puertos-peru.md](cat-63-puertos-peru.md) | [CSV](../datos/cat-63-puertos-peru.csv) |
| 64 | Aeropuertos del Perú | 31 | `cbc:FirstArrivalPortLocation/cbc:ID` con `cbc:LocationTypeCode` = "2" (Aeropuerto) y su ubigeo. | [cat-64-aeropuertos-peru.md](cat-64-aeropuertos-peru.md) | [CSV](../datos/cat-64-aeropuertos-peru.csv) |
| 65 | Código de unidades de medida (para uso solo para la GRE en DAM o DS) | 97 | `cbc:DeliveredQuantity@unitCode` de la cantidad del ítem en los traslados con DAM o DS. | [cat-65-unidades-medida-gre-dam-ds.md](cat-65-unidades-medida-gre-dam-ds.md) | [CSV](../datos/cat-65-unidades-medida-gre-dam-ds.csv) |
| D-37 | Entidades que emiten autorizaciones especiales para el traslado | 12 | `@schemeID` del número de autorización especial del remitente, del transportista y de los vehículos (SUCAMEC, DIGEMID, DIGESA, SENASA, SERFOR, MTC, PRODUCE, MIN. AMBIENTE, SANIPES, MML, MINSA, GR). | [cat-d37-entidades-autorizadoras-translado.md](cat-d37-entidades-autorizadoras-translado.md) | [CSV](../datos/cat-d37-entidades-autorizadoras-translado.csv) |

### Catálogos presentes en la fuente que no son catálogos del XML de la GRE

| No. | Nombre oficial | Filas | Situación | Ficha | Datos |
|---|---|---|---|---|---|
| 03 | Código de tipo de unidad de medida comercial | 0 | La lista vive en el estándar UN/ECE Recommendation 20 Rev. 13. En la GRE se usa para los pesos (`NetWeightMeasure@unitCode`, `GrossWeightMeasure@unitCode`) y para la cantidad del ítem. | [cat-03-codigo-tipo-unidad-medida-comercial.md](cat-03-codigo-tipo-unidad-medida-comercial.md) | [CSV](../datos/cat-03-codigo-tipo-unidad-medida-comercial.csv) |
| 52 | Códigos de leyendas | 15 | Es catálogo del Anexo N.° 8, pero de los comprobantes de pago electrónico (`/Invoice/cbc:Note`), no de la guía de remisión. La leyenda "2012 Observaciones relacionadas con el traslado de mercaderías" sí es de interés para la GRE. | [cat-52-codigos-leyendas.md](cat-52-codigos-leyendas.md) | [CSV](../datos/cat-52-codigos-leyendas.csv) |

## Catálogos solicitados que no existen en la fuente

No se han creado fichas para los catálogos siguientes porque **no aparecen en `Cat_logos.txt`**. En cada caso se indica dónde reside realmente el dato:

| Catálogo solicitado | Estado en la fuente | Dónde está realmente el dato |
|---|---|---|
| 09 — Ámbito de la SUNAT para el registro de la GRE | No existe como catálogo. | El código "09" sí aparece, pero como **tipo de documento relacionado** del Catálogo 61 ("Guía de Remisión Remitente"). Como tipo de documento de la GRE, el código es el "09" del Catálogo 01. |
| 21 — Número de DAM (Declaración Aduanera de Mercancías) | No existe como catálogo. | El DAM viaja como **documento relacionado** de código "50" (Catálogo 61) y su numeración como conceptos "7021" (número de declaración aduanera) y "7023" (número de serie en la DAM o DS) del Catálogo 55. |
| 22 — Número de Manifiesto | No existe como catálogo. | El Manifiesto de Carga es el **documento relacionado** de código "91" (Catálogo 61) y sus datos como conceptos "7024" a "7028" del Catálogo 55. |
| 23 — Ubicación | No existe como catálogo. | Los catálogos de ubicación de la fuente son el 13 (UBIGEO), el 63 (Puertos) y el 64 (Aeropuertos). |
| Tipo de vehículo, Moneda, Estado del comprobante | No aparecen en la fuente. | No se han inventado fichas; el tipo de vehículo y el estado del comprobante no tienen catálogo en el archivo entregado, y la moneda del GRE no se cataloga (se transporta como atributo `@currencyID`). |

## Parámetros y listas activas

### Parámetros

La hoja «Catálogos» del XLSX oficial **no contiene ninguna tabla de tipo «Parámetros»** [FUENTE: archivo `Cat_logos.txt`, 498 líneas, sin bloque de parámetros].

El único parámetro referenciado por la fuente es **«Parámetros (004)»**, citado en las hojas de validación «Guía-Remitente2_0» y «Guía-Transportista2_0» como el listado asociado al error 2108 «Presentacion fuera de fecha»:

> «La diferencia entre la fecha de recepción del XML y el valor del Tag UBL es mayor al límite del listado (1 día)» — ERROR 2108 — Presentacion fuera de fecha — REST — Parámetros (004)

[FUENTE: línea 15 del archivo `Gu_a_Remitente2_0.txt` y línea 15 del archivo `Gu_a_Transportista2_0.txt`]

La tabla de parámetros en sí se publica en el Portal de la SUNAT y no forma parte de los archivos entregados; el límite que la validación utiliza es de 1 día calendario.

### Listado de palabras clave (motivo de traslado)

Listado de la hoja «Palabras Clave» del mismo XLSX, usado para detectar el motivo de traslado a partir del texto libre de la «Descripción de motivo de traslado» (motivo "13" Otros). La fuente lo titula: *Descripción de motivo "Otros": mensaje de observación*.

- Archivo: [../datos/palabras-clave-motivo-traslado.csv](../datos/palabras-clave-motivo-traslado.csv) — 69 palabras.
- Columnas: `Palabra(s) clave(s)`, `Exacta`, `Dentro del texto` (marcação `X`).
- No se genera ficha `cat-<NN>-<slug>.md` porque no es un catálogo numerado del Anexo N.° 8.

### Otros listados

- El rótulo **«Otros Listados»** de la fuente da inicio al bloque del catálogo **D-37** (entidades que emiten autorizaciones especiales para el traslado).
- Los catálogos 03, 13 y 25 no traen filas de datos: remiten a listas externas (UN/ECE Rec 20, INEI, UNSPSC). Sus referencias textuales y URL están reproducidas en la sección «Filas de referencia de la fuente» de cada ficha.

### Vigencia (hoja «Control de Cambios» del XLSX oficial)

| Vigencia | Catálogo | Cambio |
|---|---|---|
| 2024-11-14 | 20 | Se agrega el código 19 (Traslado de mercancía extranjera). |
| 2024-11-14 | 55 | Se agregan nuevos conceptos (numeración de DAM/DS y datos del Manifiesto de Carga). |
| 2024-11-14 | 61 | Se agrega el tipo de documento 91 (Manifiesto de carga). |
| 2025-04-01 | 62 | Se agregan y se eliminan códigos de bienes normalizados. |

## Convenciones de los archivos CSV

- Separador de campos: `;` — cabecera en la primera línea.
- Codificación: UTF-8 (sin BOM).
- Los archivos con cabeceras de tres columnas (`Código;Nombre;GRE Aplicable`, `Partida arancelaria;Descripción;Código de producto SUNAT`, `Código;Nombre;Ubigeo`, `Código;Nombre;Abreviatura`) replican las columnas reales del catálogo.
- Ninguna descripción contiene el carácter `;`, por lo que no se requiere entrecomillado.
- Los catálogos 03, 13 y 25 se publican con su cabecera y sin filas de datos (el listado vive en una fuente externa).

## Referencias normativas

- R.S. 123-2022-SUNAT, Anexo N.° 14: Estándar UBL 2.1 Guía de remisión remitente / transportista (tags UBL, catálogos y atributos `@listURI` / `@schemeURI` citados en las fichas).
- SUNAT, XLSX de validaciones de la GRE `ValidacionesGREv20250421_0.xlsx` (hojas «Catálogos», «Guía-Remitente2_0», «Guía-Transportista2_0», «Palabras Clave», «General», «CódigosRetorno», «Control de Cambios»).
