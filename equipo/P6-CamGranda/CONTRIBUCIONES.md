# Contribuciones de P6 · CamGranda

| Campo | Valor |
|---|---|
| Documento | `equipo/P6-CamGranda/CONTRIBUCIONES.md` |
| Integrante | P6 · `CamGranda` |
| Rol | Diseñador de operación (Pedidos y Flota): `SVC-04` Pedidos y Fulfillment y `SVC-05` Flota y Operación |
| Revisora cruzada | P5 (`JessPCL`) · P6 revisa a P4 (`joshdavr`) |
| Fecha | 2026-11-27 |
| Uso | Insumo del informe final (HU-39, CA1) |

---

## 1. Historias entregadas con PR propio

| HU | Título | Issue | PR | Estado | Revisor | Artefactos |
|---|---|---|---|---|---|---|
| HU-07 | Glosario y modelo conceptual de dominio · aporte de apoyo: validación de propiedad de datos de `SVC-04` y `SVC-05` | #8 (relacionado) | #18 | Fusionado, aprobado | P5 | `equipo/P6-CamGranda/HU-07-anexo-pedidos-flota.md` |
| HU-18 | Flujo C: reserva FEFO, preparación y despacho | #49 | #63 | Fusionado, aprobado | P5 | `docs/arquitectura/flujos/flujo-c-reserva-despacho.md`, `ACT-FC-reserva-despacho`, `SEC-FC-reserva-despacho` (1.0.0) |
| HU-19 | C4 Nivel 3: Flota y Operación y ADR-007 | #50 | #66 | Fusionado, aprobado | P5 | `diagramas/c4/src/C4N3-SVC05-flota.puml`, `diagramas/c4/export/C4N3-SVC05-flota.svg` (1.0.0), `docs/adr/ADR-007-flota-como-unidad-separada.md` (*Propuesto*) |

## 2. Historias trabajadas e integradas por coordinación

| HU | Título | Issue o PR | Estado | Artefactos |
|---|---|---|---|---|
| HU-24 | Matriz de interacciones completa (asumida en suplencia) | Integrada en el PR #83 (P1, `elorgus`) | Integrada | `docs/arquitectura/matriz-interacciones.md` (1.0.0) |
| HU-26 | Diagramas de estado: Pedido, Misión AMR, AMR, Visita de vehículo y Muelle | Integrada en el PR #83 (P1, `elorgus`) | Integrada | `EST-pedido`, `EST-mision-amr`, `EST-amr`, `EST-visita-vehiculo`, `EST-muelle` (1.0.0) |
| HU-38 | Recorrido integral del flujo vertical | Issue #90 · PR #106 | En revisión (Sprint 4) | `diagramas/secuencia/src/SEC-VERTICAL-recorrido-integral.puml`, `diagramas/secuencia/export/SEC-VERTICAL-recorrido-integral.svg`, `docs/arquitectura/flujos/recorrido-integral.md` |

## 3. Revisiones realizadas a compañeros

| HU | Título | Autor | Issue o PR | Resultado |
|---|---|---|---|---|
| HU-14 | C4 Nivel 3 de Inventario y ADR-003 | P4 (`joshdavr`) | PR #56 | Aprobado |
| HU-25 | Diagramas de estado: Lote, Reserva, Sugerencia y Orden de compra | P4 (`joshdavr`) | PR #80 | Aprobado |
| HU-35 | Matriz de trazabilidad | P4 (`joshdavr`) | Issue #87 · PR #101 | Aprobado y fusionado |

## 4. Retroalimentación recibida de P5 y aplicada

- **HU-07 (PR #18):** corregí la observación O-1 (la liberación por expiración no es del lado conservador; nuevo hallazgo H-2), reclasifiqué `INT-22` e `INT-27` como salidas hacia `EXT-03` y `EXT-02`, añadí `EVT-35` por `INT-29` en `EnDespacho` → `Despachado`, y desmarqué la casilla de aprobación de la checklist.
- **HU-18 (PR #63):** marqué como «INT por definir» la cancelación y la actualización de `VL-03` (`INT-08` solo cubre `VL-01` y `VL-06`), añadí la sección «Puntos abiertos» con Q-1, agregué `INT-29` al paso 9, unifiqué los lotes del ejemplo FEFO y ubiqué `EVT-29` antes de la telemetría en `ACT-FC` y `SEC-FC`.
- **HU-19 (PR #66):** reescribí ADR-007 con la plantilla (dos opciones consideradas, decisión y consecuencias), usé el título exacto de §12, dejé el estado en *Propuesto*, eliminé marcas de la IA y moví el diagrama a `diagramas/c4/`.

## 5. Puntos abiertos que levanté sobre el contrato

Ninguno modifica `ARQUITECTURA_BASE.md`; todos siguen el proceso de §0.3.

| ID | Punto | Dónde está documentado |
|---|---|---|
| Q-1 | §9 no define cómo sale un pedido de `EnEspera`. | Flujo C, sección 7 |
| H-1 | §3.6 dice que `SVC-06` consume `EVT-32`, pero §5, §6.1 y §6.2 limitan la señal a `TRX-04`. | Anexo de HU-07 |
| H-2 | La reserva también se libera por expiración (`EVT-10`) y `SVC-04` no consume ese evento. | Anexo de HU-07 y Flujo C |
| INT por definir | Cancelar pedido (`TRX-02` → `SVC-04`, `OPS-05`) y actualización de `VL-02` y `VL-03` en `TRX-04`. | Flujo C, recorrido integral y matriz de interacciones |
