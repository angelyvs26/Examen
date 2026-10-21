# {{Código}} — Tareas: {{Nombre de la funcionalidad}}

| Campo | Valor |
|---|---|
| Código | {{0001}} |
| Feature | {{nombre}} |
| Spec | [spec.md](spec.md) |
| Diseño | [design.md](design.md) |
| Estado | {{Pendiente \| En curso \| Completada}} |

## Convenciones

- Los identificadores siguen el patrón `T-<bloque>-<secuencia>` y son estables: no se reutilizan.
- Cada tarea indica su requisito de origen (`RF-…`) y la prueba que la cierra.
- Estimaciones en puntos de historia (no horas).
- Estado: `[ ]` pendiente · `[~]` en curso · `[x]` completada · `[-]` descartada.

## Fase 0 — Fundaciones

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-00-01 | {{tarea}} | {{RF-1}} | {{1}} | — | [ ] |

## Fase 1 — Dominio y contratos

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-10-01 | {{tarea}} | {{RF-2}} | {{2}} | T-00-01 | [ ] |

## Fase 2 — Persistencia

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-20-01 | {{tarea}} | {{RF-3}} | {{2}} | T-10-01 | [ ] |

## Fase 3 — Integración

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-30-01 | {{tarea}} | {{RF-4}} | {{3}} | T-20-01 | [ ] |

## Fase 4 — Endpoints y experiencia de uso

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-40-01 | {{tarea}} | {{RF-5}} | {{2}} | T-30-01 | [ ] |

## Fase 5 — Observabilidad, pruebas y documentación

| ID | Tarea | Req. | Est. | Depende de | Estado |
|---|---|---|---|---|---|
| T-50-01 | {{tarea}} | {{RF-6}} | {{2}} | T-40-01 | [ ] |

## Resumen

| Fase | Tareas | Puntos | Completadas |
|---|---|---|---|
| 0 — Fundaciones | {{n}} | {{n}} | 0 |
| 1 — Dominio y contratos | {{n}} | {{n}} | 0 |
| 2 — Persistencia | {{n}} | {{n}} | 0 |
| 3 — Integración | {{n}} | {{n}} | 0 |
| 4 — Endpoints | {{n}} | {{n}} | 0 |
| 5 — Observabilidad y pruebas | {{n}} | {{n}} | 0 |
| **Total** | **{{n}}** | **{{n}}** | **0** |

## Definición de terminado (DoD)

Una feature está completa cuando:

- [ ] Todas las tareas de la tabla están en `[x]` o `[-]`.
- [ ] Cada requisito `RF-…` de [spec.md](spec.md) tiene al menos una prueba que lo cubre.
- [ ] El XML generado por la feature es aceptado por el validador oficial de SUNAT (para features de emisión).
- [ ] Los contratos de [contracts/](../../contracts/) se actualizaron si cambió la API.
- [ ] Métricas, logs y alertas definidos en el diseño están implementados.
- [ ] La documentación de uso (ejemplo de request/response) está actualizada.

## Fuera de alcance

- {{tarea explícitamente no incluida en esta feature}}
