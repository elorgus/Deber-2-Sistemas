# Registro de uso de IA · P5 (`JessPCL`)

> Exigido por BACKLOG §9, paso 3: el prompt usado y un resumen de lo que se aceptó o se corrigió. La IA no reemplaza la revisión cruzada: cada artefacto pasa por la checklist de `CONTRIBUTING.md` y lo revisa P4.

**Herramienta:** Claude (Anthropic)
**Contexto entregado en todas las sesiones:** enunciado del deber, `docs/ARQUITECTURA_BASE.md` v1.0.0 y `docs/BACKLOG.md` v1.0.0.

---

## HU-08 · Registro inicial de riesgos (2026-11-23)

**Prompt (resumen):** "Eres asistente de diseño de sistemas para Smart Inventory. Usa EXACTAMENTE los IDs del contrato. No diseñes APIs ni CI/CD. Soy P5 y mi HU es HU-08: redacta la matriz de riesgos iniciales en `equipo/P5-JessPCL/HU-08-riesgos.md` (mínimo 5 riesgos técnicos, de equipo y de alcance, con probabilidad, impacto y mitigación), revisa §11.2 y §11.3 para alinear los riesgos con los RNF y FAL, y dame el texto del PR."

**Qué se aceptó**
- 18 riesgos (RIE-01 a RIE-18), con escala de probabilidad e impacto de 1 a 3, mapa de calor y columna "Última revisión".
- Tablas de alineación FAL → RNF → riesgo.
- Hallazgos H-01 a H-07.

**Qué se corrigió o verificó**
- Se subió de 5 a 18 riesgos, porque HU-08 CA1 exige al menos 10.
- La ubicación del archivo se ajustó a `CONTRIBUTING.md` (carpeta personal) después de revisar el repositorio; la IA había propuesto primero `docs/gestion/riesgos.md`, según el BACKLOG.
- Se corrigieron los enlaces relativos al contrato (`../../docs/`) y se agregó el issue #9.
- Se agregó H-07, la diferencia de convenciones entre `CONTRIBUTING.md` y el BACKLOG.
- Se verificó con un script que todos los IDs existen en el contrato.

---

## HU-16 · Flujo A: recepción (2026-11-23)

**Prompt (resumen):** "Genera los entregables de HU-16 con sus comandos, paso a paso."

**Qué se aceptó**
- `ACT-FA-recepcion` y `SEC-FA-recepcion` en PlantUML, con su exportación SVG.
- El texto del flujo con validaciones V1 a V5, caminos A1 a A4 y trazabilidad.
- Los puntos abiertos PC-01 a PC-04.

**Qué se corrigió o verificó**
- Se revisó visualmente cada diagrama exportado; se ajustaron etiquetas largas y la numeración de los mensajes.
- Se verificó que cada flecha de la secuencia use una INT del contrato. La entrega de EVT-19 y EVT-20 a la réplica de OC no tiene INT, así que quedó marcada como PC-01.
- Se probaron los comandos de git sobre una copia del repositorio antes de usarlos.

---

## HU-17 · Flujo B: reposición y ADR-006 (2026-11-23)

**Prompt (resumen):** "Genera los entregables de HU-17, además de `CONTRIBUCIONES.md` y `registro-ia.md`, con los comandos paso a paso."

**Qué se aceptó**
- `ACT-FB-reposicion` y `SEC-FB-reposicion` con su exportación SVG.
- El texto del flujo B y `ADR-006` con tres opciones (detección en SVC-02, en TRX-04 o en SVC-03).
- Los puntos abiertos PC-05 a PC-08.

**Qué se corrigió o verificó**
- En el diagrama de actividad se reordenaron las ramas paralelas para que no se cruzaran las líneas.
- Las interacciones sin ID en §7.2 se marcaron como «INT por definir» en lugar de inventarles un número.
- Se verificó que los nombres de estrategias, estados y puertos coincidan con §9 y §10.

---

## Historial

| Fecha | Cambio |
|---|---|
| 2026-11-23 | Registro de HU-08, HU-16 y HU-17 |
