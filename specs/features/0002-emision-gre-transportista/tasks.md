# 0002 — Plan de tareas: Emisión de la GRE Transportista

| Campo | Valor |
|---|---|
| Código | 0002 |
| Documento | Plan de tareas |
| Estado | Borrador |
| Versión | 1.0.0 |
| Fecha | 2026-10-21 |
| Especificación | [spec.md](spec.md) |
| Diseño | [design.md](design.md) |

## 1. Convenciones

| Elemento | Convención |
|---|---|
| Identificador | `T-<bloque>-<secuencia>`, donde el bloque identifica la fase (00 a 50) y la secuencia es correlativa dentro de la fase. |
| Requisito | Referencia al requisito funcional (`RF-n`), regla de negocio (`RN-n`), regla no funcional (`RNF-n`), historia de usuario (`US-n`) o decisión de diseño (`DD-n`) del que proviene la tarea. |
| Estimación | Puntos de historia: 1 (trivial), 2 (simple), 3 (media), 5 (compleja), 8 (muy compleja). |
| Dependencias | Identificadores de tarea que deben completarse antes. |
| Estado | `Pendiente`, `En curso`, `Bloqueada`, `Completada`. |

## 2. Fases

| Fase | Nombre | Objetivo |
|---|---|---|
| 0 | Preparación y verificación | Resolver las discrepancias normative pendientes antes de escribir código. |
| 1 | Fundaciones de dominio | Modelo de dominio, strategy y catálogos internos. |
| 2 | Validación | Reglas de negocio de la hoja «Guía-Transportista2_0». |
| 3 | Generación de XML | Mapper JSON → XML UBL 2.1 del estándar Transportista. |
| 4 | Persistencia y API | Almacenamiento, orquestación, superficies REST y gRPC. |
| 5 | Pruebas, despliegue y operación | Verificación integral, piloto y puesta en producción. |

## 3. Tareas

### Fase 0 — Preparación y verificación

| ID | Tarea | Requisito | Est. | Dependencias | Estado |
|---|---|---|---|---|---|
| T-00-01 | Confirmar el literal del indicador de subcontratación (`SUNAT_Envio_IndicadorTrasporteSubcontratado` o `SUNAT_Envio_IndicadorTransporteSubcontratado`) contra el XLSX de validaciones y el WS de recepción; unificar el contrato XML y `schema.json` | TP-01, RN-12 | 3 | — | Pendiente |
| T-00-02 | Verificar en el XSD oficial la posición y cardinalidad de `cac:ShipmentStage`, `cac:Consignment` y `cac:OriginatorCustomerParty`, y los URI de namespace que exige el WS de recepción | TP-02, TP-03, DD-3 | 5 | — | Pendiente |
| T-00-03 | Revisar `schema.json`: `envio.motivo` y `envio.modalidad` como `required` condicional por `tipoDocumento`; cantidad y unidad opcionales para el ítem 51; alta de los campos de registro MTC y entidad autorizadora | TP-04, TP-05, TP-07, RN-1 | 5 | T-00-01 | Pendiente |
| T-00-04 | Determinar si `SUNAT_Envio_IndicadorTrasladoVehiculoM1L` aplica a la GRE-Transportista y documentar el resultado | TP-06, RN-14 | 2 | — | Pendiente |
| T-00-05 | Confirmar el formato del `UsernameToken` WS-Security: usuario secundario del transportista o certificado del contribuyente | TP-08 | 2 | — | Pendiente |
| T-00-06 | Versionar el contrato XML junto con el XLSX de validaciones y el catálogo D-37, de modo que `version_plantilla` sea trazable | TP-10, DD-5 | 3 | T-00-01, T-00-04 | Pendiente |

**Total Fase 0: 6 tareas, 20 puntos.**

### Fase 1 — Fundaciones de dominio

