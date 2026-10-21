# 0003 — Diseño técnico: Catálogos SUNAT como dato de referencia

| Campo | Valor |
|---|---|
| Código | 0003 |
| Feature | [Catálogos SUNAT como dato de referencia](spec.md) |
| Estado | Borrador |
| Fecha | 2026-10-21 |
| Spec de referencia | [spec.md](spec.md) |
| ADRs aplicados | [ADR-0001](../../adr/0001-seleccion-db.md), [ADR-0002](../../adr/0002-autenticacion-jwt.md) |

## 1. Resumen de la solución

Los catálogos se almacenan en **PostgreSQL con versionado por vigencia**, no como ficheros leídos en caliente. Cada catálogo tiene una tabla de metadatos (`catalogo`) y una cadena de versiones (`catalogo_version`); cada versión guarda sus ítems (`catalogo_item`), el hash SHA-256 del CSV de origen y el intervalo `[vigencia_desde, vigencia_hasta]`. La **versión activa** se resuelve por la fecha de la operación y no por «última importada», de modo que una versión con vigencia futura no sustituya a la vigente. Esta elección sigue la línea marcada por [ADR-0001](../../adr/0001-seleccion-db.md), que ya prevé `catalogo` y `catalogo_item` con `vigencia_desde` / `vigencia_hasta` y `codigo_sunat` indexado, y evita que el motor de validación (feature 0004) resuelva catálogos distintos según el momento en que cada réplica tiene su caché.

La carga se realiza con un **proceso batch idempotente**: el hash del CSV determina si la versión ya existe; si existe, la importación no vuelve a escribir nada. Cada importación pasa por una cadena de validación (BOM, codificación, separador, cabecera, unicidad de códigos, número de filas esperado) y produce un **reporte por catálogo** con filas leídas, insertadas, rechazadas y el detalle de errores. Un catálogo que falla no deja datos a medias: la transacción de ese catálogo se revierte y el resto del lote continúa.

El consumo se resuelve por dos vías sobre el mismo modelo: una **API REST** de sólo lectura (`GET /api/v1/catalogos`, `GET /api/v1/catalogos/{codigo}`, `GET /api/v1/catalogos/{codigo}/items`) con paginación y filtro de búsqueda, y un **puerto interno** (`CatalogoVigenteProvider`) que el motor de validación consume con control de versión activa y caché invalidable por versión. Los atributos variables de cada fila (columna «GRE Aplicable» del catálogo `61`, ubigeo de `63`/`64`, abreviatura de `D-37`, partida arancelaria de `62`) se modelan en una columna `JSONB` con la definición de columnas por catálogo; así se evita una tabla distinta por catálogo y la respuesta coincide con la forma genérica `atributos` del contrato OpenAPI.

Los catálogos `03`, `13` y `25` se registran **sin ítems**, con la referencia a su lista externa. No se inventan filas ni se descargan de forma automatizada: la fuente externa queda documentada y la validación de esos dominios se resuelve en el motor, no en la carga.

## 2. Contexto técnico

### 2.1 Restricciones

| ID | Restricción | Origen |
|---|---|---|
| C-1 | Los 15 catálogos del Anexo N.° 8 son dato público de referencia, compartido por todos los contribuyentes. | [ADR-0002](../../adr/0002-autenticacion-jwt.md) |
| C-2 | Los códigos y descripciones se reproducen literalmente: sin normalizar mayúsculas, tildes ni espacios. | Regla de transcripción, [catalogos/README.md](catalogos/README.md) |
| C-3 | Los CSV tienen separador `;`, UTF-8 sin BOM, cabecera en la primera línea y ninguna descripción con `;`. | Convenciones de los CSV, [catalogos/README.md](catalogos/README.md) |
| C-4 | El número de filas de cada catálogo es el declarado en su ficha (01: 40, 06: 12, 18: 2, 20: 14, 52: 15, 55: 120, 61: 27, 62: 52, 63: 21, 64: 31, 65: 97, D-37: 12). | Fichas de [catalogos/](catalogos/) |
| C-5 | El versionado se basa en la hoja «Control de Cambios»: `2024-11-14` (catálogos `20`, `55`, `61`) y `2025-04-01` (catálogo `62`). | XLSX oficial `ValidacionesGREv20250421_0.xlsx` |
| C-6 | La API de catálogos es de sólo lectura; la carga es un proceso batch. | [ADR-0002](../../adr/0002-autenticacion-jwt.md) («sin escritura por API») |
| C-7 | El motor de validación (feature 0004) necesita la versión **vigente a la fecha de la operación**, no la última disponible. | [spec.md](spec.md), RN-16 |
| C-8 | La migración de catálogos es la `V3`, ya reservada en el diseño de la feature 0001. | [design.md](../0001-emision-gre-remitente/design.md) §5.3 |
| C-9 | La caché de catálogos debe invalidarse por versión sin reiniciar el servicio. | Requisito de actualización continua |

