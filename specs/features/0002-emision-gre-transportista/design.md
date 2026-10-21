# 0002 — Diseño técnico: Emisión de la GRE Transportista

| Campo | Valor |
|---|---|
| Código | 0002 |
| Documento | Diseño técnico |
| Estado | Borrador |
| Versión | 1.0.0 |
| Fecha | 2026-10-21 |
| Especificación | [spec.md](spec.md) |
| Contrato XML | [contrato-xml-ubl.md](contrato-xml-ubl.md) |

## 1. Resumen

El diseño extiende la tubería de emisión de la [feature 0001](../0001-emision-gre-remitente/design.md) para soportar el estándar **Transportista** (tipo de documento `31`), que se diferencia del estándar **Remitente** (`09`) en el rol del emisor y en dos bloques condicionales propios: la **subcontratación** (`cac:Consignment`) y el **pagador del flete** (`cac:OriginatorCustomerParty`).

La decisión central es aislar las diferencias por tipo documental detrás de una **Strategy** (`EstrategiaGuia`), de modo que las reglas, los validadores, los mappers XML y las pruebas de la GRE-Remitente no se modifiquen al añadir la GRE-T. El motor de emisión, la reserva de correlativo, la persistencia, la firma y el envío a SUNAT se reutilizan sin cambios.

```text
        Cliente (ERP/TMS del transportista)
                    |
                    |  JSON  tipoDocumento=31  serie=V###
                    v
        +-----------------------------------+
        |  GreController / GreGrpcService   |
        +-----------------------------------+
                    |
                    v
        +-----------------------------------+
        |  GreService.emitir()              |
        |  1. validacion de esquema         |
        |  2. autorizacion por RUC (token)  |
        |  3. validacion de reglas          |
        |  4. reserva de correlativo        |
        |  5. persistencia transaccional    |
        +-----------------------------------+
                    |
                    v
        +-----------------------------------+
        |  FabricaEstrategiaGuia            |
        |  tipo=31 -> EstrategiaGreTransportista
        |  tipo=09 -> EstrategiaGreRemitente |
        +-----------------------------------+
             |                        |
             v                        v
   +------------------+   +------------------+
   | ValidacionGre    |   | ValidacionGre    |
   | Transportista    |   | Remitente        |
   | (64 items)       |   | (90 items)       |
   +------------------+   +------------------+
             |                        |
             v                        v
   +------------------+   +------------------+
   | MapperXmlGre     |   | MapperXmlGre     |
   | Transportista    |   | Remitente        |
   |  Consignacion    |   |  -               |
   |  OriginatorParty |   |                  |
   +------------------+   +------------------+
                    |
                    v
   +-----------------------------------+
   |  FirmaXml / EmpaquetadoZip        |   (feature 0005)
   |  RUC-31-VAAA-CCCCCCCC-XML / -ZIP  |
   +-----------------------------------+
                    |
                    v
   +-----------------------------------+
   |  ClienteSunat (SOAP + WS-Security)|
   |  sendPack / getStatus / getCdr    |
   +-----------------------------------+
```

## 2. Contexto y objetivos

### 2.1 Problema

El transportista genera hoy la GRE-T con herramientas externas y la carga al WS de SUNAT de forma manual. Los rechazos se concentran en reglas del estándar Transportista que no se pueden automatizar con el XML del estándar Remitente: el rol del emisor, la subcontratación, el pagador del flete y la nomenclatura del archivo con prefijo `31`.

### 2.2 Objetivos de diseño

| ID | Objetivo |
|---|---|
| OD-1 | Emitir la GRE-T reutilizando el contrato `SolicitudEmision`, sin endpoints nuevos. |
| OD-2 | No modificar el comportamiento de la GRE-Remitente (regresión cero). |
| OD-3 | Modelar la subcontratación y el pagador del flete como datos de primera clase, no como campos genéricos. |
| OD-4 | Traducir de forma determinista cada ítem de la hoja «Guía-Transportista2_0» a un tag UBL. |
| OD-5 | Garantizar la integridad referencial y la unicidad del correlativo bajo concurrencia. |
| OD-6 | Mantener la trazabilidad completa de la operación (auditoría, hash, historial de envíos). |

### 2.3 Fuera de alcance

No se implementa la GRE-Remitente (feature 0001), la comunicación de eventos del traslado (ítem 59), la anulación, la consulta a padrones de SUNAT ni la firma: la firma y el envío permanecen en la feature 0005.

## 3. Restricciones y supuestos

### 3.1 Restricciones

| ID | Restricción | Origen |
|---|---|---|
| RE-1 | El payload es el mismo `SolicitudEmision` de [schema.json](../../contracts/schema.json), con `tipoDocumento: "31"` y serie `V###`. | Contrato de API |
| RE-2 | El XML debe ser UBL 2.1 `DespatchAdvice` con `DespatchAdviceTypeCode` = `31` y `CustomizationID` = `2.0`. | [contrato-xml-ubl.md](contrato-xml-ubl.md), ítems 1, 2 y 6 |
| RE-3 | El nombre del archivo es `RUC-31-VAAA-CCCCCCCC-XML` y del ZIP `RUC-31-VAAA-CCCCCCCC-ZIP`. | Ítems 3, 15 y 22 |
| RE-4 | Los 64 ítems de la hoja «Guía-Transportista2_0» son la fuente de verdad de las validaciones. | Contrato XML |
| RE-5 | La firma y el envío a SUNAT son responsabilidades de la feature 0005. | Feature 0005 |
| RE-6 | Los catálogos se consumen desde la feature 0003, con versión y vigencia. | Feature 0003 |
| RE-7 | El RUC efectivo se toma del token, nunca del cuerpo de la petición ([ADR-0002](../../adr/0002-autenticacion-jwt.md)). | ADR-0002 |
| RE-8 | La persistencia usa PostgreSQL ([ADR-0001](../../adr/0001-seleccion-db.md)). | ADR-0001 |

