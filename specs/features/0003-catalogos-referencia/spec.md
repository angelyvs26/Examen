# 0003 — Catálogos SUNAT como dato de referencia

| Campo | Valor |
|---|---|
| Código | 0003 |
| Módulo | Catálogos de referencia SUNAT (Anexo N.° 8) |
| Estado | Borrador |
| Versión | 1.0.0 |
| Fecha | 2026-10-21 |
| Responsables | Equipo de la plataforma GRE |
| Depende de | — |
| Bloquea a | 0001 (emisión GRE Remitente), 0002 (GRE Transportista), 0004 (validación) |
| ADRs aplicables | [ADR-0001](../../adr/0001-seleccion-db.md), [ADR-0002](../../adr/0002-autenticacion-jwt.md) |
| Fichas de catálogo | [catalogos/README.md](catalogos/README.md) |
| Datos de origen | [datos/](datos/) (CSV con separador `;`, UTF-8 sin BOM) |

## 1. Objetivo

Dotar a la plataforma de los **catálogos oficiales del Anexo N.° 8** que la SUNAT exige para la Guía de Remisión Electrónica (GRE) —códigos `01`, `03`, `06`, `13`, `18`, `20`, `25`, `52`, `55`, `61`, `62`, `63`, `64`, `65` y `D-37`— y del **listado de palabras clave del motivo de traslado**, versionados por vigencia, **cargables desde los CSV** transcritos de la fuente oficial, **consultables por la API** y **consumibles por el motor de validación** (feature 0004) siempre contra la versión activa del catálogo.

El resultado es un dato de referencia único, auditable y trazable a la fuente oficial (hoja «Catálogos» y hoja «Control de Cambios» del XLSX `ValidacionesGREv20250421_0.xlsx`), que sustituye la codificación dispersa de códigos SUNAT dentro del código de emisión.

## 2. Alcance

### 2.1 Incluido

- Registro de los **15 catálogos** del Anexo N.° 8 con sus metadatos: código, nombre oficial, columnas, fuente y número de filas esperado.
- Carga de los **12 catálogos con datos** (443 ítems en total) desde los CSV de [datos/](datos/), de forma **idempotente** y **versionada por vigencia** (`vigencia_desde` / `vigencia_hasta`).
- Registro de los **3 catálogos sin datos en la fuente** (`03` UN/ECE Rec. 20, `13` ubigeos del INEI, `25` UNSPSC) con su cabecera y su referencia a la lista externa, sin inventar filas.
- Carga del **listado de 69 palabras clave** del motivo de traslado ([datos/palabras-clave-motivo-traslado.csv](datos/palabras-clave-motivo-traslado.csv)), asociado al catálogo `20` y al motivo `13` (Otros).
- **Validación del formato** de la fuente: cabecera esperada, separador `;`, codificación UTF-8 sin BOM, unicidad de códigos, número de filas esperado y detección de catálogos vacíos, con **reporte de errores por catálogo**.
- **Consulta por API** de catálogos e ítems con paginación y filtro de búsqueda, según los endpoints ya definidos en [openapi.yaml](../../contracts/openapi.yaml).
- **Consumo interno** por el motor de validación (feature 0004) mediante un puerto con **control de versión activa**, incluida la columna «GRE Aplicable» del catálogo `61` y el ubigeo de los catálogos `63`/`64`.
- Trazabilidad de cada importación: versión, origen, hash del CSV, filas leídas, filas insertadas y errores.

### 2.2 Excluido

- La **descarga automatizada** de las listas externas (UN/ECE Rec. 20, INEI, UNSPSC): la ingestión es manual y versionada en el repositorio (decisión D-3 del [design.md](design.md)).
- La **validación del contenido normativo** de cada fila: la plataforma transcribe literalmente; no corrige ortografía, no expande descripciones ni reordena códigos.
- Las **reglas de negocio** de la GRE y el mapeo de cada código a un ítem de la hoja de validación: pertenecen a la feature 0004.
- La **generación del XML UBL 2.1** y el consumo de los atributos `@listURI` / `@schemeURI` en el documento: pertenecen a las features 0001 y 0002.
- El **cálculo automático del motivo de traslado** a partir del texto libre: la feature 0003 carga y expone el listado de palabras clave; el algoritmo de detección pertenece a la feature 0004.
- La tabla de **«Parámetros (004)»** de la hoja de validación: la fuente no la incluye y sólo se referencia para el límite de 1 día del error 2108.
- Los catálogos solicitados que **no existen en la fuente** (09 ámbito SUNAT, 21 número de DAM, 22 número de manifiesto, 23 ubicación, tipo de vehículo, moneda, estado del comprobante): según el [README de catálogos](catalogos/README.md), esos datos viajan como documentos relacionados (catálogo `61`) o como conceptos (catálogo `55`).
- La **escritura de catálogos por API**: la importación es un proceso batch; no se expone endpoint de alta ni de modificación (coherente con [ADR-0002](../../adr/0002-autenticacion-jwt.md)).

## 3. Antecedentes y contexto

La R.S. 123-2022-SUNAT (Anexo N.° 14) define el estándar UBL 2.1 `DespatchAdvice` de la GRE y exige que cada valor codificado provenga del catálogo oficial correspondiente, identificado por `@listURI` o `@schemeURI` con el prefijo `urn:pe:gob:sunat:cpe:see:gem:catalogos:`. El Anexo N.° 8 es el catálogo de códigos que la SUNAT publica para la GRE.

La fuente única de esta feature es el XLSX oficial de validaciones `ValidacionesGREv20250421_0.xlsx`, del que se transcribió la hoja «Catálogos» (archivo de trabajo `Cat_logos.txt`, 498 líneas, una fila por línea con prefijo `[n]`) y la hoja «Palabras Clave». La transcripción se encuentra ya materializada en las 15 fichas de [catalogos/](catalogos/) y en los CSV de [datos/](datos/), con la convención de separador `;`, UTF-8 sin BOM y cabecera en la primera línea.

**Control de cambios de la fuente** (hoja «Control de Cambios»), que es la base del versionado por vigencia:

