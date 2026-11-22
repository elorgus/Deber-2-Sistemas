# Guía de Contribución — Smart Inventory

Este documento define cómo trabaja el equipo para garantizar que los artefactos sean consistentes, trazables y cumplan con lo exigido en el deber.

## 1. Flujo de trabajo en GitHub

1. **Rama de trabajo:** Crea una rama a partir de `main` con el formato: `feat/HU-nn-descripcion-corta` (ej. `feat/HU-02-vision-producto`).
2. **Commits:** Usa mensajes claros en español (ej. `docs(HU-02): agrega visión del producto`).
3. **Pull Request (PR):**
   - Título del PR: `HU-nn: Descripción breve`.
   - Asigna como revisor a tu pareja de revisión cruzada (ver `docs/BACKLOG.md` §1.2).
   - Completa la checklist del PR (abajo).
4. **Aprobación:** Se requiere al menos 1 aprobación para fusionar. Si modificas `ARQUITECTURA_BASE.md`, requiere además la aprobación de P1 (`elorgus`) y del diseñador del módulo afectado.
5. **Fusión:** Solo el autor del PR (o P3) puede fusionar una vez aprobado.

## 2. Ubicación de los archivos

- Tus documentos de trabajo, borradores y actas van en tu carpeta personal: `equipo/P?-tu-usuario/`.
- Los diagramas fuente (`.drawio`, `.puml`, `.mmd`) van en `docs/diagramas/fuente/`.
- Los diagramas exportados (`.png`, `.svg`) van en `docs/diagramas/export/`.
- Los ADR van en `docs/adr/ADR-nnn-titulo.md`.
- El informe consolidado va en `docs/informe/`.

## 3. Reglas de diseño y nomenclatura

- Usa **exactamente** los IDs del contrato (`SVC-xx`, `TRX-xx`, `EVT-xx`, `INT-xx`, `VL-xx`, `RNF-xx`, `ADR-xxx`).
- Todo diagrama debe citar los IDs que aparecen en él.
- No inventes nombres nuevos: si algo falta, propón un cambio vía PR + ADR "Propuesto".
- **Fuera de alcance:** No incluyas endpoints, payloads, esquemas OpenAPI, CI/CD ni pruebas.

## 4. Checklist del Pull Request (Definition of Done)

Copia esto en la descripción de cada PR:

```markdown
## Checklist
- [ ] El artefacto está en la carpeta correcta (`equipo/P?-...` o `docs/...`).
- [ ] Se citan los IDs correctos del contrato (`SVC`, `EVT`, `INT`, `ADR`).
- [ ] No se incluyen APIs, payloads, CI/CD ni pruebas.
- [ ] Se ha actualizado el `CHANGELOG.md` (si aplica).
- [ ] He asignado a mi revisor cruzado y este ha aprobado.
- [ ] El issue correspondiente ha sido referenciado (`Closes #nn`).
