# Registro de Cambios (CHANGELOG)

## diseno-v0.3.0 · 2026-11-27 · Sprint 3

**HU incluidas:** HU-24, HU-25, HU-26, HU-27, HU-28, HU-29, HU-30, HU-31, HU-32, HU-33

**Artefactos**

| Artefacto | Versión | Cambio | HU |
|---|---|---|---|
| docs/arquitectura/matriz-interacciones.md | 1.0.0 | Matriz completa vinculada a los flujos. | HU-24 |
| EST-lote, EST-reserva, EST-sugerencia, EST-orden-compra | 1.0.0 | Diagramas de estado del dominio. | HU-25 |
| EST-pedido, EST-mision-amr, EST-amr, EST-visita-vehiculo, EST-muelle | 1.0.0 | Diagramas de estado de operación. | HU-26 |
| docs/arquitectura/requisitos-calidad.md | 1.0.0 | RNF verificables por atributo. | HU-27 |
| docs/arquitectura/seguridad-roles.md | 1.0.0 | Matriz ROL × operación y OPS. | HU-28 |
| docs/arquitectura/disponibilidad-escalabilidad.md · ADR-009 | 1.0.0 | Análisis de fallos y escalabilidad. | HU-29 |
| docs/diseno/patrones.md · ADR-005 | 1.0.0 | Justificación de patrones y CQRS. | HU-30 |
| docs/diseno/solid.md | 1.0.0 | Evidencia de principios SOLID. | HU-31 |
| docs/dashboard/propuesta-dashboard.md | 2.0.0 | Dashboard v2 (vistas interna y externa). | HU-32 |
| docs/gestion/sprints/sprint-3.md | 1.0.0 | Acta del Sprint 3. | HU-33 |

**ADR aceptados:** ADR-005, ADR-009

---

## diseno-v0.2.0 · Sprint 2

**HU incluidas:** HU-11, HU-13, HU-14, HU-16, HU-17, HU-22

**Artefactos**

| Artefacto | Versión | Cambio | HU |
|---|---|---|---|
| C4N1-contexto (fuente y SVG) | 0.1.1 | Diagrama de contexto de Smart Inventory con siete roles y cinco sistemas externos. Identifica la consulta de indicadores históricos como posterior al MVP. | HU-11 |
| equipo/P5-JessPCL/HU-16-flujo-a-recepcion.md · ACT-FA-recepcion · SEC-FA-recepcion | 1.0.0 | Flujo A: recepción y actualización de existencias | HU-16 |
| equipo/P5-JessPCL/HU-17-flujo-b-reposicion.md · ACT-FB-reposicion · SEC-FB-reposicion | 1.0.0 | Flujo B: riesgo de agotamiento y reposición | HU-17 |
| docs/adr/ADR-006-deteccion-riesgo-en-reposicion.md | — (Propuesto) | La detección de riesgo de agotamiento pertenece a Reposición | HU-17 |
| docs/dashboard/propuesta-dashboard.md · DSH-planta-interna · DSH-planta-externa (fuente y SVG) | 1.0.0 | Propuesta inicial de Dashboard v1 (vistas VL-01 a VL-07, frescura y acciones hacia dueños) y bocetos de planta interna y externa | HU-22 |
| docs/arquitectura/propiedad-de-datos.md | 0.1.0 | Matriz detallada de propiedad de datos y réplicas locales. | HU-13 |
| C4N3-SVC02-inventario (fuente y SVG) | 0.1.0 | Diagrama de componentes de Inventario: puertos, FEFO, réplicas, persistencia con Outbox y detección de vencimientos. | HU-14 |
| docs/adr/ADR-003-reserva-fefo-en-inventario.md | — (Propuesto) | La reserva de stock y la selección FEFO pertenecen a Inventario | HU-14 |

**ADR aceptados:** ninguno

---

## diseno-v0.1.0 · 2026-11-23 · Sprint 1

**HU incluidas:** HU-01 a HU-10

**Artefactos**

| Artefacto | Versión | Cambio | HU |
|---|---|---|---|
| docs/ARQUITECTURA_BASE.md | 1.0.0 | Contrato maestro aprobado (tag contrato-v1.0.0) | HU-06 |
| docs/BACKLOG.md | 1.0.0 | Product backlog priorizado, plan de sprints y estructura | HU-04, HU-05 |
| CONTRIBUTING.md | 1.0.1 | DoR, DoD y checklist ampliada | HU-09 |
| docs/VERSIONAMIENTO.md | 1.0.0 | Política de versionamiento | HU-09 |
| equipo/P5-JessPCL/HU-08-riesgos.md | 1.0.0 | Documentación de riesgos | HU-08 |
| equipo/P3-RichJ-100/HU-10-acta-sprint-1.md | 1.0.0 | Acta del Sprint 1 | HU-10 |

**ADR aceptados:** ninguno
