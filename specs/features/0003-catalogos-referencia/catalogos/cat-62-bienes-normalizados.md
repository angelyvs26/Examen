# Catálogo 62 — Bienes normalizados

| Metadato | Valor |
|---|---|
| Código de catálogo | 62 |
| Nombre oficial | Bienes normalizados |
| Columnas | `Partida arancelaria`, `Descripción`, `Código de producto SUNAT` |
| Número de filas | 52 |
| Ubicación en la fuente | Hoja «Catálogos» del XLSX oficial `ValidacionesGREv20250421_0.xlsx`; filas [292] a [343] del archivo `Cat_logos.txt` |
| Archivo de datos | `../datos/cat-62-bienes-normalizados.csv` |

## Uso en la GRE

Listado de bienes normalizados por SUNAT: partida arancelaria, descripción y código de producto SUNAT asociado. La SUNAT valida que la partida arancelaria consignada en el ítem exista en este listado.

## Tag / atributo XML que lo consume

- Consumido por el campo «Partida arancelaria» del ítem: `/DespatchAdvice/cac:DespatchLine/cac:Item/cac:AdditionalItemProperty/cbc:NameCode` con valor "7020" y `/DespatchAdvice/cac:DespatchLine/cac:Item/cac:AdditionalItemProperty/cbc:Value` con la partida arancelaria.
- El «Código de producto SUNAT» de la fila corresponde al `cbc:ItemClassificationCode` del ítem (catálogo 25).

## Notas

- En la fuente, el bloque de datos de este catálogo está precedido por una fila de encabezado «Partida arancelaria».
- Se agregaron y eliminaron códigos de bienes normalizados con vigencia 2025-04-01 (hoja «Control de Cambios» del XLSX oficial).

## Tabla

| Partida arancelaria | Descripción | Código de producto SUNAT |
|---|---|---|
| 1701130000 | Azúcar de caña | 50161509 |
| 1701140000 | Los demás azúcares de caña | 50161509 |
| 1701910000 | Con adición de aromatizante o colorante | 50161509 |
| 1701999000 | Los demás | 50161509 |
| 1703100000 | Melaza de caña | 50161509 |
| 2207100000 | Alcohol etílico sin desnaturalizar con grado alcohólico volumétrico superior o igual al 80 % vol | 12352104 |
| 2207200010 | Alcohol carburante | 12352104 |
| 2207200090 | Los demás | 12352104 |
| 2208901000 | Alcohol etílico sin desnaturalizar con grado alcohólico volumétrico inferior al 80% vol | 12352104 |
| 1006200000 | Arroz descascarillado (arroz cargo o arroz pardo) | 50221101 |
| 1006300000 | Arroz semiblanqueado o blanqueado, incluso pulido o glaseado | 50221101 |
| 1006400000 | Arroz partido | 50221101 |
| 2302200000 | No vigente | 50221101 |
| 2601110000 | Sin aglomerar | 11101601 |
| 2601120000 | Aglomerados | 11101601 |
| 2601200000 | Piritas de hierro tostadas (cenizas de piritas) | 11101601 |
| 2602000000 | Minerales de manganeso y sus concentrados, incluidos los minerales de manganeso ferruginosos y sus concentrados con un contenido de manganeso superior o igual al 20 % en peso, sobre producto seco. | 11101611 |
| 2603000000 | Minerales de cobre y sus concentrados | 11101604 |
| 2604000000 | Minerales de níquel y sus concentrados | 11101606 |
| 2605000000 | Minerales de cobalto y sus concentrados | 11101615 |
| 2606000000 | Minerales de aluminio y sus concentrados | 11101605 |
| 2607000000 | Minerales de plomo y sus concentrados | 11101608 |
| 2608000010 | Concentrado de cinc de baja ley | 11101609 |
| 2608000090 | Los demás | 11101609 |
| 2609000000 | Minerales de estaño y sus concentrados | 11101610 |
| 2610000000 | Minerales de cromo y sus concentrados | 11101612 |
| 2611000000 | Minerales de volframio (tungsteno) y sus concentrados | 11101613 |
| 2612100000 | Minerales de uranio y sus concentrados | 11101603 |
| 2612200000 | Minerales de torio y sus concentrados | 11101620 |
| 2613100000 | Tostados | 11101614 |
| 2613900000 | Los demás | 11101614 |
| 2614000000 | Minerales de titanio y sus concentrados | 11101602 |
| 2615100000 | Minerales de circonio y sus concentrados | 11101623 |
| 2615900000 | Los demás | 11101600 |
| 2616100000 | Minerales de plata y sus concentrados | 11101607 |
| 2616901000 | Minerales de oro y sus concentrados | 11101616 |
| 2616909000 | Los demás | 11101600 |
| 2617100000 | Minerales de antimonio y sus concentrados | 11101622 |
| 2617900000 | Los demás | 11101600 |
| 2618000000 | Escorias granuladas (arena de escorias) de la siderurgia | 11101701 |
| 2619000000 | Escorias (excepto las granuladas), batiduras y demás desperdicios de la siderurgia | 11101701 |
| 2620110000 | Matas de galvanización | 11101701 |
| 2620190000 | Los demás | 11101701 |
| 2620210000 | Lodos de gasolina con plomo y lodos de compuestos antidetonantes con plomo | 11101701 |
| 2620290000 | Los demás | 11101701 |
| 2620300000 | Que contengan principalmente cobre | 11101701 |
| 2620400000 | Que contengan principalmente aluminio | 11101701 |
| 2620600000 | Que contengan arsénico, mercurio, talio o sus mezclas, de los tipos utilizados para la extracción de arsénico o de estos metales o para la elaboración de sus compuestos químicos | 11101701 |
| 2620910000 | Que contengan antimonio, berilio, cadmio, cromo, o sus mezclas | 11101701 |
| 2620990000 | Los demás | 11101701 |
| 2621100000 | Cenizas y residuos procedentes de la incineración de desechos municipales | 11101701 |
| 2621900000 | Las demás | 11101701 |

## Filas de referencia de la fuente (sin código de catálogo)

- [291]  | Partida arancelaria

## CSV

`../datos/cat-62-bienes-normalizados.csv` — misma tabla, separador `;`, codificación UTF-8, con cabecera.

