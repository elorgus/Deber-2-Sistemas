# Contribuciones · P5 (`JessPCL`)

**Rol:** Diseñadora de procesos y calidad · Reposición (SVC-03, flujos A y B, riesgos, requisitos de calidad, informe final)
**Revisor asignado:** P4 (`joshdavr`) · **Reviso a:** P6 (`CamGranda`) por defecto y HU-15 de P7 (BACKLOG §1.2)

> Este archivo es la base de la sección de contribuciones individuales del informe (HU-39, CA1). Se actualiza en cada sprint review.

## Historias de trabajo

| HU | Título | Sprint | Artefactos | PR | Revisor | Estado |
|---|---|---|---|---|---|---|
| HU-08 | Registro inicial de riesgos | S1 | `equipo/P5-JessPCL/HU-08-riesgos.md` | #15 | P4 | ✅ Fusionado (2026-11-23) |
| HU-16 | Flujo A: recepción y actualización de existencias | S2 | `equipo/P5-JessPCL/HU-16-flujo-a-recepcion.md`, `ACT-FA-recepcion`, `SEC-FA-recepcion` | #17 | P4 | 🔍 En revisión |
| HU-17 | Flujo B: riesgo de agotamiento y reposición, ADR-006 | S2 | `equipo/P5-JessPCL/HU-17-flujo-b-reposicion.md`, `ACT-FB-reposicion`, `SEC-FB-reposicion`, `docs/adr/ADR-006-deteccion-riesgo-en-reposicion.md` | #20 | P4 | 🔍 En revisión |

## Hallazgos aportados al equipo

| Origen | IDs | Para quién |
|---|---|---|
| HU-08 | H-01 a H-07 (latencia del flujo B, FAL sin RNF, FAL-02 en UD-1, escalabilidad, separación de funciones, estados del pedido, convenciones del repo) | P1, P2, P3, P5, P6 |
| HU-16 | PC-01 a PC-04 (INT para la réplica de OC, idempotencia de la recepción, lote vencido, recepción mayor a la OC) | P1, P4, P5, P6 |
| HU-17 | PC-05 a PC-08 (INT faltantes del flujo B, compra duplicada, segunda aprobación de OPS-03, regeneración de sugerencias) | P1, P2, P4, P5, P6 |

## Revisiones realizadas

| PR | HU | Autor | Resultado |
|---|---|---|---|
| #18 | HU-07 (anexo de SVC-04 y SVC-05) | P6 | Cambios solicitados (O-1); corregido por P6 con H-2 → aprobado |
| #19 | HU-15 (C4 N3 de SVC-03) | P7 | Comentario temprano: faltan publicación de eventos (INT-11, INT-13), réplicas de EVT-05 y EVT-09, INT-09 y datos propios |
| Issue #7 | HU-06 (contrato v1.0.0) | P1 | Aprobación de P5 con observaciones no bloqueantes (H-01 a H-07, PC-01 a PC-08) |

## Historial

| Fecha | Cambio |
|---|---|
| 2026-11-23 | Creación: HU-08, HU-16 y HU-17 |
| 2026-11-23 | Revisiones de #18 y #19, aprobación del contrato (issue #7); HU-16 y HU-17 pasan a revisión |
