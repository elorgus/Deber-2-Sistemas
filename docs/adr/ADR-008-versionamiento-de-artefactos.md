# ADR-008: Versionamiento de artefactos de diseño

**Estado:** Aceptado
**Fecha:** [Pon la fecha de hoy]
**Decisores:** P3 (Scrum Master)

## Contexto
El equipo produce diagramas, documentos y ADR. Necesitamos una forma clara de saber qué cambió, cuándo y por qué, para evitar confusiones entre versiones del proyecto.

## Opciones consideradas
1. **Opción A:** No versionar y sobrescribir archivos. (Descartada: se pierde el historial).
2. **Opción B:** Usar el versionamiento semántico (MAJOR.MINOR.PATCH) para cada artefacto y tags por sprint. (Elegida).

## Decisión
Se adopta el versionamiento semántico descrito en `docs/versionamiento/politica-versionamiento.md`, usando tags `diseno-vX.Y.Z` al final de cada sprint y una leyenda obligatoria en cada diagrama.

## Consecuencias
- **Positivas:** Historial claro, fácil de auditar, los cambios se entienden.
- **Negativas:** Requiere disciplina del equipo para actualizar las leyendas y el CHANGELOG en cada PR.