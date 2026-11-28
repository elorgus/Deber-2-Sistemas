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

---

## HU-35 · Matriz de trazabilidad (2026-11-27)

**Prompt (resumen):** "Resuelve el issue #87 (HU-35): matriz de trazabilidad con la cadena HN → SVC → EVT/INT → diagramas → ADR → RNF → versión, el 100 % de las HN de prioridad M y sin huérfanos. Usa solo los artefactos que existen en `main` (release `diseno-v0.3.0`) y no inventes vínculos."

**Qué se aceptó**
- `docs/trazabilidad/matriz-trazabilidad.md`: catálogo de artefactos con versión, matriz de las 19 HN M, HN S y C, matriz inversa (artefacto → HN) y verificación de CA1 a CA3.

**Qué se corrigió o verificó**
- Con un script se verificó que todos los IDs citados existen en el contrato, el backlog o el documento de RNF, y que cada diagrama de una fila muestra al menos un EVT o INT de esa HN. Las excepciones (C4N1 por roles; CLS-SOLID por clases canónicas) se revisaron a mano y se documentaron en §1.
- Se agregó INT-27 a HN-19 para justificar el vínculo con CLS-SOLID-flota.
- Las versiones se tomaron de la leyenda o cabecera de cada archivo. Las diferencias con el CHANGELOG se reportaron como hallazgos (T-01 a T-06) sin modificar artefactos de otros integrantes.