### 3.2 Supuestos

| ID | Supuesto |
|---|---|
| SU-1 | El certificado digital del transportista ya está cargado y vigente (feature 0005). |
| SU-2 | El ERP del transportista serializa las cantidades con 10 decimales. |
| SU-3 | Los padrones de SUNAT (RUC, DNI, placas, licencias) no se consultan localmente; los ítems que los exigen se reportan como observación o error de SUNAT. |
| SU-4 | La plataforma no verifica la existencia de la GRE-Remitente agrupada; la comprobación la realiza SUNAT y se refleja en los códigos `3433`, `3434` y `2536`. |

## 4. Decisiones de diseño

### DD-1 — Strategy por tipo documental

**Estado:** Aceptada

**Contexto.** El estándar Remitente tiene 90 ítems y el Transportista 64. Comparten cabecera, remitente, destinatario, vehicles, conductores, documentos relacionados y detalle de bienes, pero divergen en el emisor, en la consigna y en el pagador del flete. La feature 0001 implementó las reglas como un validador monolítico.

**Decisión.** Se introduce la interfaz `EstrategiaGuia` y dos implementaciones, `EstrategiaGreRemitente` (feature 0001, sin cambios funcionales) y `EstrategiaGreTransportista` (nueva). La selección se hace por `tipoDocumento` en `FabricaEstrategiaGuia`.

```java
public interface EstrategiaGuia {
    TipoDocumento tipoDocumento();
    List<ReglaValidacion> reglas();
    XmlGuia generarXml(SolicitudEmision solicitud, ContextoEmision contexto);
    void validarAntesDePersistir(SolicitudEmision solicitud, List<Violacion> violaciones);
}
```

**Alternativas consideradas.**

| Alternativa | Motivo del descarte |
|---|---|
| Un único validador con `if (tipo == 31)` en cada regla | Condicionales dispersos; con 90 + 64 reglas el costo de mantenimiento es prohibitivo y viola RNF-11. |
| Copiar la feature 0001 completa para el tipo `31` | Duplicación de ~90 % del código, con divergencia inevitable entre ambas bases. |
| Motor declarativo de reglas por JSON/YAML | La expresión de reglas con dependencias entre ítems (p. ej. ítem 51 depende del ítem 20) requiere un lenguaje de expresiones; el coste supera al beneficio para dos estándares. |

**Consecuencias.** Positivas: RNF-11 se cumple por construcción; las pruebas de la GRE-Remitente no se tocan. Negativas: cada estándar nuevo exige una implementación nueva de la estrategia; se asume un número reducido de estándares (2).

### DD-2 — Modelo de dominio explícito para consigna y pagador del flete

**Estado:** Aceptada

**Contexto.** Los indicadores `SUNAT_Envio_*` son cadenas de texto libre en `cbc:SpecialInstructions`. Un modelo genérico «lista de indicadores» obligaría a reinterpretar las cadenas en cada punto de uso y no impediría, por sí solo, combinaciones inválidas (por ejemplo, subcontratación sin subcontratador).

**Decisión.** Se modelan dos value objects con invariantes:

```java
public record IndicadorSubcontratacion(
    boolean subcontratado,
    Subcontratador subcontratador
) {
    public void validar() {
        if (subcontratado && (subcontratador == null || subcontratador.ruc() == null)) {
            throw new ViolacionReglaNegocio(Item.VALIDACION_60, CodigoRetorno.NUMERO_DOCUMENTO_SUBCONTRATADOR_AUSENTE);
        }
    }
}

public record IndicadorPagadorFlete(RolPagadorFlete rol, PagadorFlete tercero) {
    public void validar() {
        if (rol == RolPagadorFlete.TERCERO && (tercero == null || tercero.numeroDocumento() == null)) {
            throw new ViolacionReglaNegocio(Item.VALIDACION_62, CodigoRetorno.PAGADOR_FLETE_SIN_DATOS);
        }
        if (rol != RolPagadorFlete.TERCERO && tercero != null) {
            throw new ViolacionReglaNegocio(Item.VALIDACION_62, CodigoRetorno.PAGADOR_FLETE_SIN_DATOS);
        }
    }
}

public enum RolPagadorFlete {
    REMITENTE("SUNAT_Envio_IndicadorPagadorFlete_Remitente"),
    SUBCONTRATADOR("SUNAT_Envio_IndicadorPagadorFlete_Subcontratador"),
    TERCERO("SUNAT_Envio_IndicadorPagadorFlete_Tercero");
}
```

**Consecuencias.** La invariante «`GreConsignacion` existe ⇔ se declara el indicador de subcontratación» queda garantizada por el código y por una restricción `CHECK` en base de datos. El coste es que el payload JSON tiene dos formas de expressing la misma información (indicador + bloque de datos); el mapper resuelve la precedencia en favor del bloque de datos y exige coherencia entre ambos.