| Vigencia | Catálogo | Cambio |
|---|---|---|
| 2024-11-14 | 20 | Se agrega el código `19` (Traslado de mercancía extranjera). |
| 2024-11-14 | 55 | Se agregan los conceptos `7021` a `7028` (numeración de DAM/DS y datos del Manifiesto de Carga). |
| 2024-11-14 | 61 | Se agrega el tipo de documento `91` (Manifiesto de carga, MC). |
| 2025-04-01 | 62 | Se agregan y se eliminan códigos de bienes normalizados. |

**Inventario de catálogos con datos** (12 catálogos, 443 ítems):

| Código | Nombre oficial | Filas | Columnas del CSV |
|---|---|---|---|
| 01 | Código de tipo de documento | 40 | `Código;Descripción` |
| 06 | Código de tipo de documento de identidad | 12 | `Código;Descripción` |
| 18 | Código de modalidad de transporte | 2 | `Código;Descripción` |
| 20 | Código de motivo de traslado | 14 | `Código;Descripción` |
| 52 | Códigos de leyendas | 15 | `Código;Descripción` |
| 55 | Código de identificación del concepto tributario | 120 | `Código;Descripción` |
| 61 | Documentos relacionados aplicables a las GRE (versión 2.0) | 27 | `Código;Nombre;GRE Aplicable` |
| 62 | Bienes normalizados | 52 | `Partida arancelaria;Descripción;Código de producto SUNAT` |
| 63 | Puertos del Perú | 21 | `Código;Nombre;Ubigeo` |
| 64 | Aeropuertos del Perú | 31 | `Código;Nombre;Ubigeo` |
| 65 | Código de unidades de medida (para uso solo para la GRE en DAM o DS) | 97 | `Código;Descripción` |
| D-37 | Entidades que emiten autorizaciones especiales para el traslado | 12 | `Código;Nombre;Abreviatura` |

**Catálogos sin datos en la fuente** (3 catálogos, 0 ítems): `03` (UN/ECE Recommendation 20 Revision 13), `13` (catálogo de ubigeos del INEI) y `25` (UNSPSC v14_0801, nivel 3). Sus CSV se publican con cabecera y sin filas, y remiten a la lista externa.

Situación previa: los códigos se comparaban contra constantes incrustadas en el código de emisión y contra listas copiadas a mano, lo que impedía detectar los cambios de la SUNAT y mantener la trazabilidad de la fuente.

## 4. Glosario

| Término | Definición |
|---|---|
| Catálogo | Conjunto codificado de valores oficiales de SUNAT, identificado por un código del Anexo N.° 8 (`01` … `65`, `D-37`). |
| Ítem de catálogo | Fila de un catálogo: código, nombre y atributos variables según el catálogo. |
| Versión de catálogo | Conjunto de ítems con una vigencia concreta (`vigencia_desde` / `vigencia_hasta`). |
| Versión activa | Versión cuya vigencia cubre la fecha de la operación; a lo sumo una por catálogo. |
| Vigencia | Intervalo de fechas `[vigencia_desde, vigencia_hasta]` durante el cual una versión es la aplicable. |
| Anexo N.° 8 | Catálogo de códigos de SUNAT aplicable a los comprobantes electrónicos. |
| Anexo N.° 14 | Estándar UBL 2.1 de la GRE (R.S. 123-2022-SUNAT). |
| Hoja «Catálogos» | Hoja del XLSX oficial de validaciones de la GRE que contiene los 15 catálogos y sus filas. |
| Hoja «Control de Cambios» | Hoja del XLSX oficial que registra la fecha de vigencia de cada cambio de catálogo. |
| Hoja «Palabras Clave» | Hoja del XLSX oficial con las 69 palabras que permiten detectar el motivo de traslado a partir del texto libre. |
| `GRE Aplicable` | Columna del catálogo `61` que indica si el documento relacionado aplica a la GRE-Remitente, a la GRE-Transportista o a ambas. |
| Ubigeo de locación | Código de 6 dígitos asociado a un puerto (`63`) o aeropuerto (`64`), usado como dato de la locación. |
| Palabras clave | Listado auxiliar (no numerado) de 69 términos con marcas de coincidencia `Exacta` y `Dentro del texto`. |
| CSV maestro | Archivo de [datos/](datos/) que es la copia de trabajo de un catálogo, en UTF-8 sin BOM y separador `;`. |

## 5. Personas y permisos

| Actor | Descripción | Permisos |
|---|---|---|
| Motor de validación (feature 0004) | Componente interno que resuelve reglas de negocio y compara valores contra la versión activa. | Acceso de lectura interno al puerto de catálogos; sin exposure HTTP. |
| ERP / WMS | Sistema del contribuyente que consulta catálogos para poblar o validar sus datos. | `catalogos.leer` (scope del token, ver [ADR-0002](../../adr/0002-autenticacion-jwt.md)). |
| Usuario del contribuyente | Analista que consulta valores de catálogos desde la consola. | `catalogos.leer`. |
| Operador de la plataforma | Personal de soporte que ejecuta la importación batch y revisa los reportes. | `admin`; acceso al proceso batch, no a endpoints de escritura. |
| Job de importación | Proceso programado que carga los CSV y genera las versiones. | Credencial de servicio con `admin`; ejecución local o por planificador. |

## 6. Historias de usuario

### US-1 — Cargar los catálogos oficiales desde los CSV

