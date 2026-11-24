# Smart Inventory · Política de versionamiento de artefactos de diseño

| Campo | Valor |
|---|---|
| Documento | `docs/VERSIONAMIENTO.md` |
| Versión | **1.0.0** |
| Estado | Propuesto para aprobación del equipo |
| Dueño · Revisor | P3 (Scrum Master y gestor de versiones) · P2 |
| Fecha | 2026-11-23 |
| Basado en | `docs/ARQUITECTURA_BASE.md` §0.3 y `docs/BACKLOG.md` §7 |
| Decisión asociada | ADR-008 Política de versionamiento de artefactos de diseño (HU-21) |

> Esta política versiona **artefactos de diseño**: contrato, documentos, diagramas y ADR. El diseño de APIs, esquemas de mensajes, el flujo de ramas, CI/CD y las releases del **software** no se tratan aquí (`ARQUITECTURA_BASE.md` §14).

---

## 1. Qué se versiona y cómo se identifica la versión

| Artefacto | Ejemplo | Dónde vive la versión | Dónde queda el historial |
|---|---|---|---|
| Contrato maestro | `docs/ARQUITECTURA_BASE.md` | Campo *Versión* del encabezado y tag `contrato-vX.Y.Z` | Tabla "Historial de cambios" al final del archivo y `CHANGELOG.md` |
| Documento | `docs/BACKLOG.md`, `docs/dashboard/propuesta-dashboard.md` | Campo *Versión* del encabezado | Tabla "Historial de cambios" al final y `CHANGELOG.md` |
| Diagrama | `C4N3-SVC02-inventario` | Leyenda dentro del diagrama (§5) | Historial de git de la fuente y `CHANGELOG.md` |
| ADR | `ADR-003` | **No lleva versión**: lleva estado (§6) | Índice `docs/adr/README.md` |
| Paquete de diseño | Todo el repositorio al cierre de un sprint | Tag `diseno-vX.Y.Z` | `CHANGELOG.md` |

Versiones vigentes al redactar esta política: `ARQUITECTURA_BASE.md` 1.0.0 (`contrato-v1.0.0`), `BACKLOG.md` 1.0.0.

---

## 2. Versionamiento semántico (MAJOR.MINOR.PATCH)

Aplica al contrato, a cada documento y a cada diagrama.

| Tipo de cambio | Sube | Ejemplo |
|---|---|---|
| Cambia un límite o el dueño de un dato; se elimina o renombra un SVC, EVT, INT o VL | **MAJOR** | Mover la reserva de SVC-02 a SVC-04 |
| Se agrega algo compatible: un EVT, INT, VL, RNF o diagrama nuevo | **MINOR** | Agregar un evento nuevo aprobado por ADR |
| Corrección de forma sin cambiar el significado | **PATCH** | Corregir una etiqueta en un diagrama |

Reglas de aplicación:

1. Al subir MAJOR se reinician MINOR y PATCH (`1.4.2` → `2.0.0`); al subir MINOR se reinicia PATCH (`1.4.2` → `1.5.0`).
2. Si un cambio podría clasificarse en dos niveles, se aplica el mayor y el revisor del PR confirma la clasificación.
3. Un cambio de MAJOR o MINOR en el contrato exige un ADR en estado *Propuesto* (§4).
4. Los artefactos nuevos nacen en `1.0.0` cuando su revisor los aprueba por primera vez.

---

## 3. Flujo de trabajo: ramas, commits, PR y tags