### DD-3 — Mapeo por tabla de correspondencia JSON → XML

**Estado:** Aceptada

**Contexto.** El contrato XML define, ítem a ítem, el tag UBL, la obligatoriedad y el formato de cada dato. Codificar cada ítem como código imperativo en el mapper es propenso a omisiones.

**Decisión.** El mapper de la estrategia Transportista se apoya en una tabla declarativa `CorrespondenciaItem` (en memoria, construida desde el contrato XML) y en un visitor por tipo de dato. Cada entrada declara el ítem de validación, el XPath destino, el extractor del valor desde el JSON y el formateador.

```java
public record CorrespondenciaItem(
    int item,
    String xpathDestino,
    Function<SolicitudEmision, Object> extractor,
    Function<Object, String> formateador,
    boolean obligatorio
) {}
```

Las particiones opcionales (vehículos secundarios, conductores secundarios, documentos relacionados, detalle de bienes) se resuelven con generadores de listas que se saltan por completo cuando la colección de origen está vacía, de modo que el XML no contenga nodos vacíos.

**Consecuencias.** Cada ítem de la hoja tiene una prueba de correspondencia unitaria; el golden file del XML verifica el conjunto completo.

### DD-4 — Correlativo con reserva transaccional

**Estado:** Aceptada

**Contexto.** La numeración correlativa es secuencial y no admite huecos ni repeticiones. Varias instancias de la aplicación emiten en paralelo.

**Decisión.** La reserva se hace con `UPDATE ... RETURNING` sobre `serie_correlativo` dentro de la misma transacción de la emisión, con bloqueo de fila implícito:

```sql
UPDATE serie_correlativo
   SET ultimo_correlativo = ultimo_correlativo + 1,
       actualizado_en = now()
 WHERE ruc_emisor = :ruc
   AND tipo_documento = :tipoDocumento
   AND serie = :serie
   AND estado = 'ACTIVA'
RETURNING ultimo_correlativo;
```

El `INSERT` del documento con la restricción `UNIQUE (ruc_emisor, tipo_documento, serie, correlativo)` actúa como segunda barrier de seguridad. Si la transacción falla, el contador se revierte junto con el documento, por lo que no se producen huecos.

**Alternativas consideradas.** Secuencia por serie (no garantiza continuidad si hay transacciones concurrentes) y bloqueo pesimista explícito (`SELECT ... FOR UPDATE`) con más bloqueos. Se descarta la secuencia porque el requerimiento es de series consecutivas por RUC.

### DD-5 — Versionado del XML por estrategia

**Estado:** Aceptada

**Decisión.** El XML generado se versiona con el hash SHA-256 y con un identificador de versión de plantilla (`versionPlantillaXml`) que corresponde a la revisión del contrato XML y de los catálogos empleados. Si el contrato XML cambia, cambia la versión de plantilla y se conserva el XML ya emitido tal como fue aceptado por SUNAT.

### DD-6 — Envío asíncrono con reintentos clasificados

**Estado:** Aceptada

**Contexto.** Los errores del WS de SUNAT se dividen en `ERROR` (rechazo del documento), `OBSERV` (aceptado con observaciones) y fallos de transporte. Sólo estos últimos son reintentables.

**Decisión.** El envío se ejecuta de forma asíncrona con un planificador, con backoff exponencial y jitter. Un `ERROR` de SUNAT o del propio documento (por ejemplo `3374` o `1059`) marca la GRE-T como `RECHAZADA` y no reintenta; un `OBSERV` la marca `ACEPTADA_CON_OBSERVACIONES`; un fallo de transporte agota los reintentos y deja la GRE-T en `PENDIENTE_REINTENTO`. La clasificación se toma de [mapeo-errores.md](../0004-validacion-documento/mapeo-errores.md).

### DD-7 — Aislamiento multi-tenant por RUC

**Estado:** Aceptada

**Decisión.** El RUC efectivo se toma del claim `ruc` del token. Todas las consultas de lectura filtran por `ruc_emisor` y la base de datos aplica RLS por `current_setting('app.ruc')`. No se confía en el RUC del cuerpo de la petición, que sólo se usa para comparar y detectar discrepancias.

## 5. Arquitectura de componentes

| Componente | Paquete | Responsabilidad |
|---|---|---|
| `GreController` | `com.examen.gre.interfaces.rest` | Endpoints REST de emisión, consulta y envío. |
| `GreGrpcService` | `com.examen.gre.interfaces.grpc` | Superficie gRPC equivalente. |
| `GreService` | `com.examen.gre.application` | Orquesta el caso de uso de emisión. |
| `ValidacionService` | `com.examen.gre.application.validation` | Resuelve la estrategia y ejecuta sus reglas. |
| `FabricaEstrategiaGuia` | `com.examen.gre.domain.strategy` | Resuelve la estrategia por `tipoDocumento`. |
| `EstrategiaGreTransportista` | `com.examen.gre.domain.strategy.transportista` | Reglas, mapper y semántica del tipo `31`. |
| `ValidacionGreTransportista` | `com.examen.gre.domain.strategy.transportista.validation` | Implementa los ítems 1..64 de la hoja «Guía-Transportista2_0». |
| `MapperXmlGreTransportista` | `com.examen.gre.domain.strategy.transportista.xml` | Traduce el JSON al XML UBL 2.1. |
| `MapperConsignacion` | `com.examen.gre.domain.strategy.transportista.xml` | Genera `cac:Consignment` y `cac:LogisticsOperatorParty`. |
| `MapperPagadorFlete` | `com.examen.gre.domain.strategy.transportista.xml` | Genera `cac:OriginatorCustomerParty` y el indicador correspondiente. |
| `GeneradorXmlGuia` | `com.examen.gre.infrastructure.xml` | Aplica Marshaller/JAXB y serializa el documento UBL. |
| `SerieCorrelativoRepository` | `com.examen.gre.infrastructure.persistence` | Reserva transaccional del correlativo. |
| `GreRepository` | `com.examen.gre.infrastructure.persistence` | Persistencia del documento, ítems y bloques condicionales. |
| `FirmadorXml` / `EmpaquetadorZip` | `com.examen.gre.infrastructure.firma` | Firma XMLDSig y ZIP con la nomenclatura `31` (feature 0005). |
| `ClienteSunat` | `com.examen.gre.infrastructure.sunat` | Cliente SOAP con WS-Security (feature 0005). |
| `AuditoriaAspecto` | `com.examen.gre.infrastructure.auditoria` | Registro de auditoría por operación. |