**Como** operador de la plataforma
**Quiero** importar los CSV de [datos/](datos/) en la base de datos
**Para** que la plataforma disponga de los catálogos SUNAT sin depender de ficheros en caliente

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given los 15 CSV con la cabecera esperada y el separador `;`, When se ejecuta la importación, Then los 12 catálogos con datos quedan con 443 ítems en total y los 3 catálogos externos (`03`, `13`, `25`) quedan registrados con 0 ítems | Must |
| 2 | Given un CSV ya importado con el mismo contenido, When se reejecuta la importación, Then no se crea una versión nueva ni se duplican ítems | Must |
| 3 | Given una versión importada, When se consulta el catálogo, Then se devuelve su `vigenciaDesde`, su `vigenciaHasta` (o `null` si está vigente) y el `totalItems` | Must |
| 4 | Given el listado de palabras clave, When se ejecuta la importación, Then quedan 69 palabras con sus marcas `Exacta` y `Dentro del texto` | Must |
| 5 | Given un CSV con cabecera distinta de la esperada, When se ejecuta la importación, Then el catálogo se rechaza con el detalle de columnas y no se escribe ningún ítem | Must |
| 6 | Given un CSV con un código duplicado, When se ejecuta la importación, Then el catálogo se rechaza indicando los códigos duplicados | Must |
| 7 | Given un catálogo sin filas de datos que no sea `03`, `13` ni `25`, When se ejecuta la importación, Then se reporta `CATALOGO_VACIO` | Must |
| 8 | Given un CSV con un número de filas distinto del esperado para ese catálogo, When se ejecuta la importación, Then se reporta la desviación y la carga no se confirma | Should |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-1 | El conjunto de catálogos de la GRE es el de 15 códigos del Anexo N.° 8: `01`, `03`, `06`, `13`, `18`, `20`, `25`, `52`, `55`, `61`, `62`, `63`, `64`, `65`, `D-37`. | [catalogos/README.md](catalogos/README.md) |
| RN-2 | Cada catálogo se versiona por vigencia; la versión aplicable es la que cubre la fecha de la operación. | Hoja «Control de Cambios» del XLSX oficial |
| RN-3 | Los códigos y descripciones se reproducen **literalmente**: sin parafrasear, sin corregir ortografía y sin truncar. | Regla de transcripción, [catalogos/README.md](catalogos/README.md) |
| RN-4 | La clave de un ítem es el código **tal como aparece en la fuente**; dos códigos que sólo difieren en espacios (`U 3` y `U3` del catálogo `65`) son ítems distintos. | Catálogo `65`, [cat-65-unidades-medida-gre-dam-ds.md](catalogos/cat-65-unidades-medida-gre-dam-ds.md) |
| RN-5 | Los CSV usan separador `;`, codificación UTF-8 sin BOM, cabecera en la primera línea y ninguna descripción contiene `;`. | Convenciones de los CSV, [catalogos/README.md](catalogos/README.md) |
| RN-6 | El número de filas de cada catálogo con datos es el declarado en su ficha y se verifica en cada importación. | Fichas de [catalogos/](catalogos/) |
| RN-7 | Los catálogos `03`, `13` y `25` se registran sin ítems y conservan la referencia a su lista externa. | Fichas [cat-03](catalogos/cat-03-codigo-tipo-unidad-medida-comercial.md), [cat-13](catalogos/cat-13-codigo-ubicacion-geografica-ubigeo.md), [cat-25](catalogos/cat-25-codigo-producto-sunat.md) |
| RN-8 | Los cambios de vigencia de la fuente son: `2024-11-14` para los catálogos `20`, `55` y `61`; `2025-04-01` para el catálogo `62`. | Hoja «Control de Cambios» del XLSX oficial |
| RN-9 | Un catálogo no puede tener dos versiones con vigencias solapadas. | Decisión de diseño, [design.md](design.md) |
| RN-10 | La importación es idempotente: el hash SHA-256 del CSV determina si la versión ya existe. | Decisión de diseño, [design.md](design.md) |

**Validaciones / errores esperados**

| Código | Condición | Tipo | Comportamiento |
|---|---|---|---|
| `CAT-001` | Cabecera ausente o distinta de la esperada para el catálogo. | ERROR | Se rechaza el catálogo; no se inserta ningún ítem. |
| `CAT-002` | Separador distinto de `;` o número de columnas inconsistente. | ERROR | Se rechaza el catálogo indicando la línea y el número de columnas. |
| `CAT-003` | Codificación distinta de UTF-8 o presencia de BOM. | ERROR | Se rechaza el catálogo; no se intenta «limpiar» el fichero. |
| `CAT-004` | Código repetido dentro del catálogo. | ERROR | Se rechazan las filas duplicadas; la importación no se confirma. |
| `CAT-005` | Catálogo sin filas de datos y no previsto como externo (`03`, `13`, `25`). | ERROR | Se rechaza; se informa de que el catálogo llegó vacío. |
| `CAT-006` | Número de filas distinto del esperado para el catálogo. | ERROR | Se rechaza la versión; se informa del conteo real y el esperado. |
| `CAT-007` | Valor obligatorio vacío en una celda de datos. | ERROR | Se rechaza la fila indicando catálogo, línea y columna. |
| `CAT-008` | Archivo CSV que no corresponde a ningún catálogo registrado. | ERROR | Se ignora con aviso en el reporte; no se crea catálogo. |
| `CAT-009` | Solapamiento de vigencias con una versión existente del mismo catálogo. | ERROR | Se rechaza la versión; se informa del intervalo en conflicto. |
| `CAT-010` | Código de ítem no normalizable al tipo declarado (por ejemplo, ubigeo no numérico). | ERROR | Se rechaza la fila; se registra el valor original. |
| `VAL-004` | Valor consultado que no existe en el catálogo (error interno ya definido en la feature 0004). | ERROR | La validación de la GRE falla con el ítem de validación y el código de retorno de SUNAT. |

### US-2 — Consultar los catálogos vigentes por API

