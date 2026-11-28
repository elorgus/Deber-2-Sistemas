# Contribuciones · P5 (`JessPCL`)

**Rol:** Diseñadora de procesos y calidad · Reposición (SVC-03, flujos A y B, riesgos, requisitos de calidad, informe final)
**Revisor asignado:** P4 (`joshdavr`) · **Reviso a:** P6 (`CamGranda`) por defecto y HU-15 de P7 (BACKLOG §1.2)

> Este archivo es la base de la sección de contribuciones individuales del informe (HU-39, CA1). Se actualiza en cada sprint review.

## Historias de trabajo

| HU | Título | Sprint | Artefactos | PR · Issue | Revisor | Estado |
|---|---|---|---|---|---|---|
| HU-08 | Registro inicial de riesgos | S1 | `docs/gestion/riesgos.md` (v1.1.1) | #15 · #9 | P4 | ✅ Fusionado (2026-11-23) |
| HU-16 | Flujo A: recepción y actualización de existencias | S2 | `docs/arquitectura/flujos/flujo-a-recepcion.md` (v1.0.1), `ACT-FA-recepcion`, `SEC-FA-recepcion` | #17 · #47 | P4 | ✅ Aprobado y fusionado (2026-11-24) |
| HU-17 | Flujo B: riesgo de agotamiento y reposición, ADR-006 | S2 | `docs/arquitectura/flujos/flujo-b-reposicion.md` (v1.1.0), `ACT-FB-reposicion`, `SEC-FB-reposicion`, `docs/adr/ADR-006-deteccion-riesgo-en-reposicion.md` | #20 · #48 | P4 | ✅ Aprobado y fusionado (2026-11-24) |
| HU-27 | Requisitos de calidad verificables | S3 | `docs/arquitectura/requisitos-calidad.md` (33 RNF, v1.0.1) | #59 | P4 | ✅ Aprobado y fusionado (2026-11-27) |
| HU-39 | Informe final | S4 | `entregables/informe/Informe-Smart-Inventory.docx` (v0.4.0, borrador) | #62 (draft) | P4 | 🟡 En curso: se actualiza a medida que se fusionan las HU |
| — | Ubicación según BACKLOG §8.1 | S3 | Flujos A y B y sus diagramas (#79, cierra PC-08); HU-08 y HU-27 (este PR) | #79 | P4 | ✅ #79 fusionado (2026-11-27) |

## Hallazgos aportados al equipo

| Origen | IDs | Para quién |
|---|---|---|
| HU-08 | H-01 a H-07 (latencia del flujo B, FAL sin RNF, FAL-02 en UD-1, escalabilidad, separación de funciones, estados del pedido, convenciones del repo) | P1, P2, P3, P5, P6 |
| HU-16 | PC-01 a PC-04 (INT para la réplica de OC, idempotencia de la recepción, lote vencido, recepción mayor a la OC) | P1, P4, P5, P6 |
| HU-17 | PC-05 a PC-08 (INT faltantes del flujo B, compra duplicada, segunda aprobación de OPS-03, regeneración de sugerencias) | P1, P2, P4, P5, P6 |
| HU-27 | PC-09 a PC-11 (CE-03 a ≤ 3 s, SUP-04 horario operativo, registrar los RNF nuevos en el contrato). Cierra H-01, H-02 y H-04 | P1, P2 |

## Revisiones realizadas

| PR | HU | Autor | Resultado |
|---|---|---|---|
| #18 | HU-07 (anexo de SVC-04 y SVC-05) | P6 | Cambios solicitados (O-1); corregido por P6 con H-2 → aprobado y fusionado (2026-11-23) |
| #19 | HU-15 (C4 N3 de SVC-03) | P7 | Comentario temprano con 5 ajustes (2026-11-23); P7 los incorporó en la v0.2.0 → **aprobado** (2026-11-24) |
| #63 | HU-18 (flujo C) | P6 | Cambios solicitados (3 bloqueantes y 3 sugerencias); P6 los corrigió y regeneró los SVG → **aprobado** y fusionado (2026-11-26) |
| #66 | HU-19 (C4 N3 de Flota y ADR-007) | P6 | Cambios solicitados (ADR-007 sin opciones, título, estado, ruta del diagrama); P6 los corrigió → **aprobado** y fusionado (2026-11-26) |
| Contrato | HU-06 (contrato v1.0.0) | P1 | Aprobación de P5 con observaciones no bloqueantes (H-01 a H-07, PC-01 a PC-08); el acta del Sprint 1 (HU-10) registra la aprobación de los 7 |

## Historial

| Fecha | Cambio |
|---|---|
| 2026-11-23 | Creación: HU-08, HU-16 y HU-17 |
| 2026-11-23 | Revisiones de #18 y #19, aprobación del contrato; HU-16 y HU-17 pasan a revisión |
| 2026-11-24 | HU-16 y HU-17 fusionadas; aprobación del #19; HU-27 en revisión |
| 2026-11-27 | HU-27 fusionada; revisiones de #63 y #66; HU-39 en curso; los documentos pasan a las rutas de BACKLOG §8.1 |