| Elemento | Convención | Ejemplo |
|---|---|---|
| Rama principal | `main`: solo contenido aprobado | n/a |
| Rama de trabajo | Una por HU desde `main`: `feat/HU-nn-descripcion-corta` (`CONTRIBUTING.md` §1) | `feat/HU-14-c4n3-inventario` |
| Commit | Mensaje claro en español: `docs(HU-nn): qué cambió` | `docs(HU-14): agrega el puerto GestionReservas al C4 N3` |
| Pull request | Título `HU-nn: Descripción breve`; revisor = revisor de la HU; checklist de `CONTRIBUTING.md` en la descripción | `HU-14: C4 Nivel 3 de Inventario y ADR-003` |
| PR que toca el contrato | Además aprueban P1 y el diseñador del módulo afectado (`BACKLOG.md` §1.3) | n/a |
| Tag del contrato | `contrato-vX.Y.Z` en cada versión aprobada de `ARQUITECTURA_BASE.md` | `contrato-v1.0.0` |
| Tag del paquete | `diseno-vX.Y.Z` al cierre de cada sprint | `diseno-v0.2.0` |

Nada entra a `docs/` ni a `diagramas/` sin un PR aprobado por el revisor asignado.

> Las convenciones de ramas, commits y títulos de PR son las de `CONTRIBUTING.md`. `docs/BACKLOG.md` §7.1 usa otro formato (`hu-NN-...`, `HU-NN: ...`, `HU-NN · ...`); el equipo debe elegir uno y dejar una sola fuente.

### Tags del paquete de diseño

| Sprint | Tag | Contenido |
|---|---|---|
| S1 | `diseno-v0.1.0` | Product backlog, tablero, riesgos y contrato v1.0.0 aprobado |
| S2 | `diseno-v0.2.0` | C4 N1 a N3, propiedad de datos, flujos A, B y C, ADR-001 a ADR-004 y ADR-006 a ADR-008, dashboard v1 |
| S3 | `diseno-v0.3.0` | Matriz de interacciones, diagramas de estado, RNF, seguridad, fallos y escalabilidad, patrones, SOLID, ADR-005 y ADR-009, dashboard v2 |
| S4 | `diseno-v1.0.0` | Paquete coherente, trazable y presentable |

El tag se crea sobre `main` después de fusionar el último PR del sprint, durante la review.

---

## 4. Cambios al contrato (`ARQUITECTURA_BASE.md` §0.3)

1. Quien detecta la necesidad abre un PR que modifica `ARQUITECTURA_BASE.md`. Si cambia un límite, el dueño de un dato, o agrega, renombra o elimina un SVC, EVT o INT, crea además un ADR en estado *Propuesto*.
2. Revisan **P1** y el **diseñador responsable del módulo afectado** (`BACKLOG.md` §1.3).
3. Al aprobarse: se sube la versión según la sección 2 de esta política, se actualiza el historial del contrato y `CHANGELOG.md`, se crea el tag `contrato-vX.Y.Z` y se avisa al equipo.
4. Mientras no se apruebe, el elemento nuevo se marca **«PROPUESTA»** en cualquier borrador y no se usa como oficial.

---

## 5. Diagramas

**Nombre:** `TIPO-ALCANCE-descripcion`, idéntico en la fuente (`src/`) y en la exportación (`export/`).

| Tipo | Prefijo | Ejemplos |
|---|---|---|
| C4 | `C4N1`, `C4N2`, `C4N3` | `C4N1-contexto`, `C4N2-contenedores`, `C4N3-SVC02-inventario`, `C4N3-SVC03-reposicion`, `C4N3-SVC05-flota` |
| Actividad | `ACT` | `ACT-FA-recepcion`, `ACT-FB-reposicion`, `ACT-FC-reserva-despacho` |
| Secuencia | `SEC` | `SEC-FA-recepcion`, `SEC-FB-reposicion`, `SEC-FC-reserva-despacho`, `SEC-VERTICAL-recorrido-integral` |
| Estados | `EST` | `EST-pedido`, `EST-reserva`, `EST-lote`, `EST-sugerencia`, `EST-orden-compra`, `EST-mision-amr`, `EST-amr`, `EST-visita-vehiculo`, `EST-muelle` |
| Clases | `CLS` | `CLS-dominio-conceptual`, `CLS-SOLID-inventario`, `CLS-SOLID-reposicion`, `CLS-SOLID-flota` |
| Dashboard | `DSH` | `DSH-planta-interna`, `DSH-planta-externa` |

