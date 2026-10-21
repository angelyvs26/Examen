# Catálogo de errores internos

> **Diseño propio, no forman parte de la normativa SUNAT.** Son errores que el sistema devuelve al usuario antes de transmitir a SUNAT.

## GRE- Errores de negocio de la Guía de Remisión

| Código | Severidad (ERROR/WARN/INFO) | Descripción | Causa típica | Acción correctiva | Requisito/Campo relacionado |
|---|---|---|---|---|---|
| GRE-001 | ERROR | Serie no habilitada para GRE | La serie indicada no está habilitada para Guía de Remisión Electrónica en el régimen del emisor | Verificar series habilitadas en configuración SUNAT / maestro de series | cabecera/serie, ID |
| GRE-002 | ERROR | Correlativo fuera de rango/habilitado | Correlativo no corresponde al rango habilitado para la serie | Usar correlativo dentro del rango habilitado o solicitar nuevo rango | cabecera/correlativo, ID |
| GRE-003 | ERROR | RUC destinatario inválido | Formato o validez del RUC del destinatario incorrecto | Corregir RUC del destinatario (validación de dígito verificador / formato) | destinatario/PartyIdentification/cbc:ID |
| GRE-004 | ERROR | Destino = Origen | Punto de llegada igual al punto de partida | Verificar datos de traslado: Ubigeo/dirección de partida y llegada deben ser distintos | Shipment/DeliveryAddress / DepartureAddress |
| GRE-005 | ERROR | Vehículo sin placa válida | Placa del vehículo principal no cumple formato o está vacía cuando es transporte por carretera | Registrar placa válida según normativa (formato peruano) | TransportHandlingUnit/TransportEquipment/LicensePlateID |
| GRE-006 | ERROR | Conductor sin datos obligatorios | Falta DNI/licencia o datos del conductor cuando aplica transporte por carretera | Completar datos obligatorios del conductor | DriverPerson/ID, Name |
| GRE-007 | ERROR | Items sin código SUNAT | Uno o más ítems sin código de producto SUNAT obligatorio | Asignar código SUNAT válido al ítem | DespatchLine/Item/CommodityClassification |
| GRE-008 | ERROR | Motivo de traslado inconsistente con documentos relacionados | Motivo seleccionado no requiere/contradice documentos relacionados obligatorios (DAM/DS según caso) | Ajustar motivo de traslado o agregar documentos relacionados correctos | DespatchAdviceTypeCode / motivo traslado / AdditionalDocumentReference |
| GRE-009 | ERROR | Unidad de medida inválida para el ítem | Unidad de medida no permitida o incompatible | Usar unidad de medida válida del catálogo SUNAT | DespatchLine/DeliveredQuantity@unitCode |
| GRE-010 | ERROR | Cantidad o peso inválidos | Cantidad/peso cero, negativo o fuera de rango | Corregir cantidad/peso a valores válidos | DespatchLine/DeliveredQuantity / GrossWeightMeasure |

## VAL- Errores de validación de datos de entrada

| Código | Severidad (ERROR/WARN/INFO) | Descripción | Causa típica | Acción correctiva | Requisito/Campo relacionado |
|---|---|---|---|---|---|
| VAL-001 | ERROR | Campo obligatorio vacío | Campo marcado como obligatorio no tiene valor | Completar el campo obligatorio | Genérico - campos M (obligatorios) |
| VAL-002 | ERROR | Formato inválido | Valor no cumple con formato/patrón esperado | Ajustar al formato requerido (fechas, números, códigos) | Genérico - formato |
| VAL-003 | ERROR | Longitud excedida o insuficiente | Longitud fuera del rango permitido | Ajustar longitud según especificación (min/max) | Genérico - longitud |
| VAL-004 | ERROR | Valor fuera de catálogo | Valor no existe en catálogo SUNAT permitido | Seleccionar valor válido del catálogo correspondiente | Catálogos SUNAT (01,06,20,59,61,...) |
| VAL-005 | WARN | Espacios en blanco o caracteres no permitidos | Presencia de whitespace/tab/line breaks o caracteres no esperados | Limpiar y normalizar el texto | Cualquier campo alfanumérico |
| VAL-006 | ERROR | Fechas incoherentes | Fecha de emisión posterior a fecha/hora de recepción o rangos inválidos | Corregir fecha/hora de emisión | IssueDate/IssueTime |
| VAL-007 | ERROR | Múltiples documentos relacionados inválidos | Cantidad de documentos relacionados excede lo permitido según tipo/modalidad | Ajustar cantidad según reglas GRE Remitente/Transportista | AdditionalDocumentReference |

## XSD- Errores de validación contra el esquema XSD

