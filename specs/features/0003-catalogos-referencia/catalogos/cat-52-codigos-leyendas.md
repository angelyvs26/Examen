# Catálogo 52 — Códigos de leyendas

| Metadato | Valor |
|---|---|
| Código de catálogo | 52 |
| Nombre oficial | Códigos de leyendas |
| Columnas | `Código`, `Descripción` |
| Número de filas | 15 |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [115] a [129] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-52-codigos-leyendas.csv` |

## Uso en la GRE

Catálogo oficial de leyendas del Anexo N.° 8. No es un catálogo del XML de la GRE (la guía de remisión no transporta leyendas); se consume en los comprobantes de pago electrónico que sustentan el traslado (por ejemplo, la leyenda "2012 Observaciones relacionadas con el traslado de mercaderías").

## Tag / atributo XML que lo consume

- `/Invoice/cbc:Note@languageLocaleID` y `/Invoice/cbc:Note` — Código y descripción de la leyenda en la factura electrónica (Catálogo N.° 52 del Anexo N.° 8). No forma parte de la estructura `DespatchAdvice` de la GRE.

## Notas

- No referenciado por el Anexo N.° 14 (Estándar UBL 2.1 Guía de remisión - Remitente / Transportista) de la R.S. 123-2022-SUNAT.

## Tabla

| Código | Descripción |
|---|---|
| 1000 | Monto en Letras |
| 1002 | Leyenda "TRANSFERENCIA GRATUITA DE UN BIEN Y/O SERVICIO PRESTADO GRATUITAMENTE" |
| 2000 | Leyenda “COMPROBANTE DE PERCEPCIÓN” |
| 2001 | Leyenda “BIENES TRANSFERIDOS EN LA AMAZONÍA REGIÓN SELVA PARA SER CONSUMIDOS EN LA MISMA" |
| 2002 | Leyenda “SERVICIOS PRESTADOS EN LA AMAZONÍA  REGIÓN SELVA PARA SER CONSUMIDOS EN LA MISMA” |
| 2003 | Leyenda “CONTRATOS DE CONSTRUCCIÓN EJECUTADOS  EN LA AMAZONÍA REGIÓN SELVA” |
| 2004 | Leyenda “Agencia de Viaje - Paquete turístico”  |
| 2005 | Leyenda “Venta realizada por emisor itinerante”  |
| 2006 | Leyenda "Operación sujeta a detracción" |
| 2007 | Leyenda "Operación sujeta al IVAP" |
| 2008 | Leyenda: “VENTA EXONERADA DEL IGV-ISC-IPM. PROHIBIDA LA VENTA FUERA DE LA ZONA COMERCIAL DE TACNA” |
| 2009 | Leyenda: “PRIMERA VENTA DE MERCANCÍA IDENTIFICABLE ENTRE USUARIOS DE LA ZONA COMERCIAL” |
| 2010 | Restitucion Simplificado de Derechos Arancelarios |
| 2011 | Leyenda “EXPORTACION DE SERVICIOS - DECRETO LEGISLATIVO Nº 919” |
| 2012 | Observaciones relacionadas con el traslado de mercaderías |

## CSV

`../datos/cat-52-codigos-leyendas.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

