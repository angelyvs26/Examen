# ADR-0002: Autenticación y autorización de la API mediante JWT

| Campo | Valor |
|---|---|
| Estado | Aceptada |
| Fecha | 2026-10-21 |
| Decisores | Equipo de arquitectura (com.examen.gre) |
| Supervisores | Responsable de plataforma, responsable de seguridad |
| Módulo afectado | Transversal (API de emisión, consulta, catálogos, envío) |
| Etiquetas | #adr #arquitectura #seguridad |

## Contexto

La plataforma expone una API REST para emitir y consultar GRE (ver [openapi.yaml](../contracts/openapi.yaml)) y, en paralelo, un canal gRPC para integraciones de alto volumen (ver [grpc_service.proto](../contracts/grpc_service.proto)). Sus consumidores son:

1. **Sistemas ERP / WMS del contribuyente** (integración máquina a máquina, alto volumen, sin usuario humano detrás).
2. **Usuarios humanos** del contribuyente y del operador del servicio (consultan estado, corrigen datos, reenvían).
3. **Servicios de terceros** (consulta de estados, integraciones con operadores logísticos).

Requisitos:

- **Multiempresa por RUC**: un cliente autenticado puede operar sobre uno o varios RUC emisores; la autorización es por RUC, no sólo por rol global. El RUC nunca se toma del cuerpo de la petición como fuente de verdad: se toma del token.
- **Separación de identidades**: la autenticación de la API (quién llama) es distinta de la autenticación ante SUNAT (certificado digital X.509 en WS-Security). Confundirlas es un riesgo habitual en este tipo de soluciones.
- **Auditoría**: cada operación debe quedar atribuida a un sujeto (cliente + usuario) y a un RUC emisor.
- **Operación sin estado**: el servicio debe escalar horizontalmente sin sesiones pegajosas ni almacenamiento compartido de sesión.
- **Desacoplamiento**: los ERP suelen necesitar autenticación simple de configurar (API key o usuario/contraseña), mientras que el acceso humano requiere un flujo de sesión decente (login, expiración, revocación).

Restricción adicional: la plataforma **no** puede delegar en un proveedor de identidad SaaS de pago sin acuerdo con el cliente; el despliegue debe poder ser autoalojado.

## Problema

¿Qué mecanismo de autenticación y autorización se usa en la API y en gRPC, y cómo se modela el acceso multi-RUC sin que el RUC pueda ser suplantado desde el cuerpo de la petición?

## Opciones consideradas

### Opción A — Sesiones con cookie (Spring Security + Spring Session)

- **Descripción**: login con usuario/contraseña, cookie `SESSIONID`, estado de sesión en servidor o en Redis.
- **Ventajas**: modelo conocido, revocación inmediata, fácil de usar en la consola web, no requiere un análisis de expiración corta.
- **Desventajas**: acoplamiento a cookies → complejidad de CORS/CSRF; los ERP no gestionan cookies bien y necesitan un flujo de token equivalente de todas formas; estado compartido entre réplicas.
- **Costos / esfuerzo**: bajo.

### Opción B — JWT firmado con HMAC (HS256) y secreto compartido

- **Descripción**: el mismo secreto simétrico firma y verifica; payload con `exp`, `sub`, `ruc`, `scope`.
- **Ventajas**: implementación mínima, sin claves públicas, verificación local rápida; secreto rotable sin distribuirlo a los clientes.
- **Desventajas**: cualquiera que valide tokens puede **emitirlos**: el compromiso del secreto compromete toda la cadena; imposible verificar sin conocer el secreto (terceros); no hay no-repudio del emisor del token.
- **Costos / esfuerzo**: muy bajo.

### Opción C — JWT firmado asimétricamente (RS256 / ES256) con JWKS publicado

- **Descripción**: un par de claves RSA/ECDSA, sólo la clave privada en el emisor de tokens y la pública publicada en un endpoint JWKS; cada servicio valida localmente.
- **Ventajas**: sólo quien tiene la clave privada emite; los recursos (API, gRPC, otros servicios) validan sin llamada de red; se puede delegar la verificación a bibliotecas estándar; permite rotación de claves con `kid`; revocación posible vía `jti` en lista de bloqueo.
- **Desventajas**: gestión del par de claves y de su rotación; los clientes M2M necesitan el flujo de token (`client_credentials`) para no manejar claves.
- **Costos / esfuerzo**: medio.