### 5.1 Secuencia de emisión

```text
ERP        Controller        GreService        FabricaEstrategia   MapperXml      Firma      Sunat
 |             |                 |                   |                 |           |         |
 |--POST------->|                 |                   |                 |           |         |
 |             |--emitir()------->|                   |                 |           |         |
 |             |                 |--validar esquema--|                 |           |         |
 |             |                 |--autorizar RUC (token)------------>|           |         |
 |             |                 |--obtener estrategia------------->|           |         |
 |             |                 |--validar reglas----------------->|           |         |
 |             |                 |--reservar correlativo                                        |
 |             |                 |--perseguir (transacción)                                  |
 |             |                 |                   |                 |           |         |
 |             |                 |--generar XML---->|                 |           |         |
 |             |                 |                   |---firmar------->|           |         |
 |             |                 |                   |---ZIP RUC-31-VAAA-CCCCCCCC-ZIP--->|        |
 |             |                 |--persistir + hash |                 |           |         |
 |<--201 ID, XML, hash, estado--|                   |                 |           |         |
 |             |                 |--enviar()------------------------------------------------------>|
 |             |                 |                   |                 |           |<--respuesta|
 |             |                 |--perseguir envío, estado, ticket, observaciones                |
```

## 6. Diseño de datos

### 6.1 Modelo persistente

```sql
CREATE TABLE gre_documento (
    id                  BIGSERIAL PRIMARY KEY,
    ruc_emisor          VARCHAR(11)   NOT NULL,
    tipo_documento      VARCHAR(2)    NOT NULL,
    serie               VARCHAR(4)    NOT NULL,
    correlativo         INTEGER       NOT NULL,
    identificador       VARCHAR(13)   NOT NULL,
    estado              VARCHAR(24)   NOT NULL,
    fecha_emision       DATE          NOT NULL,
    hora_emision        TIME          NOT NULL,
    version_plantilla   VARCHAR(12)   NOT NULL,
    hash_xml            CHAR(64)      NOT NULL,
    xml_firmado         BYTEA         NOT NULL,
    nombre_archivo_xml  VARCHAR(40)   NOT NULL,
    nombre_archivo_zip  VARCHAR(40)   NOT NULL,
    solicitud_json      JSONB         NOT NULL,
    creado_en           TIMESTAMPTZ   NOT NULL DEFAULT now(),
    CONSTRAINT uq_gre_documento UNIQUE (ruc_emisor, tipo_documento, serie, correlativo),
    CONSTRAINT ck_gre_tipo CHECK (tipo_documento IN ('09', '31'))
);

CREATE TABLE gre_item (
    id                    BIGSERIAL PRIMARY KEY,
    gre_documento_id      BIGINT        NOT NULL REFERENCES gre_documento(id) ON DELETE CASCADE,
    orden                 INTEGER       NOT NULL,
    descripcion           VARCHAR(500)  NOT NULL,
    cantidad              NUMERIC(12,10) NOT NULL,
    unidad_medida         VARCHAR(5)    NOT NULL,
    codigo_producto       VARCHAR(20),
    gtin                  VARCHAR(14),
    CONSTRAINT uq_gre_item UNIQUE (gre_documento_id, orden)
);

CREATE TABLE gre_documento_relacionado (
    id                 BIGSERIAL PRIMARY KEY,
    gre_documento_id   BIGINT        NOT NULL REFERENCES gre_documento(id) ON DELETE CASCADE,
    tipo               VARCHAR(2)    NOT NULL,
    descripcion        VARCHAR(60),
    numero             VARCHAR(20)   NOT NULL,
    ruc_emisor_rel     VARCHAR(11),
    CONSTRAINT uq_gre_doc_rel UNIQUE (gre_documento_id, tipo, numero)
);

CREATE TABLE gre_consignacion (
    gre_documento_id      BIGINT PRIMARY KEY REFERENCES gre_documento(id) ON DELETE CASCADE,
    ruc_subcontratador    VARCHAR(11) NOT NULL,
    nombre_subcontratador VARCHAR(200) NOT NULL
);

CREATE TABLE gre_pagador_flete (
    gre_documento_id         BIGINT PRIMARY KEY REFERENCES gre_documento(id) ON DELETE CASCADE,
    rol                      VARCHAR(14) NOT NULL,
    tipo_documento_identidad VARCHAR(1)  NOT NULL,
    numero_documento         VARCHAR(11) NOT NULL,
    nombre                   VARCHAR(200) NOT NULL,
    CONSTRAINT ck_pagador_rol CHECK (rol = 'TERCERO')
);

CREATE TABLE gre_envio (
    id                 BIGSERIAL PRIMARY KEY,
    gre_documento_id   BIGINT      NOT NULL REFERENCES gre_documento(id) ON DELETE CASCADE,
    intento            INTEGER     NOT NULL,
    estado             VARCHAR(16) NOT NULL,
    codigo_retorno     VARCHAR(8),
    descripcion        VARCHAR(500),
    ticket             VARCHAR(30),
    request_xml        TEXT,
    response_xml       TEXT,
    fecha              TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT uq_gre_envio UNIQUE (gre_documento_id, intento)
);

CREATE TABLE gre_observacion (
    id               BIGSERIAL PRIMARY KEY,
    gre_documento_id BIGINT       NOT NULL REFERENCES gre_documento(id) ON DELETE CASCADE,
    item             INTEGER      NOT NULL,
    tag_ubl          VARCHAR(120),
    codigo_retorno   VARCHAR(8)   NOT NULL,
    tipo_retorno     VARCHAR(8)   NOT NULL,
    mensaje          VARCHAR(500) NOT NULL
);

CREATE TABLE serie_correlativo (
    ruc_emisor       VARCHAR(11) NOT NULL,
    tipo_documento   VARCHAR(2)  NOT NULL,
    serie            VARCHAR(4)  NOT NULL,
    ultimo_correlativo INTEGER  NOT NULL DEFAULT 0,
    estado           VARCHAR(12) NOT NULL DEFAULT 'ACTIVA',
    PRIMARY KEY (ruc_emisor, tipo_documento, serie)
);
```