**Como** ERP del contribuyente
**Quiero** consultar un catálogo y sus ítems
**Para** poblar y validar mis datos contra la fuente oficial vigente

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given un token con scope `catalogos.leer`, When se invoca `GET /api/v1/catalogos`, Then se devuelve la lista de los 15 catálogos con nombre, vigencia y total de ítems | Must |
| 2 | Given un token válido, When se invoca `GET /api/v1/catalogos/20`, Then se devuelve el catálogo `20` con `vigenciaDesde` `2024-11-14` y `totalItems` 14 | Must |
| 3 | Given un token válido, When se invoca `GET /api/v1/catalogos/20/items?pagina=0&tamano=5`, Then se devuelven 5 ítems, el número de página, el tamaño y el total de elementos | Must |
| 4 | Given un token válido, When se invoca `GET /api/v1/catalogos/65/items?busqueda=TM`, Then se devuelven sólo los ítems cuyo código o nombre contiene `TM` | Must |
| 5 | Given un token válido, When se invoca `GET /api/v1/catalogos/03/items`, Then se devuelve una página vacía con `totalElementos` 0 | Must |
| 6 | Given un token válido, When se invoca `GET /api/v1/catalogos/99`, Then se devuelve `404` | Must |
| 7 | Given un token sin el scope `catalogos.leer`, When se invoca cualquiera de los tres endpoints, Then se devuelve `403` | Must |
| 8 | Given un token ausente o expirado, When se invoca cualquiera de los tres endpoints, Then se devuelve `401` | Must |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-11 | Los endpoints de catálogos son de sólo lectura y exigen el scope `catalogos.leer`. | [ADR-0002](../../adr/0002-autenticacion-jwt.md) |
| RN-12 | La consulta de ítems expone siempre la **versión activa** a la fecha de la petición; el histórico se consulta internamente. | [openapi.yaml](../../contracts/openapi.yaml) («Devuelve el catálogo vigente») |
| RN-13 | La respuesta de un ítem es genérica (`codigo`, `nombre`, `atributos`) y los atributos variables dependen del catálogo: `GRE Aplicable` en `61`, `Ubigeo` en `63` y `64`, `Abreviatura` en `D-37`, `Partida arancelaria` y `Código de producto SUNAT` en `62`. | Esquema `PaginaItemsCatalogo` de [openapi.yaml](../../contracts/openapi.yaml) y fichas de [catalogos/](catalogos/) |
| RN-14 | Los catálogos son datos públicos de referencia: no se filtran por RUC ni contienen datos del contribuyente. | [ADR-0001](../../adr/0001-seleccion-db.md) |
| RN-15 | El filtro `busqueda` actúa sobre el código y sobre el nombre/descripción del ítem, con coincidencia parcial sin distinción de mayúsculas. | [openapi.yaml](../../contracts/openapi.yaml) (parámetro `busqueda`, longitud máxima 100) |

### US-3 — Resolver el catálogo vigente desde el motor de validación

**Como** motor de validación de la feature 0004
**Quiero** resolver el valor activo de un catálogo en un instante dado
**Para** validar la GRE contra la versión vigente y no contra la última importada

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given un catálogo con una versión vigente, When se resuelve su versión activa, Then se devuelve la versión cuyo `vigencia_desde` es más reciente y cuyo `vigencia_hasta` es nulo o posterior a la fecha consultada | Must |
| 2 | Given un catálogo sin versión que cubra la fecha consultada, When se resuelve, Then se devuelve un error explícito de «catálogo sin versión vigente» y no una versión arbitraria | Must |
| 3 | Given una importación posterior que crea una versión nueva, When se resuelve de nuevo, Then se devuelve la versión nueva sin reiniciar la aplicación | Must |
| 4 | Given un documento con motivo `13`, When se consulta el listado de palabras clave, Then se devuelven las 69 palabras con las marcas `Exacta` y `Dentro del texto` | Must |
| 5 | Given un motivo de traslado, When se comprueba contra el catálogo `20`, Then el valor `01`–`19` vigente se reconoce y un valor inexistente se rechaza | Must |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-16 | La versión activa se calcula por fecha, no por «última importada»: una versión futura no sustituye a la vigente. | Decisión de diseño, [design.md](design.md) |
| RN-17 | El catálogo `20` gobierna el motivo de traslado y sus datos exigidos; su última versión registrada es la del `2024-11-14`. | Ficha [cat-20-motivo-traslado.md](catalogos/cat-20-motivo-traslado.md) y hoja «Control de Cambios» |
| RN-18 | El listado de palabras clave se aplica al motivo `13` (Otros) sobre el texto de `cbc:HandlingInstructions`; las columnas `Exacta` y `Dentro del texto` determinan el modo de coincidencia. | Hoja «Palabras Clave» del XLSX oficial, [catalogos/README.md](catalogos/README.md) |
| RN-19 | Los catálogos `63` y `64` exigen el ubigeo asociado al código de puerto o aeropuerto; el motor de validación lo usa como dato de la locación. | Fichas [cat-63](catalogos/cat-63-puertos-peru.md) y [cat-64](catalogos/cat-64-aeropuertos-peru.md) |
| RN-20 | El catálogo `61` declara su versión como `2.0`. | Ficha [cat-61-documentos-relacionados-gre.md](catalogos/cat-61-documentos-relacionados-gre.md) |

### US-4 — Determinar el documento relacionado admisible según el tipo de GRE

**Como** motor de validación de la feature 0004
**Quiero** saber si un tipo de documento relacionado aplica a la GRE-Remitente o a la GRE-Transportista
**Para** no admitir documentos que SUNAT rechaza

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given el catálogo `61` cargado, When se consulta el código `91` (Manifiesto de carga), Then se obtiene «solo remitente» | Must |
| 2 | Given el catálogo `61` cargado, When se consulta el código `31` (Guía de Remisión Transportista), Then se obtiene «solo transportista» | Must |
| 3 | Given el catálogo `61` cargado, When se consulta el código `01` (Factura), Then se obtiene «remitente, transportista» | Must |
| 4 | Given el catálogo `61` cargado, When se resume la columna «GRE Aplicable», Then se obtienen 11 códigos «solo remitente», 7 «solo transportista» y 9 «remitente, transportista», sobre 27 en total | Must |
| 5 | Given un motivo de traslado `01` (Venta) en una GRE-Remitente, When el documento relacionado es `31`, Then la validación lo rechaza por no aplicable | Must |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-21 | La columna «GRE Aplicable» del catálogo `61` es la autoridad sobre la admisibilidad del documento relacionado en cada tipo de GRE. | Ficha [cat-61-documentos-relacionados-gre.md](catalogos/cat-61-documentos-relacionados-gre.md) |
| RN-22 | Los valores de la columna son «solo remitente», «solo transportista» y «remitente, transportista»; el valor literal se conserva sin reinterpretar. | Ficha [cat-61-documentos-relacionados-gre.md](catalogos/cat-61-documentos-relacionados-gre.md) |
| RN-23 | El tipo de documento `09` (Guía de Remisión Remitente) y el `91` (Manifiesto de carga) son «solo remitente»; el `31` (Guía de Remisión Transportista) es «solo transportista». | Ficha [cat-61-documentos-relacionados-gre.md](catalogos/cat-61-documentos-relacionados-gre.md) |