| Código | Severidad (ERROR/WARN/INFO) | Descripción | Causa típica | Acción correctiva | Requisito/Campo relacionado |
|---|---|---|---|---|---|
| XSD-001 | ERROR | XML mal formado | XML no bien formado (sintaxis inválida) | Corregir sintaxis XML | Estructura XML |
| XSD-002 | ERROR | No cumple esquema UBL 2.1/CustomizationID | Elementos/atributos requeridos faltantes o tipos incorrectos | Ajustar XML al esquema UBL DespatchAdvice requerido | UBLVersionID/CustomizationID y estructura UBL |
| XSD-003 | ERROR | Namespace incorrecto | Namespaces UBL/cac/cbc incorrectos o faltantes | Usar namespaces correctos según especificación | Namespaces |
| XSD-004 | ERROR | Cardinalidad violada | Elemento aparece 0 veces cuando es requerido o > veces cuando es único | Ajustar cardinalidad según validaciones XSL/UBL | Estructura global/por sección |
| XSD-005 | ERROR | Tipo de dato inválido | Tipo numérico/fecha/string no coincide con XSD | Corregir tipo de dato | Campos con tipos definidos |

## FIR- Errores de firma digital

| Código | Severidad (ERROR/WARN/INFO) | Descripción | Causa típica | Acción correctiva | Requisito/Campo relacionado |
|---|---|---|---|---|---|
| FIR-001 | ERROR | Certificado no encontrado o no cargado | Certificado digital no disponible en almacén/configuración | Verificar certificado (.pfx) y contraseña / almacén de certificados | Firma digital |
| FIR-002 | ERROR | Certificado vencido o no vigente | Fecha de vigencia del certificado fuera de rango | Renovar certificado o usar certificado vigente | Validez del certificado |
| FIR-003 | ERROR | Clave privada inaccesible | No se puede acceder a la clave privada del certificado | Verificar permisos / tipo de almacén / contraseña | Clave privada |
| FIR-004 | ERROR | Error al firmar el XML | Fallo en proceso de firma (Canonicalization/Signature) | Revisar algoritmo de firma, referencias y canonicalización | ds:Signature |
| FIR-005 | ERROR | Firma inválida o corrupta | Firma generada incorrectamente | Regenerar firma con certificado válido | ds:SignatureValue/X509Data |

## SUN- Errores de transporte hacia SUNAT

| Código | Severidad (ERROR/WARN/INFO) | Descripción | Causa típica | Acción correctiva | Requisito/Campo relacionado |
|---|---|---|---|---|---|
| SUN-001 | ERROR | Timeout de conexión/servicio | Timeout al conectar o esperar respuesta de SUNAT | Reintentar con backoff exponencial (ver política de reintentos); revisar conectividad | Transporte SOAP/REST a servicios SUNAT |
| SUN-002 | ERROR | Error HTTP 5xx | Servicio SUNAT retorna error del servidor (5xx) | Reintentar con backoff exponencial; si persiste, pasar a PENDIENTE_REINTENTO | Respuesta HTTP |
| SUN-003 | ERROR | WS-Security inválido/fallido | Fallo en cabecera WS-Security (token/credenciales/firma WS) | Verificar credenciales SUNAT, certificado para WS-Security y configuración | WS-Security |
| SUN-004 | ERROR | Credenciales inválidas o sin autorización | Usuario/clave incorrectos o sin perfil para envío de GRE | Verificar usuario secundario/autorización para envío electrónico | Autenticación SUNAT |
| SUN-005 | WARN/ERROR | Respuesta SOAP malformada o inesperada | Respuesta no parseable o inesperada del servicio | Loguear respuesta completa; reintentar o escalar; validar estructura de respuesta | Respuesta del servicio |

## INT- Errores internos inesperados

| Código | Severidad (ERROR/WARN/INFO) | Descripción | Causa típica | Acción correctiva | Requisito/Campo relacionado |
|---|---|---|---|---|---|
| INT-001 | ERROR | Excepción no controlada | Error inesperado en la aplicación | Revisar logs, stack trace y corregir bug; notificar al equipo | Sistema interno |
| INT-002 | ERROR | Error de configuración | Parámetros/configuración faltantes o inválidos | Verificar configuración (URLs, certificados, rutas) | Configuración |
| INT-003 | ERROR | Error de E/S o almacenamiento | No se puede leer/escribir archivo/ZIP/XML | Verificar permisos y espacio en disco | Almacenamiento/archivos |
| INT-004 | ERROR | Error de serialización/parseo | Fallo al generar/parsear XML/JSON | Revisar mapeo y transformación de datos | Mapeo de datos |
| INT-005 | ERROR | Recurso no disponible | Dependencia interna no disponible | Verificar estado de dependencias; reintentar si aplica | Recursos del sistema |

