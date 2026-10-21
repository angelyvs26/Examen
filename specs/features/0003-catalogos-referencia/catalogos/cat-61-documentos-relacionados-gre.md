# Catálogo 61 — Documentos relacionados aplicables a las GRE (versión 2.0)

| Metadato | Valor |
|---|---|
| Código de catálogo | 61 |
| Nombre oficial | Documentos relacionados aplicables a las GRE (versión 2.0) |
| Columnas | `Código`, `Nombre`, `GRE Aplicable` |
| Número de filas | 27 |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [259] a [285] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-61-documentos-relacionados-gre.csv` |

## Uso en la GRE

Tipo de documento relacionado que sustenta el traslado (factura, boleta, DAM, DS, manifiesto de carga, autorizaciones de SUCAMEC/MINSA/SERFOR, etc.). La columna "GRE Aplicable" indica si el documento puede usarse en la GRE-Remitente, en la GRE-Transportista o en ambas.

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:AdditionalDocumentReference/cbc:DocumentTypeCode` — Código del tipo de documento relacionado. Atributos: `@listAgencyName="PE:SUNAT"`, `@listName="Documento relacionado al transporte"`, `@listURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo61"`.
- `/DespatchAdvice/cac:AdditionalDocumentReference/cbc:ID` — Número del documento relacionado.
- `/DespatchAdvice/cac:AdditionalDocumentReference/cbc:DocumentType` — Descripción (texto libre) del documento relacionado.

## Notas

- Códigos de interés para la GRE: 09 (Guía de Remisión Remitente), 49 (Constancia de Depósito - IVAP), 50 (Declaración Aduanera de Mercancías), 52 (Declaración Simplificada), 80 (Constancia de Depósito - Detracción) y 91 (Manifiesto de carga, MC).
- El código 91 (Manifiesto de carga) se incorporó con vigencia 2024-11-14 (hoja «Control de Cambios» del XLSX oficial).
- La versión del catálogo declarada en la fuente es la 2.0.

## Tabla

| Código | Nombre | GRE Aplicable |
|---|---|---|
| 01 | Factura | remitente, transportista |
| 03 | Boleta de Venta | remitente, transportista |
| 04 | Liquidación de Compra | remitente, transportista |
| 09 | Guía de Remisión Remitente | remitente, transportista |
| 12 | Ticket o cinta emitido por máquina registradora | remitente, transportista |
| 31 | Guía de Remisión Transportista | solo transportista |
| 48 | Comprobante de Operaciones – Ley N° 29972 | remitente, transportista |
| 49 | Constancia de Depósito - IVAP (Ley 28211) | solo remitente |
| 50 | Declaración Aduanera de Mercancías | remitente, transportista |
| 52 | Declaración Simplificada (DS) | remitente, transportista |
| 65 | Autorización de Circulación para transportar MATPEL – Callao | solo transportista |
| 66 | Autorización de Circulación para transporte de carga y mercancías en Lima Metropolitana | solo transportista |
| 67 | Permiso de Operación Especial para el servicio de transporte de MATPEL - MTC | solo transportista |
| 68 | Habilitación Sanitaria de Transporte Terrestre de Productos Pesqueros y Acuícolas | solo transportista |
| 69 | Permiso / Autorización de operación de transporte de mercancías | solo transportista |
| 71 | Resolución de Adjudicación de bienes – SUNAT | solo remitente |
| 72 | Resolución de Comiso de bienes – SUNAT | solo remitente |
| 73 | Guía de Transporte Forestal o de Fauna - SERFOR | solo remitente |
| 74 | Guía de Tránsito – SUCAMEC | solo remitente |
| 75 | Autorización para operar como empresa de Saneamiento Ambiental – MINSA -  | solo remitente |
| 76 | Autorización para manejo y recojo de residuos sólidos peligrosos y no peligrosos | solo remitente |
| 77 | Certificado fitosanitario la movilización de plantas, productos vegetales, y otros artículos reglamentados  | solo remitente |
| 78 | Registro Único de Usuarios y Transportistas de Alcohol Etílico | solo remitente |
| 80 | Constancia de Depósito – Detracción | remitente, transportista |
| 81 | Código de autorización emitida por el SCOP | solo remitente |
| 82 | Declaración jurada de mudanza | solo transportista |
| 91 | Manifiesto de carga (MC) | solo remitente |

## CSV

`../datos/cat-61-documentos-relacionados-gre.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