### Opción D — OAuth2/OIDC con un IdP externo (Keycloak, Auth0, Entra ID)

- **Descripción**: un servidor de identidad emite tokens; la API es *resource server* puro.
- **Ventajas**:Federación con el IdP corporativo del cliente; MFA, políticas de contraseña, auditoría y SSO resueltos por el IdP; la API queda desacoplada de la identidad.
- **Desventajas**: dependencia operativa de un componente adicional; en la instalación autoalojada implica desplegar y mantener Keycloak; el flujo OIDC completo no es trivial para un integrador M2M pequeño.
- **Costos / esfuerzo**: medio-alto.

### Opción E — mTLS entre el ERP y la API

- **Descripción**: autenticación por certificado cliente en TLS.
- **Ventajas**: identidad criptográfica fuerte, sin secretos de token.
- **Desventajas**: gestión de PKI para clientes; sinoks de autorización (RUC, roles) hay que resolverla aparte; incompatible con clientes web sin proxy.
- **Costos / esfuerzo**: alto.

## Decisión

Se adopta **OAuth 2.0 sobre JWT con firma asimétrica (RS256)**, emitido por un **Authorization Server embebido (Spring Authorization Server)** y verificado localmente por la API y el servidor gRPC como *resource server*.

Componentes:

1. **Emisor de tokens** (`/oauth2/token`, `/oauth2/jwks`, `/oauth2/authorize`):
   - **M2M**: `client_credentials` para ERP/WMS. Credenciales del cliente (id + secreto) registradas en el emisor; el secreto se almacena con hash (bcrypt/argon2) y puede rotarse sin downtime.
   - **Humano**: `authorization_code` + PKCE contra la consola web, con MFA opcional delegable a un IdP en el futuro.
2. **Firma y validación**: RSA-2048/3072 (RS256), clave privada en el almacén de secretos / HSM, clave pública expuesta en JWKS con `kid`. La API valida firma, `iss`, `aud`, `exp` (tolerancia de 60 s) y `nbf` **localmente**, sin llamada de red por petición.
3. **Claims del token**:

   | Claim | Contenido | Uso |
   |---|---|---|
   | `iss` / `aud` | Emisor y audiencia de la API | Validación |
   | `sub` | Identificador del cliente (`client_id`) o del usuario | Sujeto de auditoría |
   | `exp` | Caducidad | M2M: 15 min · humano: 10 min de acceso + refresh rotativo de 8 h |
   | `jti` | Identificador único del token | Trazabilidad y bloqueo |
   | `scope` | `gre.emitir`, `gre.consultar`, `gre.enviar`, `catalogos.leer`, `admin` | Autorización por operación |
   | `ruc` | Lista de RUC emisores habilitados para el sujeto | Autorización multi-RUC (**fuente de verdad**) |

   El `ruc` habilitado se toma **siempre** del token verificado. El RUC del cuerpo de la petición se valida contra esa lista y, si no pertenece al sujeto, la petición se rechaza con `403 RUC_NO_AUTORIZADO`.
4. **Autorización**: dos capas — (a) *scope* para la operación, (b) *RUC* para el recurso. Composición evaluada en el filtro de seguridad antes de llegar al controlador.
5. **Revocación**: lista de bloqueo de `jti` en base de datos con TTL = expiración del token, para logout y para revocación de credenciales M2M. El refresh token es rotativo: al usarlo se emite uno nuevo y el anterior queda invalidado.
6. **Hardening**: rate limiting por `client_id` y por `sub` (p. ej. 60 req/min por cliente), bloqueo temporal tras N fallos de autenticación, y registro de todos los accesos en la auditoría.