| ID | Tarea | Requisito | Est. | Dependencias | Estado |
|---|---|---|---|---|---|
| T-10-01 | Implementar los value objects `IndicadorSubcontratacion`, `Subcontratador`, `IndicadorPagadorFlete`, `RolPagadorFlete` y `PagadorFlete` con sus invariantes | DD-2, RN-12, RN-12.1, RN-13 | 5 | T-00-01 | Pendiente |
| T-10-02 | Definir la clase de constantes de indicadores `SUNAT_Envio_*` y el enumerado de roles de pagador del flete, con el literal del indicador de subcontratación en un único punto | FM-4, RN-12, RN-13 | 2 | T-00-01 | Pendiente |
| T-10-03 | Definir el enumerado de estados del documento (`BORRADOR`, `VALIDADA`, `FIRMADA`, `ENVIADA`, `ACEPTADA`, `ACEPTADA_CON_OBSERVACIONES`, `RECHAZADA`, `PENDIENTE_REINTENTO`) | RF-7, RF-8 | 2 | — | Pendiente |
| T-10-04 | Definir la interfaz `EstrategiaGuia` y la `FabricaEstrategiaGuia` con la resolución por `tipoDocumento`, sin alterar la estrategia de la feature 0001 | DD-1, RNF-11 | 5 | T-00-02 | Pendiente |
| T-10-05 | Catalogar los 64 ítems de la hoja «Guía-Transportista2_0» y los 178 códigos de retorno en constantes tipadas con su descripción oficial | RF-3, RN-1..RN-25 | 3 | T-00-02 | Pendiente |
| T-10-06 | Escribir las pruebas unitarias de los value objects: subcontratación sin subcontratador, subcontratador igual al transportista, pagador tercero sin datos, pagador no tercero con datos | T-10-01 | 3 | T-10-01, T-10-02 | Pendiente |

**Total Fase 1: 6 tareas, 20 puntos.**

### Fase 2 — Validación

| ID | Tarea | Requisito | Est. | Dependencias | Estado |
|---|---|---|---|---|---|
| T-20-01 | Implementar las reglas de cabecera: `UBLVersionID` (2111, 2110), `CustomizationID` (2113, 2112), tipo de guía `31` (1050, 1051), identificador y correspondencia con el nombre del archivo (1001, 1032–1036) | RN-1, RN-2, RN-3, RF-3 | 3 | T-10-05 | Pendiente |
| T-20-02 | Implementar las reglas de fecha y hora de emisión (3436, 2108, 2329, 3437, 3438) | RN-4 | 2 | T-10-05 | Pendiente |
| T-20-03 | Implementar las reglas del transportista emisor: tipo y número de documento, RUC activo y habido, afecto a los tributos autorizados y razón social (ítems 8 y 9) | RN-5 | 5 | T-10-05 | Pendiente |
| T-20-04 | Implementar las reglas de registro MTC y de entidad autorizadora del transportista (ítems 10 y 11), incluida la unicidad del número de autorización | RN-6 | 2 | T-10-05 | Pendiente |
| T-20-05 | Implementar las reglas de identidad del remitente y la diferencia respecto del transportista (ítems 16 y 17) | RN-7, RN-8 | 3 | T-10-05 | Pendiente |
| T-20-06 | Implementar las reglas de identidad del destinatario y su coincidencia con la GRE-Remitente relacionada (ítems 18 y 19) | RN-9 | 3 | T-10-05 | Pendiente |
| T-20-07 | Implementar las reglas de documentos relacionados: descripción, cardinalidad, duplicados, formato del número y RUC del emisor (ítems 12 a 15) | RN-10, RN-10.1, RN-10.2, RN-10.3, RN-10.4 | 5 | T-10-05 | Pendiente |
| T-20-08 | Implementar las reglas de vehículos: vehículo principal obligatorio, placa, TUC o Certificado de Habilitación, entidad autorizadora y límite de secundarios (ítems 35 a 40) | RN-11 | 5 | T-10-05 | Pendiente |
| T-20-09 | Implementar las reglas de conductores: unicidad del principal, límite de secundarios, identidad, nombres y licencia (ítems 41 a 48) | RN-15 | 5 | T-10-05 | Pendiente |
| T-20-10 | Implementar las reglas de dirección de partida y de llegada: ubigeo, dirección detallada y georreferencia (ítems 29 a 34) | RN-16 | 3 | T-10-05 | Pendiente |
| T-20-11 | Implementar las reglas de fecha de inicio de traslado y de peso bruto total con su unidad (ítems 50 y 52) | RN-17, RN-18 | 3 | T-10-05 | Pendiente |
| T-20-12 | Implementar las reglas de bienes a transportar: obligatoriedad del detalle, cantidad, unidad de medida, descripción, código de producto SUNAT y GTIN (ítems 20 a 26) | RN-19, RN-20 | 5 | T-10-05 | Pendiente |
| T-20-13 | Implementar las reglas de la anotación opcional sobre los bienes y de las observaciones del documento (ítems 51 y 7) | RN-21, RN-22 | 2 | T-10-05 | Pendiente |
| T-20-14 | Implementar las reglas de indicadores: unicidad, pertenencia al conjunto admitido y coherencia con los bloques de datos (ítems 53 a 58) | RN-14, FM-1, FM-2 | 3 | T-10-02, T-10-05 | Pendiente |
| T-20-15 | Implementar las reglas de subcontratación y del bloque `cac:Consignment` (ítems 60 y 61) | RN-12, RN-12.1, RN-12.2, RF-11 | 5 | T-10-01, T-10-05 | Pendiente |
| T-20-16 | Implementar las reglas del pagador del flete y del bloque `cac:OriginatorCustomerParty` (ítems 62 y 63) | RN-13, RN-13.1, RN-13.2, RF-12 | 3 | T-10-01, T-10-05 | Pendiente |
| T-20-17 | Implementar las reglas del identificador del traslado `SUNAT_Envio` y la prohibición de consignar el tipo de evento (ítems 49 y 59) | RN-23, RN-24 | 1 | T-10-05 | Pendiente |
| T-20-18 | Implementar el orquestador `ValidacionGreTransportista` que ejecuta las reglas, agrega las violaciones y las devuelve con ítem, tag UBL, código de retorno y mensaje | US-6, RF-3, RF-15 | 5 | T-20-01..T-20-17 | Pendiente |

