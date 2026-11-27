# Registro de uso de IA · P4 (`joshdavr`)

> Exigido por BACKLOG §9, paso 3: el prompt usado y un resumen de lo que se aceptó o se corrigió. La IA no reemplaza la revisión cruzada: cada artefacto pasa por la checklist de `CONTRIBUTING.md` y lo revisa P6.

**Herramienta:** Claude (Anthropic)
**Contexto entregado:** `docs/ARQUITECTURA_BASE.md` v1.0.0, `docs/BACKLOG.md` v1.0.0, `CONTRIBUTING.md`, `docs/VERSIONAMIENTO.md` y los diagramas ya aprobados en `diagramas/` (estilo y leyenda).

---

## HU-25 · Diagramas de estado (2026-11-27)

**Prompt (resumen):** "Resuelve el issue #69 (HU-25): diagramas de estado de Lote, Reserva, Sugerencia y Orden de compra, con los criterios CA1 a CA3. Usa exactamente los estados de §9 y los IDs del contrato, PlantUML con la leyenda de §7.3, y no inventes lo que el contrato no define."

**Qué se aceptó**
- Cuatro diagramas PlantUML (`EST-lote`, `EST-reserva`, `EST-sugerencia`, `EST-orden-compra`) con sus exportaciones SVG.
- Documento `HU-25-diagramas-estado.md` con la tabla de transiciones, las transiciones inválidas y las preguntas abiertas Q-1 a Q-5.

**Qué se corrigió o verificó**
- Se tomó el disparador Enviada → RecibidaParcial de INT-09 y HN-12, porque §9 no lo indica.
- La transición Enviada → Cerrada (HN-12) no está en §9: se dejó punteada como «PROPUESTA» (Q-2) en lugar de usarla como oficial.
- La cancelación de la OC y la expiración de la reserva no tienen disparador en el contrato: se registraron como Q-3 y Q-4 sin inventar nombres.
- Se verificó que cada ID citado (EVT, INT, TX, OPS, ROL, FAL, HN) existe en el contrato o en el backlog.
- Las exportaciones SVG se generaron con Graphviz a partir del mismo contenido de las fuentes `.puml`. Si el revisor prefiere el estilo de PlantUML, basta con volver a exportar las fuentes.
