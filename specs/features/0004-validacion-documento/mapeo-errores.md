# Mapeo de errores: SUNAT → Sistema interno

## Reglas generales
- **ERROR** de SUNAT = no se reintenta automáticamente, la GRE queda en estado **RECHAZADA**.
- **OBSERV** = se acepta pero se registra la observación y la GRE queda **ACEPTADA_CON_OBSERVACIONES**.
- Errores de **timeout/5xx/WS-Security** = reintento con **backoff exponencial** hasta **N intentos** y luego pasa a estado **PENDIENTE_REINTENTO**.

## Mapeo

| Tipo de retorno SUNAT (ERROR/OBSERV) | Severidad interna | Comportamiento del sistema | Acción del usuario |
|---|---|---|---|
| ERROR (validación XSL/estructura) | ERROR | Validación rechazada. No se reintenta automáticamente. GRE → RECHAZADA. Registrar código/mensaje SUNAT en log de envío. | Corregir XML/datos según código de retorno SUNAT y reenviar manualmente. |
| ERROR (Identificación-documento: duplicado/anulado) | ERROR | Rechazo por estado/documento. No reintenta automáticamente. GRE → RECHAZADA. | Verificar estado en SUNAT/consulta; si ya fue aceptado, no reenviar; si corresponde anulación usar procedimiento adecuado. |
| ERROR (Maestro-datos: RUC inactivo/inhabido/afecto) | ERROR | Rechazo. No reintenta automáticamente. GRE → RECHAZADA. | Verificar situación del RUC (activo/habido) y régimen tributario. Corregir datos o regularizar situación. |
| ERROR (Firma/seguridad/autenticación) | ERROR | Rechazo por autenticación/firma. No reintenta automáticamente. GRE → RECHAZADA. | Verificar credenciales, certificado digital y configuración WS-Security. Corregir y reenviar. |
| OBSERV (atributos opcionales/formato menores) | WARN | Se acepta con observación. Se registra observación. GRE → ACEPTADA_CON_OBSERVACIONES. | Revisar observación; corregir para envíos futuros si desea cumplir íntegramente con recomendaciones. |
| OBSERV (campos descriptivos/estándar) | WARN | Se acepta con observación. Se registra observación. GRE → ACEPTADA_CON_OBSERVACIONES. | Revisar y normalizar datos según estándar cuando aplique. |
| SUN-001 Timeout (transporte) | ERROR | Error de transporte. **Reintento con backoff exponencial hasta N intentos**. Si supera N intentos → **PENDIENTE_REINTENTO**. | Esperar reintentos automáticos o reenviar manualmente más tarde. Verificar conectividad. |
| SUN-002 5xx (transporte) | ERROR | Error de servidor SUNAT. **Reintento con backoff exponencial hasta N intentos**. Si supera N intentos → **PENDIENTE_REINTENTO**. | Esperar reintentos automáticos. Si persiste, contactar soporte o reintentar manualmente. |
| SUN-003 WS-Security (transporte) | ERROR | Fallo WS-Security. Según política: puede reintentar con backoff exponencial hasta N intentos si es transitorio; si persiste por credenciales/config → **PENDIENTE_REINTENTO** o **RECHAZADA** según diagnóstico. | Verificar credenciales/certificado WS-Security. Corregir configuración antes de reintentos intensivos. |
| SUN-004 Credenciales/sin perfil | ERROR | Error de autorización. **No reintenta automáticamente** (salvo corrección de credenciales). GRE → **RECHAZADA** si definitivo. | Verificar usuario secundario, perfil para envío electrónico y autorización. |
| INT-* / errores internos | ERROR | Error interno. Reintento según criticidad con backoff exponencial hasta N intentos; si no recuperable → **PENDIENTE_REINTENTO** / escalado. | Revisar logs internos o contactar soporte. |