### 2.2 Supuestos y dependencias

| ID | Supuesto / dependencia | Impacto si no se cumple |
|---|---|---|
| S-1 | Los CSV de [datos/](datos/) son la copia maestra mantenida en el repositorio. | La importación debe apuntar a otra ubicación; el hash sigue siendo la clave de idempotencia. |
| S-2 | El motor de validación (feature 0004) se despliega en el mismo proceso y consume el puerto interno. | Si se despliega aparte, hay que añadir un cliente HTTP con el mismo contrato del puerto. |
| S-3 | La fecha de vigencia inicial de los catálogos sin cambio registrado se toma de la publicación del XLSX de trabajo (`2025-04-21`) [NO VERIFICADO: la fuente no declara fecha de vigencia inicial para `01`, `03`, `06`, `13`, `18`, `25`, `52`, `63`, `64`, `65` y `D-37`]. | La versión activa de esos catálogos puede adelantarse o retrasarse respecto de SUNAT; se corrige con una versión nueva. |
| S-4 | El directorio de datos es el mismo para todos los catálogos y el nombre del fichero sigue el patrón `cat-<código>-<slug>.csv`. | El registro de catálogos debe parametrizar la ruta por catálogo. |
| S-5 | No hay datos personales ni datos por RUC en los catálogos. | Habría que revisar el modelo de aislamiento (Row Level Security) de [ADR-0001](../../adr/0001-seleccion-db.md). |
| S-6 | La SUNAT publica sus cambios con una periodicidad conocida y documentada. | La reacción a un cambio depende de la vigilancia del XLSX oficial (riesgo R-1 de la spec). |

## 3. Arquitectura

### 3.1 Flujo de importación

```text
   Operador / planificador
            │
            │  CLI: catalogos:importar [--directorio datos] [--vigencia-desde 2025-04-21] [--dry-run]
            ▼
 ┌──────────────────────────────────────────────────────────────────────────┐
 │  CatalogoImportacionRunner   (com.examen.gre.catalogos.infraestructura.batch) │
 └───────────────────────────────────┬──────────────────────────────────────┘
                                     ▼
 ┌──────────────────────────────────────────────────────────────────────────┐
 │  ImportadorCatalogosUseCase   (…catalogos.aplicacion.importacion)        │
 │  ┌────────────────────────────────────────────────────────────────────┐  │
 │  │ RegistroCatalogos → metadatos de los 15 catálogos (código,         │  │
 │  │                     columnas, filas esperadas, fuente externa)    │  │
 │  └────────────────────────────────────────────────────────────────────┘  │
 │                                     ▼                                    │
 │  ┌───────────────┐   ┌──────────────────┐   ┌────────────────────────┐   │
 │  │ LectorCsv     │──▶│ ValidadorFormato │──▶│ ValidadorUnicidadCodig │   │
 │  │ (; UTF-8)     │   │ (BOM, cabecera,  │   │ (literal, por catálogo)│   │
 │  └───────────────┘   │  separador, cols)│   └───────────┬────────────┘   │
 │                      └──────────────────┘               │                │
 │                                     ▼                   ▼                │
 │                      ┌──────────────────┐   ┌────────────────────────┐   │
 │                      │ VerificadorNumero │   │ NormalizadorAtributos │   │
 │                      │ deFilas          │   │ PorCatalogo           │   │
 │                      └────────┬─────────┘   └───────────┬────────────┘   │
 └───────────────────────────────┼─────────────────────────┼────────────────┘
                                 │                         │
                                 ▼                         ▼
 ┌──────────────────────────────────────────────────────────────────────────┐
 │  Persistencia (PostgreSQL, ADR-0001)                                    │
 │  catalogo → catalogo_version → catalogo_item / catalogo_palabra_clave   │
 │  catalogo_importacion (reporte) · auditoria (append-only)               │
 └──────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Flujo de consulta

```text
   ERP / WMS                Motor de validación (feature 0004)
      │  JWT + scope catalogos.leer                  │
      ▼                                            ▼
 ┌───────────────────────────────┐      ┌───────────────────────────────┐
 │ CatalogoController            │      │ CatalogoVigenteProvider       │
 │ (…catalogos.interfaz.rest)    │      │ (…catalogos.dominio.puerto)   │
 │ GET /api/v1/catalogos         │      │ versión activa por fecha       │
 │ GET /api/v1/catalogos/{codigo}│      └──────────────┬────────────────┘
 │ GET /api/v1/catalogos/{cod}/items                  │
 └───────────────┬───────────────────────────────────┘ │
                 ▼                                    ▼
 ┌──────────────────────────────────────────────────────────────────────────┐
 │  CatalogoConsultaService / CatalogoVersionService                       │
 │  (…catalogos.aplicacion.consulta, …catalogos.dominio.servicio)           │
 └───────────────┬───────────────────────────────────┬────────────────────┘
                 ▼                                   ▼
 ┌────────────────────────────────┐   ┌────────────────────────────────────┐
 │ CacheCatalogoVigente           │   │ CatalogoRepositorio /              │
 │ (por versión; invalidable)     │   │ PalabraClaveRepositorio (JDBC)     │
 └───────────────┬────────────────┘   └──────────────┬─────────────────────┘
                 └──────────────┬─────────────────────┘
                                ▼
                  PostgreSQL (ADR-0001) — tablas de catálogos