### 6.2 Mapeo JSON → tablas

| Bloque JSON | Tabla | Nota |
|---|---|---|
| `rucEmisor`, `tipoDocumento`, `serie`, `fechaEmision`, `horaEmision` | `gre_documento` | El RUC efectivo se sobreescribe con el del token. |
| `identificador` | `gre_documento.identificador` | Debe coincidir con `V###-########`. |
| `bienes` / `detalle` | `gre_item` | Lista ordenada; la unidad y la cantidad son obligatorias salvo el caso del ítem 51. |
| `documentosRelacionados` | `gre_documento_relacionado` | El par (tipo, número) no se repite (`3340`). |
| `consignacion` (subcontratación) | `gre_consignacion` | 0..1; exige el indicador de subcontratación. |
| `pagadorFlete` (tercero) | `gre_pagador_flete` | 0..1; sólo con el indicador `_Tercero`. |
| `vehiculos`, `conductores`, `puntos`, `documentosTransporte` | `gre_item` + `gre_propiedad_item` | Se modelan como ítems con propiedades del Catálogo 55. |
| `solicitud` completa | `gre_documento.solicitud_json` | Permite reconstruir el XML sin depender de la tabla de ítems. |

### 6.3 Índices y RLS

| Objeto | Definición | Propósito |
|---|---|---|
| `idx_gre_documento_ruc_serie` | `(ruc_emisor, tipo_documento, serie, correlativo DESC)` | Consulta de series y de la última GRE emitida. |
| `idx_gre_envio_documento` | `(gre_documento_id, intento DESC)` | Historial de envíos. |
| `idx_gre_observacion_documento` | `(gre_documento_id)` | Listado de observaciones. |
| RLS | `USING (ruc_emisor = current_setting('app.ruc')::varchar)` | Aislamiento por tenant. |

## 7. Diseño del XML UBL

### 7.1 Estructura del documento