**Leyenda obligatoria** en cada diagrama:

```text
ID · vX.Y.Z · AAAA-MM-DD · autor (Px) · HU-NN
```

Ejemplo: `C4N3-SVC02-inventario · v1.0.0 · 2026-12-04 · P4 · HU-14`

Reglas:

- Todo diagrama cita los IDs del contrato que aparecen en él (`SVC`, `EVT`, `INT`, `VL`).
- Fuente y exportación van **en el mismo PR**, con el mismo nombre. Si no coinciden, el PR no se aprueba.
- Herramienta recomendada: PlantUML (con C4-PlantUML para los C4), porque es texto y el historial muestra exactamente qué cambió. Para los bocetos del dashboard se acepta draw.io (el `.drawio` también se versiona). Exportar en SVG (preferido) o PNG.
- Todo cambio de un diagrama actualiza la versión de la leyenda y agrega una entrada en `CHANGELOG.md`.

---

## 6. ADR (registros de decisiones de arquitectura)

Los ADR **no llevan número de versión**. Llevan estado:

```text
Propuesto → Aceptado → Reemplazado por ADR-nnn
                    ↘ Deprecado
```

- Un ADR *Aceptado* **no se edita**. Si la decisión cambia, se crea un ADR nuevo que lo reemplaza y el anterior pasa a *Reemplazado por ADR-nnn*. Solo se permite corregir ortografía o enlaces sin cambiar el significado, y se anota en el historial git.
- Ubicación: `docs/adr/ADR-nnn-titulo-corto.md`. El índice `docs/adr/README.md` muestra el estado de cada ADR y es mantenido por P3.
- Contenido mínimo: contexto, al menos 2 opciones consideradas, decisión, consecuencias positivas y negativas, fecha, estado y HU y diagramas relacionados.
- Cuándo se exige un ADR: cuando cambia un límite, el dueño de un dato, o se agrega, renombra o elimina un SVC, EVT o INT (sección 4).
- Un ADR pasa a *Aceptado* cuando lo aprueba su revisor en el PR.

### Registro inicial (`ARQUITECTURA_BASE.md` §12)

| ADR | Título | Resp. | HU | Estado inicial |
|---|---|---|---|---|
| ADR-001 | Monolito modular evolutivo con unidades de despliegue | P1 | HU-20 | Propuesto |
| ADR-002 | Propiedad de datos por módulo con aislamiento por esquema | P1 | HU-20 | Propuesto |
| ADR-003 | La reserva de stock y la selección FEFO pertenecen a Inventario | P4 | HU-14 | Propuesto |
| ADR-004 | Comandos síncronos y eventos de negocio con Outbox e idempotencia | P1 | HU-20 | Propuesto |
| ADR-005 | Proyección de lectura (CQRS) con consistencia eventual informada | P7 | HU-30 | Propuesto |
| ADR-006 | La detección de riesgo de agotamiento pertenece a Reposición | P5 | HU-17 | Propuesto |
| ADR-007 | Flota y Operación como unidad de despliegue separada desde el MVP | P6 | HU-19 | Propuesto |
| ADR-008 | Política de versionamiento de artefactos de diseño | P3 | HU-21 | Propuesto |
| ADR-009 | Réplicas locales de datos de referencia en lugar de consultas síncronas | P1 | HU-29 | Propuesto |

---

## 7. CHANGELOG

`CHANGELOG.md` tiene **una sección por versión del paquete**, de la más reciente a la más antigua. Cada sección incluye fecha, HU incluidas, artefactos cambiados con su versión y ADR aceptados.

```markdown
## diseno-v0.1.0 · AAAA-MM-DD · Sprint 1

**HU incluidas:** HU-01 a HU-10

**Artefactos**
| Artefacto | Versión | Cambio | HU |
|---|---|---|---|
| docs/ARQUITECTURA_BASE.md | 1.0.0 | Versión inicial del contrato (tag contrato-v1.0.0) | HU-06 |
| docs/BACKLOG.md | 1.0.0 | Versión inicial | HU-04, HU-05 |

**ADR aceptados:** ninguno
```

