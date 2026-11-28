# Auditoría de versionamiento

| Campo | Valor |
|---|---|
| Documento | `docs/trazabilidad/auditoria-versionamiento.md` |
| HU | HU-36 |

## 1. Verificación de Versionamiento
- Todos los diagramas (C4, Actividad, Secuencia, Estados, Clases) cuentan con su código fuente en `src/` y su exportación en `export/`.
- Se aplicó la convención de nombres `TIPO-ALCANCE-descripcion` en todos los archivos.
- Las leyendas de los diagramas incluyen ID, versión, fecha, autor y HU.

## 2. Verificación de ADRs
- Todos los ADR (ADR-001 a ADR-009) tienen estado *Propuesto* o *Aceptado* en el índice `docs/adr/README.md`.

## 3. Historial de Cambios
- El archivo `CHANGELOG.md` registra los cierres de los 4 sprints (`v0.1.0`, `v0.2.0`, `v0.3.0`, `v1.0.0`).