```xml
<DespatchAdvice xmlns="urn:oasis:names:specification:ubl:schema:xsd:DespatchAdvice-2"
                xmlns:cac="urn:oasis:names:specification:ubl:schema:xsd:CommonAggregateComponents-2"
                xmlns:cbc="urn:oasis:names:specification:ubl:schema:xsd:CommonBasicComponents-2">
  <cbc:UBLVersionID>2.1</cbc:UBLVersionID>
  <cbc:CustomizationID>2.0</cbc:CustomizationID>
  <cbc:ID>V001-00000001</cbc:ID>
  <cbc:IssueDate>2026-10-21</cbc:IssueDate>
  <cbc:IssueTime>10:15:30</cbc:IssueTime>
  <cbc:DespatchAdviceTypeCode>31</cbc:DespatchAdviceTypeCode>
  <cac:DespatchSupplierParty>          <!-- transportista: emisor (items 8-11) -->
    <cac:Party>
      <cbc:ID schemeID="6">20123456789</cbc:ID>
      <cac:PartyLegalEntity><cbc:RegistrationName>TRANSPORTES SAC</cbc:RegistrationName></cac:PartyLegalEntity>
    </cac:Party>
  </cac:DespatchSupplierParty>
  <cac:DeliveryCustomerParty>          <!-- destinatario (items 18-19) -->
    <cac:Party>...</cac:Party>
  </cac:DeliveryCustomerParty>
  <cac:Shipment>
    <cbc:ID>SUNAT_Envio</cbc:ID>
    <cbc:SpecialInstructions>SUNAT_Envio_IndicadorTrasporteSubcontratado</cbc:SpecialInstructions>
    <cac:ShipmentStage>
      <cac:CarrierParty>                <!-- datos MTC del transportista (items 10-11) -->
        <cac:PartyLegalEntity><cbc:CompanyID>AB0001</cbc:CompanyID></cac:PartyLegalEntity>
        <cac:AgentParty>
          <cac:PartyLegalEntity><cbc:CompanyID schemeID="01">MTC</cbc:CompanyID></cac:PartyLegalEntity>
        </cac:AgentParty>
      </cac:CarrierParty>
    </cac:ShipmentStage>
    <cac:Consignment>                   <!-- subcontratacion (items 60-61) -->
      <cbc:ID>SUNAT_Envio</cbc:ID>
      <cac:LogisticsOperatorParty>
        <cac:Party>
          <cbc:ID schemeID="6">20987654321</cbc:ID>
          <cac:PartyLegalEntity><cbc:RegistrationName>SUBCONTRATISTA SAC</cbc:RegistrationName></cac:PartyLegalEntity>
        </cac:Party>
      </cac:LogisticsOperatorParty>
    </cac:Consignment>
    <cac:OriginatorCustomerParty>       <!-- pagador del flete, solo si es tercero (items 62-63) -->
      <cac:Party>
        <cbc:ID schemeID="6">20654321098</cbc:ID>
        <cac:PartyLegalEntity><cbc:RegistrationName>CLIENTE SAC</cbc:RegistrationName></cac:PartyLegalEntity>
      </cac:Party>
    </cac:OriginatorCustomerParty>
    <cac:ShipmentStage>
      <cac:CarrierParty>
        <cac:DespatchParty>            <!-- remitente (items 16-17) -->
          <cac:Party>...</cac:Party>
        </cac:DespatchParty>
      </cac:CarrierParty>
    </cac:ShipmentStage>
  </cac:Shipment>
</DespatchAdvice>
```

El ejemplo anterior es ilustrativo y muestra la posición relativa de los nodos; la posición exacta de cada `cac:ShipmentStage` dentro de `cac:Shipment` y la cardinalidad de `cac:Consignment` y `cac:OriginatorCustomerParty` se fijan al contrastar con el XSD oficial (ver trabajo pendiente TP-01).

### 7.2 Tabla de correspondencia de los nodos propios del Transportista

| Nodo UBL | Contenido | Ítem | Reglas |
|---|---|---|---|
| `cbc:DespatchAdviceTypeCode` | `31` | 6 | `1050`, `1051` |
| `cac:DespatchSupplierParty` | Transportista (emisor) | 8 | `2511`, `2678`, `3349`, `3350`, `3439`, `4255`–`4257` |
| `cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:PartyLegalEntity/cbc:CompanyID` | Registro MTC del transportista | 10 | `4391`–`4393` |
| `cac:Shipment/cac:ShipmentStage/cac:CarrierParty/cac:AgentParty/.../cbc:CompanyID` | Entidad autorizadora (D-37) | 11 | `4394`–`4397` |
| `cac:Shipment/cac:Consignment/cbc:ID` | `SUNAT_Envio` | 60 | — |
| `cac:Shipment/cac:Consignment/cac:LogisticsOperatorParty` | Subcontratador | 60, 61 | `3390`, `3391`, `3389`, `4424`–`4427` |
| `cac:Shipment/cac:OriginatorCustomerParty` | Pagador del flete (tercero) | 62, 63 | `3399`, `3400`, `3401`, `3402`, `4401`, `4402`, `4370` |
| `cac:Shipment/cac:Delivery/cac:Despatch/cac:DespatchParty` | Remitente | 16, 17 | `2560`, `3383`–`3387`, `4420`–`4422` |
| `cac:Shipment/cbc:SpecialInstructions` | Indicadores `SUNAT_Envio_*` | 53–58 | `3344`, `3388`, `4388` |

### 7.3 Reglas de formación

| Regla | Detalle |
|---|---|
| FM-1 | Los indicadores se escriben en `cbc:SpecialInstructions`, uno por nodo, sin duplicar ni repetir el mismo literal (`3344`). |
| FM-2 | Los valores admitidos como indicador son exclusivamente los literales `SUNAT_Envio_*` del estándar vigente; cualquier otro valor es rechazado (`3388`). |
| FM-3 | `cac:Consignment` se emite sólo si el indicador de subcontratación está presente; `cac:OriginatorCustomerParty` sólo si el indicador de pagador `_Tercero` está presente. |
| FM-4 | El literal del indicador de subcontratación se toma de una constante única (`Indicadores.INDICADOR_TRANSPORTE_SUBCONTRATADO`), nunca de literales dispersos, para que la corrección del literal pendiente (TP-01) sea un cambio de una línea. |
| FM-5 | El `cbc:ID` del documento y el nombre de archivo se construyen con la misma fuente (`ruc`, `serie`, `correlativo`), garantizando la correspondencia exigida por `1034`, `1035` y `1036`. |
| FM-6 | Ningún `cac:SpecialInstructions` contiene el tipo de evento; el ítem 59 (`3374`) se verifica por aserción en el validador. |
| FM-7 | Los importes, si los hubiera, se serializan con 2 decimales; las cantidades con 10 (`n(12,10)`). |
| FM-8 | Los caracteres no ASCII se escapan según la declaración XML `UTF-8`; los saltos de línea se normalizan a `&#xA;`. |