### US-5 — Detectar el motivo de traslado a partir del texto libre

**Como** motor de validación de la feature 0004
**Quiero** contrastar la descripción del motivo con el listado de palabras clave
**Para** clasificar el motivo `13` (Otros) cuando el contribuyente lo describe en texto libre

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given el listado cargado, When se busca una coincidencia, Then se distinguen las 69 palabras marcadas `Exacta` de las 60 marcadas además `Dentro del texto` | Must |
| 2 | Given el listado cargado, When se inspecciona el contenido, Then las 69 palabras tienen la marca `Exacta` y ninguna está sin marcar | Must |
| 3 | Given un texto de motivo que no contiene ninguna palabra del listado, When se contrasta, Then no se propone un motivo y el documento se trata como `13` (Otros) | Should |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-24 | El listado de palabras clave tiene 69 entradas; las 69 están marcadas como `Exacta` y 60 también como `Dentro del texto`. | [datos/palabras-clave-motivo-traslado.csv](datos/palabras-clave-motivo-traslado.csv) |
| RN-25 | Las variantes con y sin tilde (`Importación`/`Importacion`, `Consignación`/`Consignacion`, `Exportación`/`Exportacion`) figuran como entradas distintas; la plataforma no normaliza el texto. | [datos/palabras-clave-motivo-traslado.csv](datos/palabras-clave-motivo-traslado.csv) y RN-3 |
| RN-26 | El listado de palabras clave no es un catálogo numerado del Anexo N.° 8 y no se expone bajo los endpoints `/api/v1/catalogos/{codigo}`. | [catalogos/README.md](catalogos/README.md) |

### US-6 — Verificar la fidelidad de los catálogos cargados

**Como** responsable de la plataforma
**Quiero** comparar lo cargado con los CSV maestros
**Para** garantizar que la plataforma reproduce la fuente oficial sin desviaciones

**Criterios de aceptación**

| # | Criterio | Prioridad |
|---|---|---|
| 1 | Given los 15 catálogos cargados, When se comparan con los CSV, Then los 443 ítems coinciden en código, nombre y atributos | Must |
| 2 | Given el catálogo `65`, When se inspeccionan sus códigos, Then los códigos con espacio interno (`L 6`, `M 6`, `U 3`, `U 6`) se conservan tal cual | Must |
| 3 | Given una versión de catálogo, When se consulta su trazabilidad, Then se obtienen la fuente, el hash del CSV, la fecha de vigencia y el número de filas | Must |
| 4 | Given los catálogos `03`, `13` y `25`, When se inspecciona su metadato, Then se obtiene la referencia a la lista externa y 0 ítems | Must |

**Reglas de negocio**

| ID | Regla | Origen |
|---|---|---|
| RN-27 | Cada versión registrada guarda el hash SHA-256 del CSV de origen; los ítems se pueden reconstruir desde ese CSV. | Requisito de integridad y trazabilidad |
| RN-28 | Los 12 catálogos con datos suman 443 ítems y el listado de palabras clave suma 69 entradas. | Fichas de [catalogos/](catalogos/) y [datos/](datos/) |

## 7. Flujo principal

### 7.1 Importación de catálogos

1. El operador invoca el proceso batch de importación, indicando el directorio de los CSV y, opcionalmente, la vigencia a atribuir.
2. El proceso recorre el registro de los 15 catálogos soportados y localiza el CSV correspondiente en [datos/](datos/).
3. Por cada CSV se calcula el hash SHA-256 del fichero.
4. Si el hash ya está registrado en `catalogo_version`, la importación de ese catálogo termina sin cambios (idempotencia) y se registra en el reporte como «ya aplicado».
5. Si el hash es nuevo, se valida el formato: presencia de BOM, decodificación UTF-8, separador `;`, cabecera igual a la esperada y número de columnas por fila.
6. Se leen las filas y se comprueba la unicidad de los códigos (literal, sin normalizar) y el número de filas esperado para ese catálogo.
7. Se normaliza cada fila al modelo de ítem (`codigo`, `nombre`, `atributos`) según la definición de columnas del catálogo.
8. Se cierra la vigencia de la versión anterior (asignando `vigencia_hasta`) y se crea la versión nueva con su `vigencia_desde`, su hash y su fuente.
9. Se insertan los ítems de la versión y, para el catálogo `20`, las 69 palabras clave.
10. Se persiste el reporte de importación con las filas leídas, insertadas, rechazadas y el detalle de errores.
11. El proceso emite las métricas de importación y deja traza en la auditoría con el actor, la versión y el hash.

### 7.2 Consulta por API

1. El ERP obtiene un token del emisor OAuth 2.0 con el scope `catalogos.leer` (ver [ADR-0002](../../adr/0002-autenticacion-jwt.md)).
2. El ERP invoca `GET /api/v1/catalogos` y la API devuelve la relación de catálogos con nombre, vigencia y total de ítems.
3. El ERP invoca `GET /api/v1/catalogos/{codigo}` y la API devuelve la cabecera y los metadatos de la **versión activa** a la fecha de la petición.
4. El ERP invoca `GET /api/v1/catalogos/{codigo}/items` con `pagina`, `tamano` y, opcionalmente, `busqueda`.
5. La API resuelve la versión activa, aplica el filtro de búsqueda sobre código y nombre, pagina el resultado y devuelve `contenido`, `pagina`, `tamano` y `totalElementos`.
6. Para los catálogos `03`, `13` y `25` la página se devuelve vacía con `totalElementos` 0 y el metadato apunta a la lista externa.

### 7.3 Consumo interno por el motor de validación

1. El motor de validación (feature 0004) solicita la versión activa de un catálogo para la fecha de la operación.
2. El proveedor interno resuelve la versión vigente y cachea los ítems por versión; la caché se invalida cuando se registra una versión nueva.
3. El motor compara los valores del documento contra los ítems de esa versión y, ante un valor inexistente, emite la violación con el ítem de validación y el código de retorno de SUNAT ([catalogo-errores-internos.md](../0004-validacion-documento/catalogo-errores-internos.md), `VAL-004`).
4. Para los documentos relacionados, el motor consulta la columna «GRE Aplicable» del catálogo `61` y compara el tipo de GRE en curso.
5. Para los catálogos `63` y `64`, el motor obtiene el ubigeo asociado al código de locación.