```

| Componente | Responsabilidad | Ubicación |
|---|---|---|
| `ImportadorCatalogosUseCase` | Orquesta la importación de un catálogo: validación, versionado, inserción y reporte | `com.examen.gre.catalogos.aplicacion.importacion` |
| `RegistroCatalogos` | Declaración de los 15 catálogos (código, nombre oficial, columnas, filas esperadas, fuente externa) | `com.examen.gre.catalogos.dominio.registro` |
| `LectorCsvCatalogo` | Lectura en streaming del CSV con separador `;` y decodificación UTF-8 estricta | `com.examen.gre.catalogos.infraestructura.csv` |
| `ValidadorFormatoCsv` | BOM, codificación, separador, cabecera esperada y consistencia de columnas | `com.examen.gre.catalogos.aplicacion.importacion.validacion` |
| `ValidadorUnicidadCodigos` | Unicidad del código literal dentro del catálogo | `com.examen.gre.catalogos.aplicacion.importacion.validacion` |
| `VerificadorNumeroFilas` | Contraste con el número de filas esperado del catálogo | `com.examen.gre.catalogos.aplicacion.importacion.validacion` |
| `NormalizadorAtributosPorCatalogo` | Mapeo de las columnas del CSV al modelo de ítem (`codigo`, `nombre`, `atributos`) | `com.examen.gre.catalogos.aplicacion.importacion.normalizacion` |
| `Catalogo`, `CatalogoVersion`, `CatalogoItem`, `CatalogoPalabraClave` | Modelo de dominio | `com.examen.gre.catalogos.dominio.modelo` |
| `CatalogoVersionService` | Resolución de la versión activa por fecha y control de no solapamiento | `com.examen.gre.catalogos.dominio.servicio` |
| `CatalogoVigenteProvider` | Puerto de lectura para el motor de validación (feature 0004) | `com.examen.gre.catalogos.dominio.puerto` |
| `CatalogoConsultaService` | Casos de uso de consulta: catálogo, ítems con paginación y búsqueda | `com.examen.gre.catalogos.aplicacion.consulta` |
| `CatalogoRepositorio` | Acceso a `catalogo`, `catalogo_version` y `catalogo_item` | `com.examen.gre.catalogos.infraestructura.persistencia` |
| `PalabraClaveRepositorio` | Acceso a `catalogo_palabra_clave` | `com.examen.gre.catalogos.infraestructura.persistencia` |
| `ImportacionRepositorio` | Persistencia del reporte de importación | `com.examen.gre.catalogos.infraestructura.persistencia` |
| `CacheCatalogoVigente` | Caché de ítems por versión, invalidable al registrar una versión nueva | `com.examen.gre.catalogos.infraestructura.cache` |
| `CatalogoController` | Endpoints REST de lectura y traducción de errores | `com.examen.gre.catalogos.interfaz.rest` |
| `MapeadorItemCatalogo` | Proyección del ítem al formato `codigo` / `nombre` / `atributos` del contrato | `com.examen.gre.catalogos.interfaz.rest.mapeo` |
| `CatalogoImportacionRunner` | Punto de entrada batch (CLI) de la importación | `com.examen.gre.catalogos.infraestructura.batch` |
| `AuditoriaAspecto` | Registro append-only de importaciones y consultas | `com.examen.gre.auditoria` (transversal) |

## 4. Decisiones de diseño

### 4.1 Almacenamiento en PostgreSQL con vigencia, en lugar de ficheros en caliente

- **Contexto**: los catálogos son el dato contra el que se valida cada GRE emitida. Un fichero en caliente (CSV servido o montado en el contenedor) obliga a recargar o reiniciar el proceso en cada cambio de la SUNAT, impide responder «qué versión estaba vigente el día de la emisión» y no permite auditar qué se cargó. [ADR-0001](../../adr/0001-seleccion-db.md) ya anticipa `catalogo` y `catalogo_item` con `vigencia_desde` / `vigencia_hasta`.
- **Decisión**: los catálogos se persisten en PostgreSQL 16 (migración `V3__catalogos.sql`), con una tabla de metadatos por catálogo y una tabla de versiones. Cada ítem pertenece a una versión; la versión activa se resuelve por la fecha de la operación. El CSV sigue siendo la **copia maestra en el repositorio**, pero es la entrada de la carga, no la fuente de lectura en caliente.
- **Alternativas descartadas**: *ficheros CSV leídos en cada petición* — sin versionado ni auditoría, coste de E/S por validación y riesgo de fichero a medio copiar; *Redis como caché de catálogos* — no es fuente de verdad ni resuelve el histórico; *tabla única sin versionado* — imposible responder por la versión vigente de una GRE emitida el día anterior; *carga de catálogos externos desde la web* — ver decisión 4.3.
- **Consecuencias**: la base de datos pasa a ser la fuente de verdad del dato de referencia, por lo que su respaldo pasa a ser crítico y debe entrar en la prueba de restauración. Se gana trazabilidad por hash y vigencia, y se puede responder el histórico. A cambio, cada importación es una escritura y hay que garantizar que el backup incluya las tablas de catálogos.

### 4.2 Versionado inmutable e idempotencia de la importación

- **Contexto**: la hoja «Control de Cambios» del XLSX oficial registra vigencias (`2024-11-14`, `2025-04-01`); los catálogos se corrigen a lo largo del tiempo y el proceso de carga puede ejecutarse más de una vez (reintentos, despliegue, pipeline).
- **Decisión**: cada versión es **inmutable**: una vez confirmada, ni sus ítems ni su vigencia se modifican. La idempotencia se apoya en el **hash SHA-256 del CSV**: `catalogo_version` tiene una restricción única sobre `(catalogo_codigo, hash_csv)`. Al importar, si el hash ya existe, el proceso termina sin escribir. Para una versión nueva se cierra la vigencia de la versión anterior (`vigencia_hasta = vigencia_desde - 1 día`) y se abre la nueva. Una restricción de exclusión de PostgreSQL (`EXCLUDE USING gist`) impide dos versiones solapadas del mismo catálogo.
- **Alternativas descartadas**: *actualizar en sitio los ítems del catálogo* — destruye el histórico y hace imposible auditar una GRE ya emitida; *borrar y recargar* — mismo problema y ventana de inconsistencia; *clave idempotente externa del proceso* — no protege frente a dos ejecuciones concurrentes; *`INSERT ... ON CONFLICT DO NOTHING` sobre los ítems* — no distingue «mismo contenido» de «contenido distinto con códigos coincidentes».
- **Consecuencias**: la carga es segura de reintentar y la concurrencia entre dos procesos de importación se resuelve por la restricción única. Cada versión queda congelada, lo que obliga a que la corrección de un dato se materialice como una versión nueva con su vigencia, y por tanto a que el proceso de actualización sea explícito. La caché de versión activa se invalida al confirmar una versión.

### 4.3 Carga de los catálogos externos `03`, `13` y `25`

- **Contexto**: la hoja «Catálogos» no incluye las filas de tres catálogos: `03` remite al estándar UN/ECE Recommendation 20 Rev. 13, `13` al catálogo de ubigeos del INEI y `25` a UNSPSC v14_0801 (nivel 3). Sus CSV se publican con cabecera y sin filas.
- **Decisión**: los tres catálogos se registran en `catalogo` con `filas_esperadas = 0`, `fuente_externa` y la URL de la lista oficial, y **no se crean ítems**. La API responde `200` con `totalItems = 0` y una página vacía. La validación de valores de esos dominios (ubigeos, UNSPSC, unidades UN/ECE) queda en el motor de validación (feature 0004), no en la carga.
- **Alternativas descartadas**: *descargar y parsear las listas externas en la importación* — formatos heterogéneos (XLS de UN/ECE, CSV del INEI, descarga con registro en UNSPSC), licencias y disponibilidad no garantizadas; *omitir los catálogos del registro* — el cliente no podría saber que el dominio existe y que su lista es externa; *inventar un subconjunto de valores* — invención de datos normativos.
- **Consecuencias**: la plataforma no valida por sí sola esos tres dominios y debe documentar la dependencia externa. Si en el futuro se desea la carga automática, se hará como un proceso aparte que respete la misma idempotencia por hash, sin tocar el diseño de las tablas.

### 4.4 Normalización de atributos variables por catálogo

- **Contexto**: los catálogos tienen estructuras distintas: `Código;Descripción` (01, 06, 18, 20, 52, 55, 65), `Código;Nombre;GRE Aplicable` (61), `Partida arancelaria;Descripción;Código de producto SUNAT` (62), `Código;Nombre;Ubigeo` (63, 64) y `Código;Nombre;Abreviatura` (D-37). El contrato OpenAPI ya define una forma genérica: `contenido[].codigo`, `contenido[].nombre` y `contenido[].atributos` (mapa de texto).
- **Decisión**: un modelo de ítem único con `codigo`, `nombre` y `atributos` en `JSONB`. El **registro de catálogos** declara, por catálogo, qué columna es el código, cuál es el nombre y cuáles son atributos, con su tipo y obligatoriedad. La fila se normaliza con esa declaración y el `JSONB` se indexa con GIN para permitir filtrar por atributo. El campo `atributos` se expone al cliente con el nombre de la columna de origen (`GRE Aplicable`, `Ubigeo`, `Abreviatura`, `Partida arancelaria`, `Código de producto SUNAT`).
- **Alternativas descartadas**: *una tabla por catálogo* — 15 tablas, 15 repositorios y un esquema que no escala con cada catálogo nuevo; *modelo EAV (`catalogo_item_atributo` con `atributo`/`valor`)* — consultas más costosas y pérdida de tipo; *columnas físicas fijas (`atributo_1` a `atributo_4`)* — frágil ante catálogos con más columnas y sin tipo; *`JSONB` sin declaración de columnas* — se pierde la validación de que la cabecera del CSV corresponde al catálogo.
- **Consecuencias**: añadir un catálogo nuevo es una entrada en el registro, sin migración de esquema. El coste es que las consultas por atributo dependen de la indexación GIN y de la forma del JSON, y que la respuesta es genérica: el cliente debe consultar la definición de columnas del catálogo (metadato de `catalogo`) para saber qué esperar en `atributos`.

### 4.5 Resolución de la versión activa por fecha de la operación

- **Contexto**: la plataforma debe validar una GRE contra los catálogos que estaban vigentes **cuando se emitió**, y no contra el último catálogo cargado; además, una importación puede preparar una versión con vigencia futura.
- **Decisión**: la versión activa se resuelve con la condición `vigencia_desde <= :fecha AND (vigencia_hasta IS NULL OR vigencia_hasta >= :fecha)`, donde `:fecha` es la fecha de la operación (emisión o validación). Si no hay ninguna versión que cubra la fecha, el sistema devuelve un error explícito «catálogo sin versión vigente» en lugar de devolver la última disponible. La API usa la fecha del servidor.
- **Alternativas descartadas**: *«última versión importada»* — no respeta la vigencia y se rompe con cargas anticipadas; *resolver por la versión en caché sin fecha* — mismo problema; *permitir un catálogo sin versión y tratar todo como válido* — degradación silenciosa y peligrosamente permisiva.
- **Consecuencias**: el motor de validación (feature 0004) debe pasar la fecha de la operación al resolver el catálogo, y la API expone siempre la versión vigente. Obliga a que cada carga declare su `vigencia_desde` y a que las vigencias no se solapen (restricción de exclusión).

### 4.6 Listado de palabras clave como catálogo auxiliar del catálogo `20`

- **Contexto**: la hoja «Palabras Clave» del XLSX no es un catálogo numerado del Anexo N.° 8; son 69 términos con dos marcas (`Exacta`, `Dentro del texto`) que sirven para detectar el motivo de traslado a partir del texto libre de `cbc:HandlingInstructions` cuando el motivo es `13` (Otros).
- **Decisión**: se almacena en una tabla propia `catalogo_palabra_clave`, referenciando la versión del catálogo `20` a la que pertenece; no se le asigna un código del Anexo N.° 8 ni se expone bajo `/api/v1/catalogos/{codigo}`. Se consume por el puerto interno (`CatalogoVigenteProvider.palabrasClave`), que entrega las 69 entradas con sus marcas.
- **Alternativas descartadas**: *asignarle un código propio del Anexo N.° 8* — invención de una nomenclatura oficial; *tratarlo como el catálogo `20`* — mezclaría dos conjuntos de datos con naturalezas distintas; *incorporarlo al catálogo `20` como ítems* — el catálogo `20` tiene 14 filas y su número está verificado contra la fuente.
- **Consecuencias**: el listado no es consultable por API con el contrato actual (se anota en «Trabajo pendiente»). La comparación de textos no se resuelve aquí: las variantes con y sin tilde (`Importación`/`Importacion`) están listadas como entradas distintas, por lo que el motor no debe normalizar el texto (RN-25).

## 5. Modelo de datos

### 5.1 Entidades

| Entidad | Campos | Notas |
|---|---|---|
| `Catalogo` | codigo (PK), nombre, descripcion, columnas (JSONB), filasEsperadas, fuente, fuenteExterna, activo, creadoEn | Registro maestro de los 15 catálogos. `columnas` es la lista de declaraciones `{nombre, tipo, obligatorio, rol}` con `rol` ∈ {CODIGO, NOMBRE, ATRIBUTO}. |
| `CatalogoVersion` | id (PK), catalogoCodigo (FK), version (int), vigenciaDesde (date), vigenciaHasta (date, null), fuente, fechaControlCambios, hashCsv (char 64), numFilas, importadoEn, importadoPor | Inmutable. Único por `(catalogoCodigo, version)` y por `(catalogoCodigo, hashCsv)`. |
| `CatalogoItem` | id (PK), versionId (FK), codigo, nombre, atributos (JSONB), orden | Único por `(versionId, codigo)` con el código literal. |
| `CatalogoPalabraClave` | id (PK), versionId (FK → versión del catálogo `20`), palabra, exacta (bool), dentroDelTexto (bool), orden | Único por `(versionId, palabra)`. |
| `CatalogoImportacion` | id (PK), versionId (FK, null si la carga falló), catalogoCodigo, archivo, hashCsv, filasLeidas, filasInsertadas, filasRechazadas, estado, errores (JSONB), iniciadoEn, finalizadoEn | Una fila por catálogo y ejecución. `estado` ∈ {APLICADO, YA_APLICADO, RECHAZADO}. |
| `Auditoria` | sujeto, operacion, recurso, fecha, detalle | Append-only; tabla compartida con la feature 0001. |

### 5.2 Persistencia

| Tabla | Propósito | Índices |
|---|---|---|
| `catalogo` | Metadatos de los 15 catálogos | PK `codigo`; índice sobre `activo` |
| `catalogo_version` | Versiones con vigencia | PK `id`; único `(catalogo_codigo, version)`; único `(catalogo_codigo, hash_csv)`; `(catalogo_codigo, vigencia_desde DESC)`; **exclusión** `(catalogo_codigo WITH =, daterange(vigencia_desde, vigencia_hasta, '[]') WITH &&)` |
| `catalogo_item` | Ítems de cada versión | PK `id`; FK `version_id`; único `(version_id, codigo)`; `(version_id, orden)`; GIN sobre `atributos` |
| `catalogo_palabra_clave` | Palabras clave del motivo `13` | PK `id`; FK `version_id`; único `(version_id, palabra)` |
| `catalogo_importacion` | Reporte de cada carga | PK `id`; `(catalogo_codigo, iniciado_en DESC)`; `(estado)` |
| `auditoria` | Traza de operaciones | `(recurso, fecha)` (tabla existente, feature 0001) |

Notas de diseño:

- `vigencia_hasta` es `NULL` para la versión vigente; la condición de vigencia usa `COALESCE(vigencia_hasta, DATE '9999-12-31')` en los índices.
- La exclusión de rangos requiere la extensión `btree_gist`, que la migración habilita.
- El `JSONB` de atributos se almacena con las claves tal como aparecen en la cabecera del CSV (por ejemplo, `GRE Aplicable`), de modo que el cliente reconozca el vocabulario oficial.
- El catálogo `62` tiene como código la partida arancelaria y como atributos la descripción y el código de producto SUNAT; el código del ítem es la clave primaria lógica del catálogo.

### 5.3 Migraciones

| Versión | Descripción | Script |
|---|---|---|
| V3 | Extensión `btree_gist`; tablas `catalogo`, `catalogo_version`, `catalogo_item`, `catalogo_palabra_clave`, `catalogo_importacion`, con sus índices, la restricción de unicidad por hash y la de no solapamiento de vigencias. | `src/main/resources/db/migration/V3__catalogos.sql` |

Correspondencia con la feature 0001: la tarea `T-20-03` de la [feature 0001](../0001-emision-gre-remitente/tasks.md) («Migración V3: tablas de catálogos con vigencia») se materializa con esta migración y queda a cargo de esta feature; no se crean dos scripts `V3`.

## 6. Flujo de ejecución

### 6.1 Secuencia de importación

| Paso | Actor / Componente | Acción | Resultado |
|---|---|---|---|
| 1 | `CatalogoImportacionRunner` | Lee los parámetros (`--directorio`, `--vigencia-desde`, `--dry-run`) y valida el directorio | Contexto de importación disponible |
| 2 | `RegistroCatalogos` | Expone los 15 catálogos con sus columnas, filas esperadas y fuente externa | Plan de importación |
| 3 | `ImportadorCatalogosUseCase` | Localiza el CSV de cada catálogo y calcula su hash SHA-256 | Hash por catálogo |
| 4 | `CatalogoRepositorio` | Busca una versión con ese hash | Si existe: se registra `YA_APLICADO` y se pasa al siguiente catálogo (vuelve al paso 3) |
| 5 | `LectorCsvCatalogo` | Abre el fichero en UTF-8 estricto y comprueba la ausencia de BOM | Fichero legible o `CAT-003` |
| 6 | `ValidadorFormatoCsv` | Verifica separador `;`, cabecera esperada y número de columnas por fila | Formato válido o `CAT-001` / `CAT-002` |
| 7 | `LectorCsvCatalogo` | Lee las filas en streaming | Filas en memoria (máximo 120 filas por catálogo) |
| 8 | `ValidadorUnicidadCodigos` | Comprueba la unicidad del código literal | Correcto o `CAT-004` |
| 9 | `VerificadorNumeroFilas` | Compara con `filasEsperadas` | Correcto o `CAT-006` |
| 10 | `ValidadorFormatoCsv` | Comprueba celdas obligatorias no vacías y tipos declarados | Correcto o `CAT-007` / `CAT-010` |
| 11 | `NormalizadorAtributosPorCatalogo` | Proyecta cada fila a `codigo`, `nombre`, `atributos` según la declaración del catálogo | Ítems normalizados |
| 12 | `ImportadorCatalogosUseCase` | Abre la transacción del catálogo | — |
| 13 | `CatalogoVersionService` | Cierra la vigencia anterior y crea la versión nueva (`vigencia_desde`, `hash_csv`, `num_filas`) | Versión creada |
| 14 | `CatalogoRepositorio` | Inserta los ítems | Ítems insertados |
| 15 | `CatalogoPalabraClave` (catálogo `20`) | Inserta las 69 palabras con sus marcas | Palabras insertadas |
| 16 | `ImportacionRepositorio` | Persiste el reporte (leídas, insertadas, rechazadas, errores) | Reporte persistido |
| 17 | `AuditoriaAspecto` | Registra la operación con actor, versión y hash | Traza |
| 18 | `CacheCatalogoVigente` | Invalida la caché de la versión anterior | Caché coherente |
| 19 | `ImportadorCatalogosUseCase` | Confirma la transacción y emite las métricas | Importación aplicada |

Ante cualquier excepción entre los pasos 5 y 11, la transacción se revierte, el catálogo se marca `RECHAZADO` en el reporte con el detalle del error y el proceso continúa con el siguiente catálogo.

### 6.2 Secuencia de consulta

| Paso | Actor / Componente | Acción | Resultado |
|---|---|---|---|
| 1 | `SecurityFilter` | Valida el JWT y exige el scope `catalogos.leer` | Contexto de seguridad o `401` / `403` |
| 2 | `CatalogoController` | Resuelve la operación (`listar`, `consultar`, `listarItems`) | Parámetros validados (`400` si `pagina`, `tamano` o `busqueda` son inválidos) |
| 3 | `CatalogoVersionService` | Resuelve la versión activa del catálogo a la fecha del servidor | Versión o «sin versión vigente» |
| 4 | `CacheCatalogoVigente` | Devuelve los ítems de la versión si están cacheados | Ítems en memoria |
| 5 | `CatalogoRepositorio` | Si no hay caché, carga los ítems con el filtro de búsqueda y la paginación | Página de ítems |
| 6 | `MapeadorItemCatalogo` | Proyecta a `codigo`, `nombre`, `atributos` y arma la página | Respuesta `200` |
| 7 | `AuditoriaAspecto` | Registra la consulta | Traza |

### 6.3 Manejo de errores

| Situación | Excepción / código | Tratamiento | Resultado observable |
|---|---|---|---|
| Cabecera incorrecta | `FormatoCsvInvalidoException` (`CAT-001`) | Se revierte el catálogo; se registra el detalle de columnas | Reporte `RECHAZADO`; sin ítems escritos |
| Separador o columnas incorrectas | `FormatoCsvInvalidoException` (`CAT-002`) | Ídem, indicando línea y columnas | Reporte `RECHAZADO` |
| BOM o codificación no UTF-8 | `FormatoCsvInvalidoException` (`CAT-003`) | Ídem; no se «limpia» el fichero | Reporte `RECHAZADO` |
| Código duplicado | `CodigoDuplicadoException` (`CAT-004`) | Se listan los códigos duplicados | Reporte `RECHAZADO` |
| Catálogo vacío inesperado | `CatalogoVacioException` (`CAT-005`) | Se informa del catálogo vacío | Reporte `RECHAZADO` |
| Filas desviadas | `NumeroFilasInvalidoException` (`CAT-006`) | Se informa conteo real y esperado | Reporte `RECHAZADO` |
| Celda obligatoria vacía | `DatoInvalidoException` (`CAT-007`) | Se informa catálogo, línea y columna | Reporte `RECHAZADO` |
| CSV sin catálogo asociado | `CatalogoDesconocidoException` (`CAT-008`) | Se ignora con aviso | Aviso en el reporte |
| Vigencias solapadas | `VigenciaSolapadaException` (`CAT-009`) | Se informa el intervalo en conflicto | Reporte `RECHAZADO` |
| Tipo de dato inválido | `DatoInvalidoException` (`CAT-010`) | Se rechaza la fila conservando el valor original | Reporte `RECHAZADO` |
| Fallo de base de datos | `DataAccessException` | Rollback de la transacción del catálogo | Reporte `RECHAZADO`; la versión anterior sigue vigente |
| Catálogo inexistente en API | `CatalogoNoEncontradoException` | Se traduce a `404` | `404` con `application/problem+json` |
| Sin versión vigente | `VersionVigenteNoEncontradaException` | Error explícito; no se degrada a la última versión | `404` en API; error de configuración `INT-002` en el motor de validación |

## 7. Integraciones

| Sistema | Protocolo | Autenticación | Campos / operaciones usadas |
|---|---|---|---|
| PostgreSQL 16 | JDBC | usuario/contraseña en almacén de secretos | lectura de catálogos, versiones e ítems; escritura de versiones, ítems, palabras clave y reportes |
| XLSX oficial de SUNAT (`ValidacionesGREv20250421_0.xlsx`) | Proceso manual y versionado en el repositorio | No aplica (fichero público) | transcripción a CSV; hojas «Catálogos», «Palabras Clave» y «Control de Cambios» |
| INEI — ubigeos | Consulta externa manual | No aplica | lista del catálogo `13` (referenciada, no descargada) |
| UN/ECE Rec. 20 Rev. 13 | Consulta externa manual | No aplica | lista del catálogo `03` (referenciada, no descargada) |
| UNSPSC v14_0801 | Consulta externa (requiere registro) | No aplica | lista del catálogo `25` (referenciada, no descargada) |
| Motor de validación (feature 0004) | Puerto Java en proceso | Contexto de aplicación | `CatalogoVigenteProvider`: versión activa, ítems, «GRE Aplicable», ubigeo de locación, palabras clave |
| Features 0001 y 0002 (emisión) | Puerto Java en proceso | Contexto de aplicación | metadatos `@listURI` / `@schemeURI` por catálogo para el generador XML |

## 8. Seguridad

| Aspecto | Aplicación |
|---|---|
| Autenticación | OAuth 2.0 / JWT RS256 validado localmente ([ADR-0002](../../adr/0002-autenticacion-jwt.md)). |
| Autorización | Lectura con scope `catalogos.leer`; importación batch con `admin`; no hay endpoint de escritura. |
| Aislamiento por RUC | No aplica: los catálogos son datos públicos de referencia sin datos por contribuyente ([ADR-0001](../../adr/0001-seleccion-db.md)). |
| Datos sensibles | Ninguno: no hay datos personales ni información de clientes en las tablas de catálogos. |
| Auditoría | Se registra el actor, el catálogo, la versión, el hash del CSV y los conteos en `catalogo_importacion` y en `auditoria`. |
| Superficie de ataque | El proceso batch se ejecuta fuera del exposición HTTP; no se habilitan endpoints de escritura ni credenciales embebidas. |

## 9. Observabilidad

| Señal | Tipo | Etiquetas | Uso |
|---|---|---|---|
| `catalogo.importacion.iniciada` | Contador | `catalogo` | Volumen de importaciones |
| `catalogo.importacion.finalizada` | Contador | `catalogo`, `estado` (`APLICADO`, `YA_APLICADO`, `RECHAZADO`) | Resultado por catálogo |
| `catalogo.importacion.filas` | Histograma | `catalogo` | Detección de catálogos truncados |
| `catalogo.importacion.errores` | Contador | `catalogo`, `codigo_error` (`CAT-001`…`CAT-010`) | Tasa de rechazo por causa |
| `catalogo.importacion.duracion` | Histograma | `catalogo` | RNF-2 |
| `catalogo.consulta.duracion` | Histograma | `catalogo` | RNF-1 |
| `catalogo.consulta.total` | Contador | `catalogo`, `resultado` | Uso de la API |
| `catalogo.version.vigente` | Gauge | `catalogo` | Vigencia viva por catálogo |
| `catalogo.cache.hit` | Contador | `catalogo` | Eficacia de la caché del motor de validación |
| `catalogo.importacion.log` | Log estructurado | `catalogo`, `version`, `hash`, `traceId` | Trazabilidad end-to-end |

<!--CONTINUACION-->