## 8. Flujos de error y reintento

| Situación | Detección | Tratamiento | Estado final |
|---|---|---|---|
| Violación de esquema JSON | Validador de esquema | `400`, sin persistir | — |
| RUC no autorizado | Claim `ruc` del token | `403 RUC_NO_AUTORIZADO` | — |
| Regla de negocio incumplida | `ValidacionGreTransportista` | `422` con la lista de violaciones | — |
| Serie inexistente o anulada | `SerieCorrelativoRepository` | `400`/`409` | — |
| Conflicto de idempotencia | Almacén de idempotencia | `409 IDEMPOTENCIA_CONFLICTO` | — |
| Certificado no vigente | `FirmadorXml` | Se conserva el documento en `VALIDADA` y se reintenta la firma | `VALIDADA` |
| SUNAT responde `ERROR` | `ClienteSunat` | Se persiste el código; no se reintenta | `RECHAZADA` |
| SUNAT responde `OBSERV` | `ClienteSunat` | Se persisten las observaciones | `ACEPTADA_CON_OBSERVACIONES` |
| Fallo de transporte o 5xx | `ClienteSunat` | Reintento con backoff exponencial (1 s, 5 s, 30 s, 2 min, 10 min, máximo 6 intentos) | `PENDIENTE_REINTENTO` |
| XML no parseable por SUNAT (`0306`) | `ClienteSunat` | Se regenera desde `solicitud_json` y se reintenta una vez | según el reintento |

## 9. Seguridad

| Aspecto | Medida |
|---|---|
| Autenticación | JWT firmado por el emisor OAuth 2.0; [ADR-0002](../../adr/0002-autenticacion-jwt.md). |
| Autorización | RLS por `app.ruc` y filtro `ruc_emisor` en todas las consultas. El RUC del cuerpo se compara con el del token y la discrepancia se rechaza. |
| Firma | XMLDSig con el certificado del transportista; la firma se valida antes de cada envío. |
| WS-Security | `UsernameToken` con usuario secundario del transportista y clave del WS, más el certificado del contribuyente; la contraseña viaja cifrada por TLS 1.2 y nunca se persiste en claro. |
| Transporte | TLS 1.2 con los WS de SUNAT. |
| Datos sensibles | Los DNI de conductores y de los contactos no se exponen en las respuestas de consulta; se enmascaran (`****1234`). |
| Auditoría | Registro append-only de toda la operación con sujeto, RUC, recurso y hash. |
| Secretos | Gestión por variables de entorno o gestor de secretos; sin secretos en el repositorio. |

## 10. Observabilidad

| Elemento | Detalle |
|---|---|
| Logs | JSON estructurado con `traceId`, `rucEmisor`, `tipoDocumento`, `serie`, `correlativo`, `item` y `codigoRetorno` en cada violación. |
| Métricas | `gre_transportista_emisiones_total{tipo,resultado}`, `gre_transportista_violaciones_total{item,codigo}`, `gre_transportista_envio_total{estado}`, `gre_transportista_envio_latencia_segundos`. |
| Trazas | Span por etapa: `validar`, `reservar_correlativo`, `persistir`, `generar_xml`, `firmar`, `empaquetar`, `enviar`. |
| Alertas | Tasa de `RECHAZADA` > 5 % en 1 hora; `PENDIENTE_REINTENTO` > 10 documentos; tiempo de espera de firma > 15 min. |
| Panel | Serie y correlativo último emitido por RUC, distribution de códigos de retorno por ítem. |

## 11. Estrategia de pruebas

| Nivel | Alcance | Herramientas |
|---|---|---|
| Unitario | Cada ítem 1..64 como prueba de una regla; mappers XML; value objects de consigna y pagador. | JUnit 5, AssertJ |
| Property-based | Formato del identificador, unicidad de indicadores, correspondencia JSON ↔ XML. | jqwik |
| Integración | Persistencia, reserva de correlativo bajo concurrencia, RLS, idempotencia. | Testcontainers (PostgreSQL) |
| Contrato | XML generado validado contra el XSD oficial; payload validado contra [schema.json](../../contracts/schema.json). | XSD UBL 2.1, JSON Schema |
| End-to-end | Emisión → firma → envío simulado → estado y ticket. | WireMock para el WS de SUNAT |
| Regresión | Suite completa de la feature 0001 sin cambios. | JUnit 5 |

Criterios de cobertura: 100 % de las líneas de `ValidacionGreTransportista` y `MapperXmlGreTransportista`, con una prueba por cada código de retorno de la hoja «Guía-Transportista2_0».

## 12. Despliegue

| Fase | Contenido | Reversión |
|---|---|---|
| Pre | Migraciones de esquema (tablas nuevas, RLS, índices), carga de catálogos de la feature 0003 y despliegue del validador `31` con la estrategia aún sin activar. | Eliminar las migraciones y desplegar la versión anterior. |
| Post | Activación de `EstrategiaGreTransportista` mediante una bandera de configuración `gre.tipos-documento=09,31`. | Desactivar la bandera; la GRE-Remitente no se ve afectada. |
| Piloto | Habilitación por RUC para un transportista piloto, con seguimiento diario de los códigos de retorno. | Revocar la habilitación del piloto. |

Plan de reversión: la feature se activa por configuración y por RUC; el esquema es retrocompatible porque todas las tablas y columnas nuevas son aditivas. No hay migraciones destructivas.

