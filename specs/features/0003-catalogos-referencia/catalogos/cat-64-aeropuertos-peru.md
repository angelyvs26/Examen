# Catálogo 64 — Aeropuertos del Perú

| Metadato | Valor |
|---|---|
| Código de catálogo | 64 |
| Nombre oficial | Aeropuertos del Perú |
| Columnas | `Código`, `Nombre`, `Ubigeo` |
| Número de filas | 31 |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [375] a [405] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-64-aeropuertos-peru.csv` |

## Uso en la GRE

Códigos de los aeropuertos del Perú, con su ubigeo. Se usa cuando el traslado tiene como origen o destino un aeropuerto.

## Tag / atributo XML que lo consume

- `/DespatchAdvice/cac:Shipment/cac:FirstArrivalPortLocation/cbc:ID` — Código del aeropuerto. Atributos: `@schemeAgencyName="PE:SUNAT"`, `@schemeName="Aeropuertos"`, `@schemeURI="urn:pe:gob:sunat:cpe:see:gem:catalogos:catalogo64"`.
- `/DespatchAdvice/cac:Shipment/cac:FirstArrivalPortLocation/cbc:LocationTypeCode` — Tipo de locación: "2" Aeropuerto (catálogo 64) o "1" Puerto (catálogo 63).

## Tabla

| Código | Nombre | Ubigeo |
|---|---|---|
| AQP | Rodríguez Ballón | 040104 |
| ANS | Andahuaylas | 030201 |
| ATA | Comandante FAP Germán Arias Graciani | 020604 |
| AYP | Coronel FAP Alfredo Mendívil Duarte | 050113 |
| CJA | Mayor Gral. FAP Armando Revoredo Iglesias | 060108 |
| CHM | Tnte. FAP Jaime De Montruil M. | 021809 |
| CUZ | Alejandro Velazco Astete | 080108 |
| CHH | Chachapoyas | 010101 |
| CIX | Capitán FAP José Quiñones G. | 140101 |
| HUU | Alférez FAP David Figueroa Fernandini | 100101 |
| ILO | Ilo | 180301 |
| IQT | Coronel FAP Francisco Secada Vignetta | 160101 |
| JAE | Jaén - Shumba | 060802 |
| JJI | Juanjuí | 220601 |
| JUL | Manco Cápac | 211101 |
| JAU | Francisco Carlé | 120430 |
| LIM | Internacional Jorge Chávez | 070101 |
| MBP | Moyobamba | 220101 |
| PIO | Capitán FAP Renán Elías Olivera | 110506 |
| PIU | Capitán FAP Carlos Concha Iberico | 200104 |
| PCL | Capitán FAP David Abensur Rengifo | 250105 |
| PEM | Padre Aldamiz | 170101 |
| RIJ | Juan Simons Vela - Rioja | 220801 |
| TCQ | Coronel FAP Carlos Ciriani Santa Rosa | 230101 |
| TYL | Capitán FAP Montes Arias | 200701 |
| TPP | Cadete FAP Guillermo del Castillo Paredes | 220901 |
| TIG | Tingo María | 100601 |
| TRU | Capitán FAP Carlos Martínez Pinillos | 130104 |
| TBP | Capitán FAP Pedro Canga Rodríguez | 240101 |
| ATG | Atalaya - Tnte. Gral. Gerardo Pérez Pinedo | 250201 |
| YMS | Moisés Benzaquen Rengifo | 160201 |

## CSV

`../datos/cat-64-aeropuertos-peru.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

