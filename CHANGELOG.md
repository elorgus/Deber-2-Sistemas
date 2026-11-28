# Registro de Cambios (CHANGELOG)

## diseno-v0.3.0 · Sprint 3 (En progreso)

**HU incluidas:** HU-25, HU-28, HU-30, HU-31

**Artefactos**

| Artefacto | Versión | Cambio | HU |
|---|---|---|---|
| docs/diseno/solid.md · CLS-SOLID-inventario · CLS-SOLID-reposicion · CLS-SOLID-flota (fuente y SVG) | 0.1.0 | Documenta los cinco principios SOLID con pseudocódigo, diagramas de clases y un cambio futuro por principio. | HU-31 |
| docs/diseno/patrones.md | 0.1.1 | Justifica seis patrones de Smart Inventory y enlaza sus diagramas de evidencia. | HU-30 |
| docs/adr/ADR-005-proyeccion-de-lectura-cqrs.md | — (Propuesto) | Documenta la separación entre consultas del dashboard y escrituras del negocio, y los estados de frescura de las vistas. | HU-30 |
| EST-lote · EST-reserva · EST-sugerencia · EST-orden-compra (fuente y SVG) · equipo/P4-joshdavr/HU-25-diagramas-estado.md | 1.0.0 | Diagramas de estado de Lote, Reserva, Sugerencia de reposición y Orden de compra, con disparadores, transiciones inválidas y preguntas abiertas (Q-1 a Q-5) | HU-25 |
| docs/arquitectura/disponibilidad-escalabilidad.md · docs/adr/ADR-009-replicas-locales-de-referencia.md | 0.1.0 | Análisis de fallos (FAL-01 a FAL-09), tabla de escalabilidad (RNF-ESC-01) y decisión de réplicas locales. | HU-29 |
| docs/arquitectura/seguridad-roles.md | 1.0.0 | Especificación de seguridad: roles ROL-01 a ROL-07, matriz ROL × operación, operaciones sensibles OPS-01 a OPS-08, umbral de USD 5.000 en OPS-03, auditoría, separación de funciones y secretos | HU-28 |

**ADR aceptados:** ninguno

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