**Total Fase 2: 18 tareas, 63 puntos.**

### Fase 3 — Generación de XML

| ID | Tarea | Requisito | Est. | Dependencias | Estado |
|---|---|---|---|---|---|
| T-30-01 | Construir la tabla de correspondencia `CorrespondenciaItem` (ítem, XPath destino, extractor, formateador, obligatoriedad) a partir del contrato XML | DD-3, RF-4 | 5 | T-00-02, T-10-05 | Pendiente |
| T-30-02 | Implementar el mapper de cabecera, emisor, remitente y destinatario | RF-4, RN-1, RN-5, RN-8 | 5 | T-30-01 | Pendiente |
| T-30-03 | Implementar el `MapperConsignacion`: `cac:Consignment` con `cbc:ID` = `SUNAT_Envio` y `cac:LogisticsOperatorParty` | RF-11, RN-12, FM-3 | 3 | T-30-01, T-10-02 | Pendiente |
| T-30-04 | Implementar el `MapperPagadorFlete`: indicador correspondiente y `cac:OriginatorCustomerParty` | RF-12, RN-13, FM-3 | 3 | T-30-01, T-10-02 | Pendiente |
| T-30-05 | Implementar los mappers de vehículos, conductores, puntos de partida y llegada y documentos de transporte | RF-4, RN-14 | 5 | T-30-01 | Pendiente |
| T-30-06 | Implementar el mapper de ítems y bienes, con la excepción del ítem 51 | RF-4, RN-21.1 | 5 | T-30-01 | Pendiente |
| T-30-07 | Implementar el `GeneradorXmlGuia` con JAXB/Marshaller, namespaces UBL 2.1, escape de caracteres y normalización de saltos de línea | RF-4, FM-7, FM-8 | 5 | T-30-02..T-30-06 | Pendiente |
| T-30-08 | Implementar la construcción de los nombres de archivo `RUC-31-VAAA-CCCCCCCC-XML` y `RUC-31-VAAA-CCCCCCCC-ZIP` desde la misma fuente que el `cbc:ID` | RN-2, FM-5 | 2 | T-30-07 | Pendiente |
| T-30-09 | Generar los golden files del XML de la GRE-T y validarlos contra el XSD oficial en CI | RF-4, RNF-7 | 5 | T-30-07, T-30-08 | Pendiente |

**Total Fase 3: 9 tareas, 38 puntos.**

### Fase 4 — Persistencia y API