**Frontera explícita con SUNAT**: el canal SOAP con SUNAT se autentica con **certificado digital X.509 mediante WS-Security** (`UsernameToken` con clave privada PKCS#12), conforme al manual de SUNAT para comprobantes de pago electrónico. Ese certificado es material del emisor de comprobantes y **no** se relaciona con los tokens JWT de la API: un token válido de API no habilita por sí solo el envío a SUNAT, y el par de claves JWT no sustituye al certificado.

**Tecnología elegida**: Spring Authorization Server 1.4 (emisor), Spring Security 6 (resource server, validación de JWT y control de acceso por scope/RUC), Nimbus JOSE+JWT para firma/verificación, JWKS expuesto en `/.well-known/jwks.json`.

**Configuración clave**: `issuer=https://api.<dominio>/oauth2`; `audience=gre-api`; TTL acceso M2M 15 min, humano 10 min; refresh humano 8 h rotativo; skew de reloj 60 s; claves RSA 3072 con rotación semestral anunciada por `kid`.

## Consecuencias

### Positivas

- La API es sin estado: escala horizontalmente sin sesiones compartidas.
- La verificación es local y rápida (RSA-2048 verifica en ~0,1 ms), sin acoplar disponibilidad con el emisor de tokens.
- El RUC proviene del token, lo que elimina la clase de fallos de suplantación de tenant.
- El mismo modelo sirve para M2M y humano: un ERP no necesita cookies ni navegador; un usuario sí tiene login, expiración y revocación.
- La frontera con SUNAT queda documentada: dos mecanismos de autenticación, dos dominios de riesgo y de rotación de material criptográfico.

### Negativas / riesgos

- **El emisor de tokens es un componente crítico**: si cae, no se pueden obtener tokens nuevos (los ya emitidos siguen valiendo hasta `exp`) → mitigación: alta disponibilidad (2 réplicas), y tokens M2M de vida corta para acotar la ventana.
- **Gestión de claves**: la clave privada del emisor es un activo crítico → mitigación: almacén de secretos (o HSM en la fase de producción), rotación programada con `kid`, procedimiento documentado de revocación.
- **Complejidad de la integración M2M**: el cliente ERP debe consumir `/oauth2/token` y refrescar el token → mitigación: se publica un SDK/cliente de ejemplo y un flujo alternativo de API key *no recomendado* para evitar que los clientes inventen su propia gestión de tokens.
- **Scope creep de la autorización**: con scopes + RUC, la matriz de permisos crece → mitigación: matriz de scopes documentada en la spec de la feature 0001 y pruebas de autorización por rol.

### Impacto en otros módulos

| Módulo | Impacto | Acción requerida |
|---|---|---|
| 0001 Emisión GRE Remitente | Alto | El endpoint de emisión exige scope `gre.emitir` y valida el RUC del token contra el de la serie. |
| 0002 Emisión GRE Transportista | Alto | Igual, con la serie `V###` ligada al RUC del transportista. |
| 0003 Catálogos | Bajo | Lectura autenticada con scope `catalogos.leer`; sin escritura por API (sólo importación batch). |
| 0004 Validación | Medio | El resultado de la validación se guarda asociado al `jti`/`sub` que la originó. |
| 0005 Firma y envío | Alto | El envío a SUNAT usa WS-Security con el certificado del contribuyente; el JWT no interviene en ese canal. |

## Validación

- Prueba de acceso directo sin token → `401`; token con scope insuficiente → `403`; token válido con RUC no habilitado → `403 RUC_NO_AUTORIZADO`.
- Prueba de token expirado (más allá del `exp` + skew) → `401`; token firmado con otra clave → `401` (firma inválida); token con `aud` distinto → `401`.
- Prueba de rotación de clave: con `kid` antiguo y clave pública aún publicada → `401` tras retirarla del JWKS; con clave nueva → `200`.
- Prueba de revocación: `jti` bloqueado → `401` aunque el token no haya expirado.
- Prueba de aislamiento multi-RUC: el cliente A no puede leer, listar, emitir ni enviar GRE del cliente B, ni siquiera indicando el RUC en el cuerpo.
- Prueba de rotación de refresh: reutilizar un refresh token ya usado → `401` y revocación de la familia de tokens.

## Alternativas en evaluación

- **Federar con Keycloak** cuando el cliente exija SSO corporativo y MFA: la API seguirá siendo *resource server*, por lo que el cambio no altera el contrato de los endpoints (verificado con los esquemas de `openapi.yaml`).
- **mTLS** como factor adicional para clientes de alto volumen, si la política de seguridad del cliente lo exige.

## Referencias

- RFC 7519 (JSON Web Token), RFC 7518 (JOSE), RFC 8725 (Buenas prácticas de JWT).
- OAuth 2.0 (RFC 6749) y OpenID Connect Core 1.0.
- R.S. 123-2022-SUNAT, Anexo N.° 14 (UBL 2.1 GRE) y manual de envío de comprobantes de pago electrónico (WS-Security).
- [Contrato de API REST](../contracts/openapi.yaml), [Contrato gRPC](../contracts/grpc_service.proto).
