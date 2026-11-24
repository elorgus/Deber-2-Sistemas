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

## HU-27 · Requisitos de calidad verificables (2026-11-24)

**Prompt (resumen):** "Genera los entregables de HU-27 con los comandos paso a paso: convierte los RNF del contrato en escenarios medibles y completa los que faltan."

**Qué se aceptó**
- 33 RNF en formato de escenario (15 semilla del contrato y 18 nuevos), cada uno con fuente, estímulo, entorno, respuesta y medida.
- La validación de los SUP del contrato y el supuesto nuevo SUP-04 (horario operativo).
- La tabla de trazabilidad y los puntos abiertos PC-09 a PC-11.
- La actualización de HU-08 a v1.1.0: cambia el estado de dos riesgos y la cobertura de fallos.

**Qué se corrigió o verificó**
- CE-03 (≤ 5 s) más CE-01 no caben en los 5 s que pide HN-09 (H-01). La IA no cambió el contrato: lo dejó como propuesta (PC-09, CE-03 a ≤ 3 s), porque los cambios al contrato se aprueban por ARQUITECTURA_BASE §0.3.
- RNF-DIS-04 usa "horario operativo" sin definirlo (H-04). SUP-04 quedó como supuesto propuesto (PC-10), no como ID oficial.
- Se verificó con un script que todos los IDs existen en el contrato o están marcados como nuevos.
- `CHANGELOG.md` no se tocó, porque lo consolida P3 (VERSIONAMIENTO §7). Las filas se propusieron en la descripción del PR.
- El conflicto de `CHANGELOG.md` al traer `main` se resolvió conservando la versión de `main`.

---

## HU-39 · Informe final (2026-11-24)

**Prompt (resumen):** "Arma el informe final en Word con lo que ya está en `main` y actualízalo cuando se fusionen más HU."

**Qué se aceptó**
- `docs/informe/Informe-Smart-Inventory.docx`, generado con un script, con la estructura alineada a la rúbrica y a BACKLOG §5.2.
- Las figuras se exportaron desde las fuentes del repositorio y conservan su leyenda `ID · vX.Y.Z · fecha · autor (Px) · HU-NN`.
- Un recuadro "PENDIENTE" con el responsable en cada sección cuya HU no está fusionada.
- La v0.1.0 con el contenido hasta el Sprint 2 y HU-27. La v0.2.0 agrega HU-13, HU-15, HU-21 y HU-22.

**Qué se corrigió o verificó**
- Solo se incluyó contenido fusionado en `main` o aprobado; para las HU pendientes no se inventó contenido.
- Se renderizó a PDF y se revisó página por página: tablas que no se parten, tamaño de las figuras y numeración de figuras.
- Se validó el archivo de Word y se comprobó con un script que todos los IDs existen en el contrato.
- Los nombres completos se tomaron del historial de commits y quedan pendientes de confirmar por cada integrante.

---

## Revisiones cruzadas con apoyo de IA (2026-11-23 y 2026-11-24)

**Uso:** comparar los IDs de cada PR con el contrato y aplicar la checklist de `plantillas/plantilla-revision-pr.md`. La decisión de aprobar o pedir cambios fue de P5.

- PR #18 (HU-07, P6) y PR #19 (HU-15, P7): revisados y aprobados.
- Rama de HU-18 (P6), antes de abrir el PR: los IDs coinciden con §6.2, §7.2 y §9. Se le avisó a P6 que faltan los diagramas de CA1, la ruta de los diagramas según `CONTRIBUTING.md` e INT-29 en el paso 9.

---

## Historial

| Fecha | Cambio |
|---|---|
| 2026-11-23 | Registro de HU-08, HU-16 y HU-17 |
| 2026-11-24 | Registro de HU-27, HU-39 y de las revisiones cruzadas |
