# {{Código}} — Diseño técnico: {{Nombre de la funcionalidad}}

| Campo | Valor |
|---|---|
| Código | {{0001}} |
| Feature | {{nombre}} |
| Estado | {{Borrador \| En revisión \| Aprobada}} |
| Fecha | {{YYYY-MM-DD}} |
| Spec de referencia | [spec.md](spec.md) |
| ADRs aplicados | {{ADR-0001, ADR-0002}} |

## 1. Resumen de la solución

{{Dos o tres párrafos: enfoque técnico de la feature, decisiones de diseño principales y
por qué se eligió este camino frente a las alternativas.}}

## 2. Contexto técnico

### 2.1 Restricciones

| ID | Restricción | Origen |
|---|---|---|
| C-1 | {{restricción}} | {{norma / plataforma / decisión previa}} |

### 2.2 Supuestos y dependencias

| ID | Supuesto / dependencia | Impacto si no se cumple |
|---|---|---|
| S-1 | {{supuesto}} | {{impacto}} |

## 3. Arquitectura

```text
{{Diagrama de componentes o flujo en ASCII, con los límites del módulo marcados}}
```

| Componente | Responsabilidad | Ubicación (paquete / ruta) |
|---|---|---|
| {{nombre}} | {{qué hace}} | {{com.examen.gre.xxx}} |

## 4. Decisiones de diseño

### 4.1 {{Título de la decisión}}

- **Contexto**: {{qué obliga a decidir}}
- **Decisión**: {{qué se hace}}
- **Alternativas descartadas**: {{por qué no}}
- **Consecuencias**: {{qué cambia}}

## 5. Modelo de datos

### 5.1 Entidades

| Entidad | Campos | Notas |
|---|---|---|
| {{Gre}} | {{campo: tipo}} | {{reglas}} |

### 5.2 Persistencia

| Tabla / colección | Propósito | Índices |
|---|---|---|
| {{nombre}} | {{qué guarda}} | {{índices}} |

### 5.3 Migraciones

| Versión | Descripción | Script |
|---|---|---|
| {{V1}} | {{qué cambia}} | {{ruta}} |

## 6. Flujo de ejecución

### 6.1 Secuencia principal

| Paso | Actor / Componente | Acción | Resultado |
|---|---|---|---|
| 1 | {{actor}} | {{acción}} | {{resultado}} |

### 6.2 Manejo de errores

| Situación | Excepción / código | Tratamiento | Resultado observable |
|---|---|---|---|
| {{situación}} | {{código}} | {{tratamiento}} | {{resultado}} |

## 7. Integraciones

| Sistema | Protocolo | Autenticación | Campos / operaciones usadas |
|---|---|---|---|
| {{SUNAT WS}} | {{SOAP 1.1 + WS-Security}} | {{certificado digital}} | {{sendPack}} |

## 8. Seguridad

| Aspecto | Aplicación |
|---|---|
| Autenticación | {{esquema}} |
| Autorización | {{modelo}} |
| Datos sensibles | {{cifrado / enmascaramiento}} |
| Auditoría | {{qué se registra}} |

## 9. Observabilidad

| Señal | Tipo | Etiquetas | Uso |
|---|---|---|---|
| {{gre.emitida}} | {{contador}} | {{ruc, tipo}} | {{alerta / panel}} |

## 10. Pruebas

| Nivel | Qué cubre | Herramienta |
|---|---|---|
| Unitario | {{reglas puras}} | {{JUnit 5 + AssertJ}} |
| Integración | {{persistencia, SOAP mock}} | {{Testcontainers}} |
| Contrato | {{OpenAPI / proto}} | {{ Pact / grpc-testing}} |

## 11. Despliegue y operación

| Aspecto | Detalle |
|---|---|
| Variables de entorno | {{lista}} |
| Recursos | {{CPU / memoria / conexiones}} |
| Strategy de despliegue | {{blue/green, rolling}} |
| Plan de rollback | {{pasos}} |

## 12. Cumplimiento normativo

| Requisito normativo | Dónde se implementa |
|---|---|
| {{R.S. 123-2022-SUNAT Anexo N.° 14}} | {{componente}} |

## 13. Trazabilidad con la spec

| Requisito (spec.md) | Diseño | Prueba |
|---|---|---|
| {{RF-1}} | {{componente}} | {{prueba}} |

## 14. Trabajo pendiente / decisiones diferidas

- {{ítem}}