## 8. Flujos alternativos y de error

| ID | Situación | Disparo | Respuesta del sistema | Estado resultante |
|---|---|---|---|---|
| FA-1 | CSV con cabecera distinta de la esperada | Archivo de otro catálogo o cabecera alterada | El catálogo se rechaza con `CAT-001` y el detalle de columnas; los demás catálogos del lote se procesan | Versión no creada |
| FA-2 | CSV con BOM o codificación distinta de UTF-8 | Fichero exportado desde Excel en UTF-8 con BOM | El catálogo se rechaza con `CAT-003`; no se «limpia» el fichero | Versión no creada |
| FA-3 | Separador o número de columnas incorrecto | Fichero con `,` en lugar de `;` o fila truncada | El catálogo se rechaza con `CAT-002` indicando línea y columnas | Versión no creada |
| FA-4 | Código duplicado dentro del catálogo | Fila repetida en el CSV | El catálogo se rechaza con `CAT-004` listando los códigos duplicados | Versión no creada |
| FA-5 | Catálogo vacío inesperado | CSV con cabecera y sin filas para un catálogo con datos | El catálogo se rechaza con `CAT-005` | Versión no creada |
| FA-6 | Número de filas distinto del esperado | Ficha desactualizada o CSV truncado | El catálogo se rechaza con `CAT-006` indicando conteo real y esperado | Versión no creada |
| FA-7 | Importación repetida del mismo CSV | Reejecución del proceso batch | No se crea versión nueva; el reporte marca «ya aplicado» | Sin cambios |
| FA-8 | Solapamiento de vigencias | Se importa un CSV con `vigencia_desde` anterior a la versión vigente | Se rechaza con `CAT-009` indicando el intervalo en conflicto | Versión no creada |
| FA-9 | Fichero CSV sin catálogo asociado | Archivo extra en el directorio de datos | Se ignora con aviso `CAT-008` en el reporte; no se crea catálogo | Sin cambios |
| FA-10 | Fallo de base de datos durante la importación | Error de conexión o restricción | Se revierte la transacción del catálogo afectado; los catálogos ya confirmados permanecen | Versión no creada |
| FA-11 | Consulta de un catálogo inexistente | `GET /api/v1/catalogos/99` | `404` con `application/problem+json` | — |
| FA-12 | Consulta sin scope `catalogos.leer` | Token sin el scope requerido | `403` | — |
| FA-13 | Consulta sin token o con token expirado | Token ausente o `exp` vencido | `401` | — |
| FA-14 | Catálogo sin versión vigente | Se consulta un catálogo cuya vigencia expiró y no tiene sucesor | Error explícito «catálogo sin versión vigente»; no se devuelve la versión expirada | — |
| FA-15 | Motor de validación sin catálogo cargado | La feature 0004 se ejecuta antes de la primera importación | La validación falla con error de configuración `INT-002`; no se degrada silenciosamente a «sin catálogo» | — |

## 9. Requisitos funcionales

| ID | Requisito | Historia / regla que lo origina |
|---|---|---|
| RF-1 | Registrar los 15 catálogos con sus metadatos (código, nombre oficial, columnas, fuente, filas esperadas) | US-1, RN-1 |
| RF-2 | Importar los ítems de un catálogo desde su CSV de forma idempotente, versionada y trazable | US-1, RN-2, RN-10 |
| RF-3 | Validar el formato del CSV: cabecera, separador `;`, codificación UTF-8 sin BOM y unicidad de códigos | US-1, RN-4, RN-5 |
| RF-4 | Detectar catálogos vacíos y desviaciones del número de filas esperado, con reporte de errores por catálogo | US-1, RN-6, RN-7 |
| RF-5 | Registrar el historial de versiones con `vigencia_desde`/`vigencia_hasta` y resolver la versión activa por fecha | US-3, RN-9, RN-16 |
| RF-6 | Exponer la consulta de catálogos e ítems por API con paginación y filtro de búsqueda | US-2, RN-11, RN-15 |
| RF-7 | Proveer al motor de validación (feature 0004) un puerto de consulta con control de versión activa | US-3, RN-16, RN-19 |
| RF-8 | Cargar y exponer el listado de 69 palabras clave asociado al motivo `13` | US-5, RN-18, RN-24 |
| RF-9 | Exponer la columna «GRE Aplicable» del catálogo `61` para determinar la admisibilidad del documento relacionado | US-4, RN-21, RN-22 |
| RF-10 | Exponer los metadatos de cada catálogo (nombre oficial, columnas, fuente, total de ítems y vigencia) | US-2, US-6, RN-13 |
| RF-11 | Registrar los catálogos externos `03`, `13` y `25` sin ítems y con su referencia a la lista externa | US-1, US-6, RN-7 |
| RF-12 | Auditar toda importación y toda consulta de catálogo | US-1, US-6, RN-27 |
| RF-13 | Normalizar los atributos variables de cada fila según la definición de columnas de su catálogo | US-6, RN-13, RN-28 |

## 10. Requisitos no funcionales

| ID | Categoría | Requisito | Métrica / criterio de aceptación |
|---|---|---|---|
| RNF-1 | Rendimiento | Consulta de ítems con paginación y filtro | p95 < 200 ms con `tamano` 100 sobre el catálogo más grande (`65`, 97 ítems) |
| RNF-2 | Rendimiento | Importación completa de los 15 catálogos | < 60 s para 443 ítems + 69 palabras clave |
| RNF-3 | Disponibilidad | API de catálogos | 99,9 % mensual; su indisponibilidad no impide emitir (el motor de validación usa la vía interna con caché) |
| RNF-4 | Trazabilidad | Cada importación queda auditada con actor, catálogo, versión, hash del CSV y conteos | 100 % de las importaciones |
| RNF-5 | Idempotencia | Reejecutar la importación del mismo CSV no crea versiones ni ítems duplicados | 0 versiones duplicadas y 0 ítems duplicados |
| RNF-6 | Integridad | Cada versión conserva el hash SHA-256 del CSV de origen y los ítems se reconstruyen desde él | Verificación en cada importación y prueba de restauración |
| RNF-7 | Compatibilidad | Los valores cargados coinciden literalmente con el CSV (código, nombre y atributos, sin normalizar) | 443/443 ítems coincidentes en la prueba de fidelidad |
| RNF-8 | Seguridad | Lectura con scope `catalogos.leer`; importaciónbatch con `admin`; sin escritura por API | 0 accesos sin scope válido |
| RNF-9 | Aislamiento | Los catálogos no contienen datos del contribuyente y no se filtran por RUC | 0 datos personales en las tablas de catálogos |