Los cambios de un artefacto individual se registran también en la tabla "Historial de cambios" de su propio archivo.

---

## 8. Historial y trazabilidad

### 8.1 Historial

Para cualquier artefacto debe poder responderse **qué cambió, por qué, quién lo cambió, cuándo y qué lo revisó**:

| Pregunta | Dónde está la respuesta |
|---|---|
| Qué cambió | Diferencias del PR (la fuente PlantUML o Markdown) y entrada en `CHANGELOG.md` |
| Por qué | HU enlazada en el PR; ADR si cambió un límite, un dato o un SVC, EVT o INT |
| Quién | Autor del commit y autor (Px) en la leyenda o el historial |
| Cuándo | Fecha de la leyenda y del historial; fecha del tag |
| Quién lo aprobó | Aprobación del revisor asignado en el PR (y de P1 y del diseñador del módulo si toca el contrato) |

### 8.2 Cadena de trazabilidad

La matriz de trazabilidad (HU-35) sigue esta cadena para cada historia de negocio:

```text
HN → SVC → EVT / INT → diagramas → ADR → RNF → versión del artefacto
```

Ejemplo ilustrativo con HN-09 (alerta cuando un SKU cae bajo su mínimo):

| Eslabón | Valor |
|---|---|
| HN | HN-09 |
| SVC | SVC-03 Reposición y Compras (detecta), SVC-02 Inventario (publica el saldo) |
| EVT / INT | EVT-05 SaldoActualizado → EVT-14 RiesgoAgotamientoDetectado; EVT-15 RiesgoAgotamientoResuelto |
| Diagramas | `ACT-FB-reposicion`, `SEC-FB-reposicion` |
| ADR | ADR-006 La detección de riesgo de agotamiento pertenece a Reposición |
| RNF | RNF-REN-03 (frescura en VL-01) |
| Versión | `[versión de cada diagrama y ADR al momento de la matriz]` |

Reglas de trazabilidad:

- Ningún artefacto queda huérfano: todo artefacto enlaza a una HU y a una HN o módulo (SVC/TRX).
- Todo PR enlaza su HU y el issue (`Closes #nn`).
- La auditoría final de versionamiento la hace P3 en HU-36.

---

## 9. Ejemplo de cambio versionado

> Ejemplo ilustrativo. HU-21 lo reemplazará por un cambio real de un diagrama del Sprint 2.

| Paso | Qué ocurre |
|---|---|
| 1 | P4 publica `C4N3-SVC02-inventario` **v1.0.0** (HU-14). Entrada en `CHANGELOG.md` de `diseno-v0.2.0`. |
| 2 | P6 detecta una etiqueta mal escrita. Corrige y publica **v1.0.1** (**PATCH**: forma, mismo significado). Revisa P6. |
| 3 | Se aprueba un nuevo EVT por ADR. P4 lo agrega al diagrama y publica **v1.1.0** (**MINOR**: algo compatible). Fuente y exportación en el mismo PR. |
| 4 | Si la reserva se moviera a SVC-04, sería **v2.0.0** (**MAJOR**): cambia un límite, exige ADR *Propuesto* y aprobación de P1 y del diseñador del módulo afectado. |

---

## 10. Checklist de versionamiento (para cada PR)

- [ ] La versión del artefacto subió según la sección 2.
- [ ] La leyenda del diagrama (o el encabezado del documento) muestra la nueva versión, fecha, autor y HU.
- [ ] Fuente y exportación están en el mismo PR con el mismo nombre.
- [ ] Hay entrada en `CHANGELOG.md` y en el historial del archivo.
- [ ] Si cambió un límite, un dueño de datos o un SVC, EVT o INT: existe el ADR *Propuesto* y el contrato sigue la sección 4.
- [ ] El PR enlaza su HU y su issue.

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-23 | P3 | Versión inicial de la política (borrador anticipado de HU-21) | HU-21 |