## 13. Cumplimiento normativo

| Exigencia | Implementación | Ítem |
|---|---|---|
| UBL 2.1 con `CustomizationID` 2.0 | `GeneradorXmlGuia` + `MapperXmlGreTransportista` | 1, 2 |
| Tipo de documento `31` | `EstrategiaGreTransportista.tipoDocumento()` | 6 |
| Nomenclatura `RUC-31-VAAA-CCCCCCCC` | `GeneradorXmlGuia` construye el nombre desde `ruc`, `tipoDocumento`, `serie` y `correlativo` | 3, 15, 22 |
| Firma digital obligatoria | `FirmadorXml` (feature 0005) | 64 |
| Reglas de la hoja «Guía-Transportista2_0» | `ValidacionGreTransportista` | 1..64 |
| Codificación de la respuesta del WS | Cada código de retorno y su mensaje se registran en `gre_observacion` con la severidad interna `ERROR` o `WARN` según el mapeo de errores vigente. | [mapeo-errores.md](../0004-validacion-documento/mapeo-errores.md) |

## 14. Trazabilidad con la especificación

| Requisito | Componente | Artefacto | Prueba |
|---|---|---|---|
| RF-1 | `GreController`, `GreService` | `SolicitudEmision` | `SolicitudEmisionTransportistaTest` |
| RF-2 | `SerieCorrelativoRepository` | `serie_correlativo` | `EmisionConcurrenteTransportistaIT` |
| RF-3 | `ValidacionGreTransportista` | 64 reglas | `ValidacionTransportistaTest` |
| RF-4 | `MapperXmlGreTransportista`, `GeneradorXmlGuia` | XML UBL 2.1 | `XmlTransportistaTest` (golden files) |
| RF-5 | `FirmadorXml`, `EmpaquetadorZip` | ZIP `31` | `FirmaYZipTransportistaTest` |
| RF-6 | `GreRepository`, `GreController` | `gre_documento` | `GreControllerTransportistaIT` |
| RF-7 | `GreRepository` | `gre_envio` | `EnvioTransportistaIT` |
| RF-8 | `ClienteSunat`, planificador | Backoff | `EnvioResilienteTransportistaIT` |
| RF-9 | `GreGrpcService` | Contrato gRPC | `GreGrpcServiceTransportistaIT` |
| RF-10 | `AuditoriaAspecto` | `auditoria` | `AuditoriaTransportistaTest` |
| RF-11 | `MapperConsignacion`, `ValidacionGreTransportista` | `gre_consignacion` | `SubcontratacionTest` |
| RF-12 | `MapperPagadorFlete` | `gre_pagador_flete` | `PagadorFleteTest` |
| RF-13 | `ValidacionGreTransportista` (ítems 13–15) | `gre_documento_relacionado` | `AgrupacionGuiasTest` |
| RF-14 | `SerieService` | `serie_correlativo` | `SeriePrefijoTest` |
| RF-15 | `GreRepository` | `gre_observacion` | `ObservacionesTransportistaIT` |

## 15. Trabajo pendiente

| ID | Descripción | Impacto | Responsable |
|---|---|---|---|
| TP-01 | Confirmar el literal del indicador de subcontratación: `SUNAT_Envio_IndicadorTrasporteSubcontratado` (contrato XML, ítems 54 y 57) o `SUNAT_Envio_IndicadorTransporteSubcontratado` ([schema.json](../../contracts/schema.json)). Unificar ambos contratos. | Alto | Negocio + Integración |
| TP-02 | Confirmar la posición y cardinalidad exacta de `cac:ShipmentStage`, `cac:Consignment` y `cac:OriginatorCustomerParty` dentro de `cac:Shipment` contra el XSD oficial (el contrato XML no es explícito al respecto). | Alto | Integración |
| TP-03 | Confirmar los URI y prefijos de namespace (`urn:oasis:...:DespatchAdvice-2` y los agregados) que exige el WS de recepción. | Alto | Integración |
| TP-04 | Revisar `schema.json`: exige incondicionalmente `envio.motivo` y `envio.modalidad`, campos que no existen en el estándar Transportista; debe pasar a `required` condicional por `tipoDocumento`. | Alto | Backend + Negocio |
| TP-05 | Revisar `schema.json`: exige cantidad y unidad en todos los ítems, pero el Transportista admite la anotación opcional del ítem 51 (`3458`, `4434`). | Medio | Backend |
| TP-06 | Verificar si `SUNAT_Envio_IndicadorTrasladoVehiculoM1L` aplica a la GRE-Transportista; figura como **[NO VERIFICADO]** en el contrato XML. | Medio | Negocio |
| TP-07 | Modelar en el esquema el registro MTC y la entidad autorizadora del transportista emisor (ítems 10 y 11), hoy no contemplados en `schema.json`. | Medio | Backend + Negocio |
| TP-08 | Confirmar el formato del `UsernameToken` WS-Security: usuario secundario del transportista frente a certificado del contribuyente. | Medio | Integración |
| TP-09 | Definir si la plataforma consulta padrones de SUNAT para anticipar los rechazos `3389`, `3359`, `4398`, `4400` y `4412`. | Medio | Negocio + Arquitectura |
| TP-10 | Versionar el contrato XML junto con el XLSX de validaciones y el catálogo D-37, para que `version_plantilla` sea trazable. | Bajo | Backend |