## 11. Modelo de datos

| Entidad | Campos principales | Notas |
|---|---|---|
| `Catalogo` | codigo (PK), nombre, descripcion, columnas (JSONB: nombre, tipo, obligatorio), fuente, fuente_externa, activo | Registro maestro de los 15 catálogos; no versionado (la versión vive en `CatalogoVersion`). |
| `CatalogoVersion` | id, catalogo_codigo (FK), version, vigencia_desde, vigencia_hasta, fuente, fecha_control_cambios, hash_csv, num_filas, importado_en, importado_por | Inmutable una vez confirmada. Único por `(catalogo_codigo, version)` y por `(catalogo_codigo, hash_csv)`. |
| `CatalogoItem` | id, version_id (FK), codigo, nombre, atributos (JSONB), orden | Único por `(version_id, codigo)` con el código literal. |
| `CatalogoPalabraClave` | id, version_id (FK del catálogo `20`), palabra, exacta (bool), dentro_del_texto (bool), orden | Único por `(version_id, palabra)`. |
| `CatalogoImportacion` | id, version_id (FK), archivo, hash_csv, filas_leidas, filas_insertadas, filas_rechazadas, estado, errores (JSONB), iniciado_en, finalizado_en | Una fila por ejecución de importación de un catálogo. |
| `Auditoria` | sujeto, operacion, recurso, fecha, detalle | Append-only; reutiliza la tabla de la feature 0001. |

Invariantes garantizadas por el motor:

- Un catálogo tiene **a lo sumo una** versión con vigencia que cubra una fecha dada (sin solapamientos).
- El hash SHA-256 del CSV es único por catálogo: garantiza la idempotencia de la importación.
- El código de un ítem es único dentro de su versión y se conserva literal (sin normalizar espacios ni tildes).

## 12. Contratos

| Contrato | Ubicación | Descripción |
|---|---|---|
| OpenAPI | [openapi.yaml](../../contracts/openapi.yaml) | Superficie REST de catálogos: `GET /api/v1/catalogos`, `GET /api/v1/catalogos/{codigo}` y `GET /api/v1/catalogos/{codigo}/items` (schemas `Catalogo` y `PaginaItemsCatalogo`). |
| JSON Schema | [schema.json](../../contracts/schema.json) | Declara los valores esperados de los catálogos en el payload de emisión (códigos `01`, `06`, `18`, `20`, `D-37`, `61`, `63`/`64`, `25`, `55`, `13`). |
| gRPC | [grpc_service.proto](../../contracts/grpc_service.proto) | Referencia los catálogos en los mensajes de dominio; no expone métodos propios de catálogo. |
| Fichas de catálogo | [catalogos/](catalogos/) | Transcripción de la fuente: nombre oficial, columnas, número de filas, tag UBL consumidor y notas. |
| Datos maestros | [datos/](datos/) | CSV de carga, separador `;`, UTF-8 sin BOM. |
| Errores internos | [catalogo-errores-internos.md](../0004-validacion-documento/catalogo-errores-internos.md) | Catálogo `VAL-004` (valor fuera de catálogo) que el motor de validación reutiliza. |

## 13. Criterios de aceptación de la feature

- [ ] Los 15 catálogos del Anexo N.° 8 quedan registrados y consultables por `GET /api/v1/catalogos`.
- [ ] Los 12 catálogos con datos suman 443 ítems y coinciden literalmente con los CSV (código, nombre y atributos).
- [ ] Los catálogos `03`, `13` y `25` se consultan con 0 ítems y su metadato referencia la lista externa.
- [ ] El listado de palabras clave tiene 69 entradas, 69 con marca `Exacta` y 60 con `Dentro del texto`.
- [ ] Reejecutar la importación del mismo CSV no crea versiones ni ítems duplicados.
- [ ] Un CSV con cabecera alterada, BOM, separador incorrecto, código duplicado o número de filas desviado se rechaza con su código `CAT-xxx` y no deja datos a medias.
- [ ] `GET /api/v1/catalogos/20` devuelve la versión vigente (cambio del `2024-11-14`) con `totalItems` 14.
- [ ] `GET /api/v1/catalogos/65/items` pagina y filtra por `busqueda` y conserva los códigos con espacio interno (`U 3`, `U 6`).
- [ ] `GET /api/v1/catalogos/91` devuelve `404` y `GET /api/v1/catalogos/20/items` sin token devuelve `401`.
- [ ] El motor de validación resuelve la versión activa por fecha y falla de forma explícita si el catálogo no tiene versión vigente.
- [ ] La columna «GRE Aplicable» del catálogo `61` distingue 11 códigos «solo remitente», 7 «solo transportista» y 9 «remitente, transportista».
- [ ] Cada importación queda auditada con actor, versión, hash del CSV y conteos.

## 14. Trazabilidad