| ID | Tarea | Requisito | Est. | Dependencias | Estado |
|---|---|---|---|---|---|
| T-40-01 | Crear las migraciones de esquema: `gre_documento`, `gre_item`, `gre_documento_relacionado`, `gre_consignacion`, `gre_pagador_flete`, `gre_envio`, `gre_observacion` y `serie_correlativo`, con sus restricciones `UNIQUE` y `CHECK` | DD-4, §6.1 | 5 | T-10-01 | Pendiente |
| T-40-02 | Crear los índices de consulta y aplicar la política RLS por `app.ruc` | DD-7, RNF-9 | 3 | T-40-01 | Pendiente |
| T-40-03 | Implementar los repositorios de documento, ítems, documentos relacionados, consigna, pagador, envíos y observaciones | RF-6, RF-7 | 5 | T-40-01 | Pendiente |
| T-40-04 | Implementar la reserva de correlativo con `UPDATE ... RETURNING` dentro de la transacción de emisión | DD-4, RN-2, RF-2 | 5 | T-40-01 | Pendiente |
| T-40-05 | Implementar `GreService.emitir`: validación de esquema, autorización por RUC, validación de reglas, reserva de correlativo, persistencia y generación del XML | US-1, RF-1..RF-5 | 8 | T-20-18, T-30-07, T-40-03, T-40-04 | Pendiente |
| T-40-06 | Implementar el control de idempotencia por `Idempotency-Key` con detección de conflicto por cuerpo distinto | RNF-8, FA-5 | 3 | T-40-05 | Pendiente |
| T-40-07 | Exponer los endpoints REST de emisión, consulta de detalle, descarga de XML y ZIP, y envío | RF-1, RF-6, RF-8 | 5 | T-40-05 | Pendiente |
| T-40-08 | Exponer la superficie gRPC equivalente, con `TIPO_DOCUMENTO_GRE_TRANSPORTISTA` = `31` | RF-9 | 5 | T-40-07 | Pendiente |
| T-40-09 | Implementar la validación previa sin persistencia ni consumo de correlativo | US-6, RF-3 | 3 | T-20-18 | Pendiente |
| T-40-10 | Integrar la firma XMLDSig y el empaquetado ZIP con la nomenclatura `31` de la feature 0005 | RF-5, RN-25 | 5 | T-40-05, T-00-05 | Pendiente |
| T-40-11 | Implementar el envío a SUNAT con la clasificación de errores y el plan de reintentos con backoff exponencial | RF-8, DD-6, FA-7..FA-9 | 5 | T-40-07, T-40-10 | Pendiente |
| T-40-12 | Implementar el registro de auditoría de las operaciones de emisión, consulta y envío | RF-10, RNF-4 | 2 | T-40-05 | Pendiente |
| T-40-13 | Implementar logs estructurados, métricas, trazas por etapa y alertas | RNF-3, §10 | 3 | T-40-05 | Pendiente |
| T-40-14 | Implementar el enmascaramiento de los DNI de conductores y contactos, y el manejo seguro de credenciales | RNF-10, §9 | 3 | T-40-05 | Pendiente |

**Total Fase 4: 14 tareas, 60 puntos.**

### Fase 5 — Pruebas, despliegue y operación

| ID | Tarea | Requisito | Est. | Dependencias | Estado |
|---|---|---|---|---|---|
| T-50-01 | Completar la suite unitaria con una prueba por cada código de retorno de la hoja «Guía-Transportista2_0» | RNF-7, §11 | 8 | T-20-18 | Pendiente |
| T-50-02 | Implementar las pruebas de integración con Testcontainers: persistencia, RLS, idempotencia y reserva de correlativo | RF-2, RNF-9 | 5 | T-40-05 | Pendiente |
| T-50-03 | Implementar la prueba de concurrencia de 50 emisiones sobre la misma serie `V###` | RN-2, RNF-8 | 3 | T-40-04, T-40-05 | Pendiente |
| T-50-04 | Implementar la prueba end-to-end con WireMock: emisión, firma, envío, ticket, observaciones y rechazo | US-5, RF-7, RF-8 | 5 | T-40-11 | Pendiente |
| T-50-05 | Ejecutar la suite de regresión de la feature 0001 sin cambios | RNF-11, DD-1 | 3 | T-40-05 | Pendiente |
| T-50-06 | Ejecutar el piloto con un transportista, con seguimiento diario de los códigos de retorno por ítem | §12 | 5 | T-50-01..T-50-05 | Pendiente |
| T-50-07 | Actualizar la documentación de la API y redactar la guía de integración para el ERP del transportista | RF-9, §12 | 3 | T-40-07, T-40-08 | Pendiente |
| T-50-08 | Elaborar el plan de reversión, el runbook operativo y el procedimiento de atención de rechazos con los códigos de retorno | §8, §12 | 2 | T-50-06 | Pendiente |

