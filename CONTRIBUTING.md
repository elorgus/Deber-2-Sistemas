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
- Los diagramas fuente (`.puml`, `.drawio`, `.mmd`) y exportados (`.png`, `.svg`) se organizan en la carpeta `diagramas/` en la raíz del proyecto (ver `docs/BACKLOG.md` §8.1):
  - Fuentes en `diagramas/<tipo>/src/` (ej. `diagramas/c4/src/`, `diagramas/dashboard/src/`).
  - Exportaciones en `diagramas/<tipo>/export/`.
- Los ADR van en `docs/adr/ADR-nnn-titulo.md`.
- El informe consolidado y la presentación final van en `entregables/informe/` y `entregables/presentacion/`.

## 3. Reglas de diseño y nomenclatura

- Usa **exactamente** los IDs del contrato (`SVC-xx`, `TRX-xx`, `EVT-xx`, `INT-xx`, `VL-xx`, `RNF-xx`, `ADR-xxx`).
- Todo diagrama debe citar los IDs que aparecen en él.
- No inventes nombres nuevos: si algo falta, propón un cambio vía PR + ADR "Propuesto".
- **Fuera de alcance:** No incluyas endpoints, payloads, esquemas OpenAPI, CI/CD ni pruebas.

## 4. Definition of Ready (antes de empezar una HU)

Una HU pasa a *En curso* solo si cumple todo:

- [ ] Tiene **responsable y revisor** asignados.
- [ ] Sus **criterios de aceptación son verificables**.
- [ ] Está vinculada a una **HN o a un módulo (SVC/TRX)** y a un **criterio de la rúbrica**.
- [ ] Los **artefactos de los que depende ya están aprobados** (o se acordó trabajar en paralelo).
- [ ] **Cabe en el sprint** según la capacidad declarada.

## 5. Definition of Done (para cerrar una HU)

Una HU pasa a *Terminado* solo si cumple todo:

1. Cumple todos sus **criterios de aceptación** y está vinculada a su módulo (SVC/TRX) y a sus HN.
2. Los diagramas, modelos de proceso o ADR afectados se actualizaron y **los aprobó el revisor asignado** en el PR.
3. Los artefactos modificados están **versionados** (ver `docs/VERSIONAMIENTO.md`): leyenda con versión, entrada en `CHANGELOG.md`, y fuente y exportación en el mismo PR.
4. Pasa la **checklist de revisión cruzada** (sección 6).
5. La **evidencia de revisión, retroalimentación y decisiones** quedó en el registro del sprint.
6. La tarjeta está en *Terminado* con **enlace al PR**.
7. No se exige evidencia de pipeline ni de despliegue (fuera del alcance de la materia).

## 6. Checklist del Pull Request (Definition of Done)

Copia esto en la descripción de cada PR:

```markdown
## Checklist
- [ ] El artefacto está en la carpeta correcta (`equipo/P?-...` o `docs/...`).
- [ ] Se citan los IDs correctos del contrato (`SVC`, `EVT`, `INT`, `ADR`).
- [ ] No se incluyen APIs, payloads, CI/CD ni pruebas.
- [ ] Se ha actualizado el `CHANGELOG.md` (si aplica).
- [ ] He asignado a mi revisor cruzado y este ha aprobado.
- [ ] El issue correspondiente ha sido referenciado (`Closes #nn`).
- [ ] Los nombres de SVC, TRX, EXT, EVT, INT, VL, estados y puertos son idénticos a `ARQUITECTURA_BASE.md`.
- [ ] Ningún diagrama muestra a un módulo leyendo o escribiendo datos de otro (PRI-02).
- [ ] Cada interacción indica S o A y coincide con su INT.
- [ ] Los eventos están en pasado y los emite solo su dueño.
- [ ] El dashboard y la proyección no aparecen como origen de decisiones ni como fuente de verdad.
- [ ] Fuente y exportación presentes y sincronizadas; leyenda completa (`ID · vX.Y.Z · AAAA-MM-DD · autor (Px) · HU-NN`).
- [ ] Sin secretos ni datos personales reales.
- [ ] La tarjeta del tablero está en *En revisión* con el enlace al PR.
```