| Requisito | Artefacto | Prueba |
|---|---|---|
| RF-1 | `Catalogo` (registro maestro) | `CatalogoRegistroTest` — metadatos de los 15 códigos |
| RF-2 | `ImportadorCatalogosUseCase` | `ImportacionCatalogosIT` — carga de los 443 ítems y numeración de versiones |
| RF-3 | `ValidadorFormatoCsv`, `ValidadorUnicidadCodigos` | `ValidadorFormatoCsvTest` — cabecera, BOM, separador, duplicados |
| RF-4 | `VerificadorNumeroFilas`, `ImportacionRepositorio` | `ReporteImportacionTest` — catálogo vacío y desviación de filas |
| RF-5 | `CatalogoVersion`, `CatalogoRepositorio` | `VersionActivaIT` — versión activa por fecha y no solapamiento |
| RF-6 | `CatalogoController` | `CatalogoControllerIT` — paginación, búsqueda, 404, scope |
| RF-7 | `CatalogoVigenteProvider` | `CatalogoVigenteProviderIT` — paridad con la feature 0004 |
| RF-8 | `CatalogoPalabraClave`, `PalabraClaveRepositorio` | `PalabrasClaveIT` — 69 entradas y marcas |
| RF-9 | `CatalogoItem.atributos` (catálogo `61`) | `DocumentoRelacionadoAdmisibleIT` — 11/7/9 |
| RF-10 | `CatalogoController` (cabecera) | `CatalogoMetadatosIT` — nombre, columnas, fuente, total de ítems |
| RF-11 | `Catalogo` (campos `fuente_externa`, `filas_esperadas`) | `CatalogosExternosTest` — `03`, `13`, `25` sin ítems |
| RF-12 | `AuditoriaAspecto` | `AuditoriaImportacionTest` — registro por importación |
| RF-13 | `NormalizadorAtributosPorCatalogo` | `NormalizadorAtributosTest` — mapeo por columnas de cada catálogo |
| RNF-1 | `CatalogoRepositorio` | `CatalogoConsultaRendimientoTest` — p95 < 200 ms |
| RNF-5 | `ImportadorCatalogosUseCase` | `ImportacionIdempotenciaIT` — doble ejecución |
| RNF-7 | `ComparadorCsvBaseDatos` | `FidelidadCatalogosIT` — 443/443 ítems coincidentes |

## 15. Riesgos y supuestos

| ID | Riesgo / supuesto | Impacto | Mitigación |
|---|---|---|---|
| R-1 | La SUNAT modifica el XLSX de validaciones sin aviso. | Alto | Versionado por vigencia, hash por versión y procedimiento documentado de actualización (fase 5). |
| R-2 | La versión activa calculada por fecha no coincide con lo que SUNAT aplica en la fecha de recepción del XML. | Alto | La vigencia se toma del control de cambios oficial y se audita cada importación; el motor de validación (0004) usa la fecha de la operación. |
| R-3 | Códigos con formato heterogéneo (catálogo `65`: `U 3` frente a `U3`) provocan colisiones si se normalizan. | Medio | RN-4: la clave es el código literal; prueba de fidelidad que verifica la conservación literal. |
| R-4 | Los catálogos `03`, `13` y `25` se consultan vacíos y el motor de validación no puede validar contra ellos. | Medio | Se documenta que la validación de esos dominios queda fuera de alcance; se expone la referencia externa para carga manual. |
| R-5 | La fecha de vigencia inicial de los catálogos sin cambio registrado no consta en la fuente. | Medio | Se adopta la fecha de publicación del XLSX de trabajo (`2025-04-21`) y se marca `[NO VERIFICADO]` en el diseño. |
| R-6 | El catálogo `52` no es un catálogo del XML de la GRE y puede inducir a error su uso en la emisión. | Bajo | Se carga y se documenta como catálogo del Anexo N.° 8 aplicable a comprobantes de pago; la feature 0001 no lo consume. |
| R-7 | El listado de palabras clave no está numerado y no encaja en el patrón de código del contrato. | Medio | Se expone por vía interna y se anota la necesidad de un endpoint propio en «Trabajo pendiente». |
| S-1 | Se asume que los CSV de [datos/](datos/) son la copia maestra que se mantiene en el repositorio. | — | El proceso de importación usa el hash del fichero; cualquier divergencia queda registrada en el reporte. |
| S-2 | Se asume que el motor de validación (feature 0004) consumirá los catálogos a través del puerto interno, no por HTTP. | — | Si el motor se despliega aparte, se conserva el contrato del puerto y se añade un cliente HTTP. |

## 16. Preguntas abiertas

- ¿La vigencia inicial de los catálogos sin cambio registrado (01, 03, 06, 13, 18, 25, 52, 63, 64, 65, D-37) debe fijarse a la fecha de publicación del XLSX de trabajo (`2025-04-21`) o a la fecha en que SUNAT publicó cada catálogo? (responsable: negocio)
- ¿Debe el listado de palabras clave exponerse como un catálogo numerado propio (por ejemplo, un código reservado) o mediante un endpoint específico bajo `/api/v1/catalogos/20`? (responsable: arquitectura)
- ¿Se requiere consultar versiones históricas de un catálogo desde la API (parámetro de versión o de fecha) para auditar una GRE emitida en el pasado? (responsable: negocio)
- ¿La plataforma debe distinguir la versión del catálogo `61` declarada en la fuente (2.0) de la versión de la plataforma, o basta con la versión de vigencia? (responsable: negocio)
- ¿Quién responde por la transcripción cuando la SUNAT publica una versión con una estructura de columnas distinta de la actual? (responsable: equipo de la plataforma)

## 17. Trabajo pendiente (contratos)

- `contracts/openapi.yaml` — el esquema `Catalogo` no expone `version`, `estado`, `columnas` ni `filasEsperadas`; se propone añadirlos para que el cliente vea la versión activa y la estructura del catálogo.
- `contracts/openapi.yaml` — `GET /api/v1/catalogos/{codigo}` no admite un parámetro de versión o de fecha de vigencia, por lo que el histórico no es consultable; se propone `?vigencia=YYYY-MM-DD` o `?version=N`.
- `contracts/openapi.yaml` — el patrón del parámetro `codigo` (`^[0-9]{2}$|^D-37$`) no admite el listado de palabras clave; se propone un endpoint propio (por ejemplo, `GET /api/v1/catalogos/20/palabras-clave`) o un código reservado.
- `contracts/openapi.yaml` — el esquema `PaginaItemsCatalogo.contenido[].nombre` es genérico: para el catálogo `62` el nombre natural es la descripción y el código es la partida arancelaria; hay que documentar la correspondencia `codigo`/`nombre`/`atributos` por catálogo en la respuesta.
- `contracts/openapi.yaml` — no se documenta el scope requerido (`catalogos.leer`) de forma específica en cada operación de la etiqueta «Catálogos»; se propone añadirlo a cada `operationId`.
