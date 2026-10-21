# {{Código}} — {{Nombre de la funcionalidad}}

| Campo | Valor |
|---|---|
| Código | {{0001}} |
| Módulo | {{nombre del módulo}} |
| Estado | {{Borrador \| En revisión \| Aprobada \| Implementada \| Retirada}} |
| Versión | {{1.0.0}} |
| Fecha | {{YYYY-MM-DD}} |
| Responsables | {{nombres / roles}} |
| Depende de | {{otras features}} |
| Bloquea a | {{otras features}} |
| ADRs aplicables | {{ADR-0001, ADR-0002}} |

## 1. Objetivo

{{Una o dos frases: qué resuelve la funcionalidad y por qué ahora.}}

## 2. Alcance

### 2.1 Incluido

- {{capacidad de negocio que sí se entrega}}

### 2.2 Excluido

- {{capacidad explícitamente fuera de alcance, con la razón}}

## 3. Antecedentes y contexto

{{Normativa que origina la funcionalidad (R.S. 123-2022-SUNAT, catálogos SUNAT, manuales del
contribuyente), pain points del usuario y estado previo del proceso.}}

## 4. Definitions / Glosario

| Término | Definición |
|---|---|
| {{término}} | {{definición}} |

## 5. Personas y permisos

| Actor | Descripción | Permisos |
|---|---|---|
| {{actor}} | {{qué hace}} | {{permisos}} |

## 6. Historias de usuario

### US-{{n}} — {{título}}

**Como** {{persona}}
**Quiero** {{capacidad}
**Para** {{beneficio}

**Criterios de aceptación**

| # | Criterio (Given / When / Then) | Prioridad |
|---|---|---|
| 1 | Given {{contexto}}, When {{acción}}, Then {{resultado esperado}} | {{Must \| Should \| Could}} |

**Reglas de negocio**

| ID | Regla | Origen (norma / ítem de validación SUNAT) |
|---|---|---|
| RN-{{n}} | {{regla}} | {{fuente}} |

**Validaciones / errores esperados**

| Código | Condición | Tipo de retorno | Comportamiento |
|---|---|---|---|
| {{código SUNAT o interno}} | {{condición}} | {{ERROR \| OBSERV \| REST}} | {{qué hace el sistema}} |

## 7. Flujo principal

{{Descripción del camino feliz, en pasos numerados, incluyendo puntos de decisión.}}

## 8. Flujos alternativos y de error

| ID | Situación | Disparo | Respuesta del sistema | Estado resultante |
|---|---|---|---|---|
| {{FA-1}} | {{situación}} | {{condición}} | {{respuesta}} | {{estado}} |

## 9. Requisitos funcionales

| ID | Requisito | Historia / regla que lo origina |
|---|---|---|
| RF-{{n}} | {{requisito}} | {{US-x / RN-x}} |

## 10. Requisitos no funcionales

| ID | Categoría | Requisito | Métrica / criterio de aceptación |
|---|---|---|---|
| RNF-{{n}} | {{Rendimiento \| Seguridad \| Disponibilidad \| Trazabilidad \| Compatibilidad}} | {{requisito}} | {{métrica}} |

## 11. Modelo de datos

{{Entidades, atributos, cardinalidades y reglas de negocio asociadas. Si aplica, indicar
el esquema lógico y las invariantes que la base de datos debe garantizar.}}

## 12. Contratos

| Contrato | Ubicación | Descripción |
|---|---|---|
| {{nombre}} | {{enlace o ruta}} | {{qué define}} |

## 13. Criterios de aceptación de la feature

- [ ] {{criterio verificable 1}}
- [ ] {{criterio verificable 2}}

## 14. Trazabilidad

| Requisito | Artefacto | Prueba |
|---|---|---|
| {{RF-1}} | {{clase / endpoint}} | {{prueba}} |

## 15. Riesgos y supuestos

| ID | Riesgo / supuesto | Impacto | Mitigación |
|---|---|---|---|
| {{R-1}} | {{descripción}} | {{alto / medio / bajo}} | {{mitigación}} |

## 16. Preguntas abiertas

- {{pregunta y responsable}}