**Total Fase 5: 8 tareas, 34 puntos.**

## 4. Resumen

| Fase | Tareas | Puntos |
|---|---|---|
| 0 — Preparación y verificación | 6 | 20 |
| 1 — Fundaciones de dominio | 6 | 20 |
| 2 — Validación | 18 | 63 |
| 3 — Generación de XML | 9 | 38 |
| 4 — Persistencia y API | 14 | 60 |
| 5 — Pruebas, despliegue y operación | 8 | 34 |
| **Total** | **61** | **235** |

## 5. Grafo de dependencias

```text
Fase 0 (verificación normativa)
   |  T-00-01, T-00-02, T-00-03, T-00-04, T-00-05, T-00-06
   v
Fase 1 (dominio)
   |  T-10-01 .. T-10-06
   v
Fase 2 (validación)          Fase 3 (XML)
   |  T-20-01 .. T-20-18      |  T-30-01 .. T-30-09
   |                           |
   +-----------+---------------+
               v
Fase 4 (persistencia y API)   T-40-01 .. T-40-14
               v
Fase 5 (pruebas y despliegue) T-50-01 .. T-50-08
```

Ruta crítica estimada: `T-00-02` → `T-30-01` → `T-30-07` → `T-40-05` → `T-50-04`.

## 6. Trazabilidad requisito → tarea

| Requisito | Tareas |
|---|---|
| RF-1 | T-40-05, T-40-07, T-40-08, T-40-09 |
| RF-2 | T-40-04, T-50-03 |
| RF-3 | T-20-01..T-20-18, T-40-09, T-50-01 |
| RF-4 | T-30-01..T-30-07, T-30-09 |
| RF-5 | T-40-05, T-40-10 |
| RF-6 | T-40-03, T-40-07 |
| RF-7 | T-40-03, T-40-11, T-50-04 |
| RF-8 | T-40-11, T-50-04 |
| RF-9 | T-40-08, T-50-07 |
| RF-10 | T-40-12 |
| RF-11 | T-10-01, T-20-15, T-30-03, T-40-01, T-50-01 |
| RF-12 | T-10-01, T-20-16, T-30-04, T-40-01, T-50-01 |
| RF-13 | T-20-07, T-20-12, T-40-03 |
| RF-14 | T-10-03, T-20-01, T-40-04 |
| RF-15 | T-20-12, T-40-03, T-40-11 |
| RNF-1, RNF-2 | T-40-05, T-40-09, T-50-04 |
| RNF-3 | T-40-13 |
| RNF-4 | T-40-12 |
| RNF-5 | T-40-05, T-40-14 |
| RNF-6 | T-30-08, T-40-05 |
| RNF-7 | T-30-09, T-50-01 |
| RNF-8 | T-40-06, T-50-03 |
| RNF-9 | T-40-02, T-50-02 |
| RNF-10 | T-40-14 |
| RNF-11 | T-10-04, T-50-05 |
| US-1 | T-20-01..T-20-06, T-20-08..T-20-13, T-20-17, T-40-05, T-40-07 |
| US-2 | T-20-15, T-30-03, T-40-05 |
| US-3 | T-20-16, T-30-04, T-40-05 |
| US-4 | T-20-07, T-20-12 |
| US-5 | T-40-03, T-40-07, T-50-04 |
| US-6 | T-20-18, T-40-09 |
| DD-1 | T-10-04, T-50-05 |
| DD-4 | T-40-01, T-40-04 |
| DD-6 | T-40-11 |
| DD-7 | T-40-02, T-40-05 |

## 7. Hitos

| Hito | Tareas que lo cierran | Criterio de cierre |
|---|---|---|
| H1 — Normativa verificada | T-00-01..T-00-06 | Las discrepancias TP-01 a TP-10 están resueltas o documentadas con decisión. |
| H2 — Dominio y validación | T-10-01..T-10-06, T-20-01..T-20-18 | Los 64 ítems tienen regla y prueba unitaria. |
| H3 — XML conforme | T-30-01..T-30-09 | El golden file de la GRE-T valida contra el XSD oficial sin observaciones. |
| H4 — API operativa | T-40-01..T-40-14 | La emisión persiste, firma, envía y expone su estado; la regresión de la feature 0001 pasa. |
| H5 — Listo para producción | T-50-01..T-50-08 | Piloto sin rechazos por causa no documentada y plan de reversión validado. |
