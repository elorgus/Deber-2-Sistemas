# Smart Inventory · Matriz de trazabilidad

| Campo | Valor |
|---|---|
| Documento | `docs/trazabilidad/matriz-trazabilidad.md` |
| Versión | **1.0.0** |
| Estado | Propuesto para revisión (Sprint 4 · HU-35) |
| Fecha | 2026-11-27 |
| Responsable | P4 (`joshdavr`) |
| Revisor | P6 (`CamGranda`) |
| Historia | HU-35 · issue #87 · rúbrica: Trazabilidad y presentación |
| Contrato de referencia | [`docs/ARQUITECTURA_BASE.md`](../ARQUITECTURA_BASE.md) v1.0.0 (§6.2 eventos, §7.2 interacciones, §11.2 RNF) |
| Línea base revisada | `main` en el commit `764652c` (release `diseno-v0.3.0`, cierre del Sprint 3) |

---

## 1. Cómo leer esta matriz

Cada historia de negocio (HN) se sigue por la cadena del criterio CA1:

**HN → SVC → EVT / INT → diagramas → ADR → RNF → versión del artefacto**

Reglas usadas para decidir cada vínculo, para que cualquiera pueda verificarlo:

| Eslabón | Cuándo se vincula a una HN |
|---|---|
| SVC | Capacidad dueña de la HN según `BACKLOG.md` §2.2 (épica) y `ARQUITECTURA_BASE.md` §3. |
| EVT | Eventos nombrados en el criterio de aceptación de la HN, o que publica o consume la capacidad para cumplirla (§6.2). |
| INT | Interacciones de §7.2 por las que viajan esos comandos, consultas o eventos. |
| Diagrama | El diagrama muestra esos EVT, INT o estados (se verificó en la fuente `.puml` o `.drawio` y en su exportación). C4N1 se vincula por el rol o sistema externo que muestra (Administrador, Analista, EXT-04), y los CLS-SOLID por la clase canónica (§10), por ejemplo `EstrategiaReposicion` o `ReposicionPorPronostico`. |
| ADR | ADR citado por el documento de flujo de la HN o por el RNF vinculado (columna "Patrón/ADR" de `requisitos-calidad.md`). |
| RNF | RNF de `requisitos-calidad.md` cuyo vínculo (SVC, INT o VL) coincide con el de la HN, o que la HN cita en su criterio. |
| Versión | Versión declarada en la leyenda del diagrama o en la cabecera del documento en la línea base. Los ADR se trazan por su estado, porque no llevan versión. |

---

## 2. Catálogo de artefactos y versiones

Todos los artefactos de diseño del repositorio en la línea base. La columna **ID corto** es la que se usa en las matrices.

### 2.1 Diagramas

| ID corto | Diagrama | Versión | HU | Fuente |
|---|---|---|---|---|
| C4N1 | C4N1-contexto | 0.1.1 | HU-11 | `docs/diagramas/fuente/C4N1-contexto.drawio` |
| C4N2 | C4N2-contenedores | 1.0.0 | HU-12 | `docs/diagramas/fuente/C4N2-contenedores.puml` |
| C4N3-02 | C4N3-SVC02-inventario | 0.1.0 | HU-14 | `docs/diagramas/fuente/C4N3-SVC02-inventario.drawio` |
| C4N3-03 | C4N3-SVC03-reposicion | 0.2.0 | HU-15 | `docs/diagramas/fuente/C4N3-SVC03-reposicion.drawio` |
| C4N3-05 | C4N3-SVC05-flota | 1.0.0 | HU-19 | `diagramas/c4/src/C4N3-SVC05-flota.puml` |
| ACT-FA · SEC-FA | ACT-FA-recepcion · SEC-FA-recepcion | 1.0.0 | HU-16 | `diagramas/actividad/` · `diagramas/secuencia/` |
| ACT-FB · SEC-FB | ACT-FB-reposicion · SEC-FB-reposicion | 1.0.0 | HU-17 | `diagramas/actividad/` · `diagramas/secuencia/` |
| ACT-FC · SEC-FC | ACT-FC-reserva-despacho · SEC-FC-reserva-despacho | 1.0.0 | HU-18 | `diagramas/actividad/` · `diagramas/secuencia/` |
| EST-lote | EST-lote | 1.0.0 | HU-25 | `diagramas/estados/src/` |
| EST-reserva | EST-reserva | 1.0.0 | HU-25 | `diagramas/estados/src/` |
| EST-sugerencia | EST-sugerencia | 1.0.0 | HU-25 | `diagramas/estados/src/` |
| EST-oc | EST-orden-compra | 1.0.0 | HU-25 | `diagramas/estados/src/` |
| EST-pedido | EST-pedido | 1.0.0 | HU-26 | `docs/diagramas/fuente/` |
| EST-mision | EST-mision-amr | 1.0.0 | HU-26 | `docs/diagramas/fuente/` |
| EST-amr | EST-amr | 1.0.0 | HU-26 | `docs/diagramas/fuente/` |
| EST-visita | EST-visita-vehiculo | 1.0.0 | HU-26 | `docs/diagramas/fuente/` |
| EST-muelle | EST-muelle | 1.0.0 | HU-26 | `docs/diagramas/fuente/` |
| CLS-inv | CLS-SOLID-inventario | 0.1.0 | HU-31 | `diagramas/clases/src/` |
| CLS-rep | CLS-SOLID-reposicion | 0.1.0 | HU-31 | `diagramas/clases/src/` |
| CLS-flota | CLS-SOLID-flota | 0.1.0 | HU-31 | `diagramas/clases/src/` |
| DSH-int | DSH-planta-interna | 1.1.0 | HU-32 | `diagramas/dashboard/src/` |
| DSH-ext | DSH-planta-externa | 1.1.0 | HU-32 | `diagramas/dashboard/src/` |

### 2.2 Documentos de diseño

| ID corto | Documento | Versión | HU |
|---|---|---|---|
| Flujo A | `docs/arquitectura/flujos/flujo-a-recepcion.md` | 1.0.1 | HU-16 |
| Flujo B | `docs/arquitectura/flujos/flujo-b-reposicion.md` | 1.1.0 | HU-17 |
| Flujo C | `docs/arquitectura/flujos/flujo-c-reserva-despacho.md` | 1.0.0 | HU-18 |
| Propiedad | `docs/arquitectura/propiedad-de-datos.md` | 0.1.0 | HU-13 |
| Matriz INT | `docs/arquitectura/matriz-interacciones.md` | 1.0.0 | HU-24 |
| Estados P4 | `equipo/P4-joshdavr/HU-25-diagramas-estado.md` | 1.0.0 | HU-25 |
| RNF | `docs/arquitectura/requisitos-calidad.md` | 1.0.0 | HU-27 |
| Seguridad | `docs/arquitectura/seguridad-roles.md` | 1.0.0 | HU-28 |
| Disponib. | `docs/arquitectura/disponibilidad-escalabilidad.md` | 1.0.0 | HU-29 |
| Patrones | `docs/diseno/patrones.md` | 0.1.1 | HU-30 |
| SOLID | `docs/diseno/solid.md` | 0.1.0 | HU-31 |
| Dashboard | `docs/dashboard/propuesta-dashboard.md` | 1.1.0 | HU-32 |

### 2.3 ADR

| ADR | Decisión | Estado en el archivo |
|---|---|---|
| ADR-001 | Monolito modular evolutivo | Propuesto |
| ADR-002 | Propiedad de datos por esquema | Propuesto |
| ADR-003 | Reserva FEFO en Inventario | Propuesto |
| ADR-004 | Síncrono y eventos con Outbox | Propuesto |
| ADR-005 | Proyección de lectura CQRS | Propuesto |
| ADR-006 | Detección de riesgo en Reposición | Propuesto |
| ADR-007 | Flota como unidad separada | Propuesto |
| ADR-009 | Réplicas locales de referencia | Propuesto |

ADR-008 (versionamiento de artefactos) es una decisión de gestión del repositorio y no de producto; ver §7.

---

## 3. Matriz principal · HN de prioridad M (CA1 y CA2)

Las 19 HN de prioridad M de `BACKLOG.md` §2.2.

| HN | SVC | EVT | INT | Diagramas (versión) | ADR | RNF | Documento (versión) |
|---|---|---|---|---|---|---|---|
| **HN-01** Registrar productos | SVC-01 | EVT-01, EVT-02 | INT-01, INT-04 | C4N1 0.1.1, C4N2 1.0.0, C4N3-02 0.1.0, C4N3-03 0.2.0, ACT-FA 1.0.0 (V3) | ADR-001, ADR-002, ADR-009 | RNF-DIS-04, RNF-MAN-02, RNF-SEG-03 | Flujo A 1.0.1, Propiedad 0.1.0, Seguridad 1.0.0 (OPS-06) |
| **HN-02** Bodegas, zonas y ubicaciones | SVC-01 | EVT-03 | INT-01, INT-04 | C4N2 1.0.0, C4N3-02 0.1.0, C4N3-05 1.0.0, ACT-FA 1.0.0 (V2, V4) | ADR-001, ADR-002, ADR-009 | RNF-DIS-04, RNF-MAN-02, RNF-SEG-03 | Flujo A 1.0.1, Propiedad 0.1.0, Seguridad 1.0.0 (OPS-06) |
| **HN-03** Registrar recepción | SVC-02 | EVT-04, EVT-05 · TX-01 | INT-01, INT-03, INT-06, INT-07, INT-08 | ACT-FA 1.0.0, SEC-FA 1.0.0, EST-lote 1.0.0, C4N3-02 0.1.0, C4N2 1.0.0, CLS-inv 0.1.0 | ADR-004, ADR-005, ADR-009 | RNF-REN-03, RNF-DIS-01, RNF-DIS-04, RNF-CON-02 | Flujo A 1.0.1, Disponib. 1.0.0 (FAL-01) |
| **HN-04** Consultar existencias | SVC-02 | EVT-05 | INT-15, INT-03 | C4N3-02 0.1.0, CLS-inv 0.1.0, DSH-int 1.1.0, DSH-ext 1.1.0, C4N2 1.0.0 | ADR-002, ADR-005 | RNF-REN-01, RNF-DIS-04 | Matriz INT 1.0.0, Propiedad 0.1.0 |
| **HN-07** Lotes próximos a vencer | SVC-02 | EVT-12, EVT-13 | INT-03 | EST-lote 1.0.0, C4N3-02 0.1.0, DSH-int 1.1.0, DSH-ext 1.1.0 | ADR-004, ADR-005 | RNF-CON-02 | Estados P4 1.0.0, Dashboard 1.1.0 |
| **HN-08** Política mínimo y reorden | SVC-03 | — (auditoría OPS-04) | INT-01 | C4N3-03 0.2.0, ACT-FB 1.0.0, CLS-rep 0.1.0 | ADR-006 | RNF-SEG-01, RNF-MAN-01 | Flujo B 1.1.0, Seguridad 1.0.0 (OPS-04) |
| **HN-09** Alerta bajo el mínimo | SVC-03, TRX-04 | EVT-05, EVT-14, EVT-15 | INT-10, INT-11, INT-03 | ACT-FB 1.0.0, SEC-FB 1.0.0, C4N3-03 0.2.0, CLS-rep 0.1.0, DSH-int 1.1.0, DSH-ext 1.1.0 | ADR-005, ADR-006 | RNF-CON-02, RNF-ESC-01 | Flujo B 1.1.0, Dashboard 1.1.0 |
| **HN-10** Sugerencia de reposición | SVC-03 | EVT-16, EVT-37 | INT-10, INT-11, INT-16 | EST-sugerencia 1.0.0, ACT-FB 1.0.0, SEC-FB 1.0.0, C4N3-03 0.2.0, CLS-rep 0.1.0 | ADR-006 | RNF-MAN-01, RNF-MAN-02 | Flujo B 1.1.0, SOLID 0.1.0 |
| **HN-11** Aprobar o rechazar sugerencia | SVC-03, EXT-01 | EVT-17, EVT-18, EVT-19 · TX-04 | INT-12, INT-13, INT-14 | EST-sugerencia 1.0.0, EST-oc 1.0.0, ACT-FB 1.0.0, SEC-FB 1.0.0, C4N3-03 0.2.0, DSH-int 1.1.0 | ADR-004 | RNF-SEG-01, RNF-DIS-04 | Flujo B 1.1.0, Estados P4 1.0.0, Seguridad 1.0.0 (OPS-03), Patrones 0.1.1, Disponib. 1.0.0 (FAL-05) |
| **HN-12** Recepción contra OC | SVC-02, SVC-03 | EVT-04, EVT-20 | INT-06, INT-09 | EST-oc 1.0.0, ACT-FA 1.0.0, SEC-FA 1.0.0, ACT-FB 1.0.0, SEC-FB 1.0.0, C4N3-03 0.2.0 | ADR-004, ADR-009 | RNF-DIS-01 | Flujo A 1.0.1, Estados P4 1.0.0 |
| **HN-13** Reserva FEFO | SVC-04, SVC-02 | EVT-21, EVT-08, EVT-22, EVT-05 · TX-02 | INT-17, INT-18, INT-19, INT-20 | EST-reserva 1.0.0, EST-lote 1.0.0, EST-pedido 1.0.0, ACT-FC 1.0.0, SEC-FC 1.0.0, C4N3-02 0.1.0, CLS-inv 0.1.0 | ADR-003, ADR-004 | RNF-REN-02, RNF-CON-01, RNF-DIS-02, RNF-ESC-02, RNF-MAN-01 | Flujo C 1.0.0, Patrones 0.1.1, SOLID 0.1.0 |
| **HN-14** Pedido en espera | SVC-04, SVC-02 | EVT-23, EVT-09 | INT-19, INT-20 | EST-pedido 1.0.0, EST-reserva 1.0.0, ACT-FC 1.0.0, SEC-FC 1.0.0, C4N3-02 0.1.0 | ADR-003, ADR-004 | RNF-CON-01, RNF-DIS-02 | Flujo C 1.0.0, Disponib. 1.0.0 (FAL-02) |
| **HN-15** Tarea de preparación | SVC-04, SVC-05 | EVT-24, EVT-25, EVT-26 | INT-21, INT-25, INT-26 | EST-pedido 1.0.0, EST-mision 1.0.0, ACT-FC 1.0.0, SEC-FC 1.0.0, C4N3-05 1.0.0 | ADR-004, ADR-007 | RNF-DIS-04, RNF-ESC-02 | Flujo C 1.0.0 |
| **HN-17** Asignar AMR y ruta | SVC-05, EXT-03 | EVT-29, EVT-30, EVT-31, EVT-32 | INT-21, INT-22, INT-23, INT-24 | EST-mision 1.0.0, EST-amr 1.0.0, C4N3-05 1.0.0, ACT-FC 1.0.0, SEC-FC 1.0.0, DSH-int 1.1.0 | ADR-007 | RNF-DIS-03, RNF-REN-04, RNF-ESC-01 | Flujo C 1.0.0, Disponib. 1.0.0 (FAL-03) |
| **HN-18** Llegada de camiones y muelle | SVC-05, EXT-05 | EVT-33, EVT-34, EVT-36 | INT-28, INT-33, INT-32 | EST-visita 1.0.0, EST-muelle 1.0.0, C4N3-05 1.0.0, ACT-FC 1.0.0, SEC-FC 1.0.0, DSH-ext 1.1.0 | ADR-007 | RNF-CON-02, RNF-ESC-01 | Flujo C 1.0.0, Disponib. 1.0.0 (FAL-09) |
| **HN-19** Despacho y descuento de stock | SVC-05, SVC-04, SVC-02 | EVT-35, EVT-27, EVT-11, EVT-05 · TX-03 | INT-27, INT-29, INT-30 | EST-visita 1.0.0, EST-pedido 1.0.0, EST-reserva 1.0.0, ACT-FC 1.0.0, SEC-FC 1.0.0, C4N3-02 0.1.0, CLS-flota 0.1.0 | ADR-003, ADR-004, ADR-007 | RNF-ESC-02, RNF-DIS-01 | Flujo C 1.0.0, Matriz INT 1.0.0 |
| **HN-20** KPIs y alertas con antigüedad | TRX-04, TRX-02 | EVT-04, EVT-05, EVT-14, EVT-15, EVT-21 a EVT-28 (vistas VL-01 a VL-03) | INT-03, INT-08, INT-32 | DSH-int 1.1.0, DSH-ext 1.1.0, C4N2 1.0.0, SEC-FA 1.0.0 | ADR-005 | RNF-CON-02, RNF-REN-03, RNF-ESC-01, RNF-ESC-02 | Dashboard 1.1.0, Patrones 0.1.1 (CQRS), Disponib. 1.0.0 (FAL-06) |
| **HN-21** Planta interna y externa | TRX-04, SVC-05 | EVT-31, EVT-32, EVT-33, EVT-34, EVT-36 (VL-04, VL-05) | INT-03, INT-32 | DSH-int 1.1.0, DSH-ext 1.1.0, C4N3-05 1.0.0, C4N2 1.0.0 | ADR-005, ADR-007 | RNF-REN-04, RNF-CON-02, RNF-ESC-01 | Dashboard 1.1.0 |
| **HN-24** Asignar roles | Todos los SVC, EXT-04 | — (autorización en la capacidad dueña) | INT-02, INT-12 | SEC-FB 1.0.0 (paso B6), C4N1 0.1.1, C4N2 1.0.0 | ADR-002 | RNF-SEG-01, RNF-SEG-03 | Seguridad 1.0.0 (OPS-07), Flujo B 1.1.0 |

---

## 4. HN de prioridad S y C

No las exige el CA2, pero se trazan para que no quede ninguna HN sin artefacto.

| HN | Prior. | SVC | EVT | INT | Diagramas (versión) | ADR | RNF | Documento (versión) |
|---|---|---|---|---|---|---|---|---|
| HN-05 Transferir existencias | S | SVC-02 | EVT-06 | INT-01 | C4N3-02 0.1.0, C4N2 1.0.0 | ADR-002 | — | Matriz INT 1.0.0 |
| HN-06 Ajustes con motivo | S | SVC-02 | EVT-07 | INT-01, INT-02 | C4N2 1.0.0 | ADR-002 | RNF-SEG-01, RNF-SEG-03 | Seguridad 1.0.0 (OPS-01) |
| HN-16 Cancelar pedido | S | SVC-04, SVC-02 | EVT-28, EVT-10 | INT-31 | EST-pedido 1.0.0, EST-reserva 1.0.0, ACT-FC 1.0.0, SEC-FC 1.0.0, C4N3-02 0.1.0 | ADR-003, ADR-004 | RNF-CON-03 | Flujo C 1.0.0, Seguridad 1.0.0 (OPS-05) |
| HN-22 Indicadores históricos | C | SVC-06, TRX-05 | EVT-38 | INT-05, INT-34 | C4N1 0.1.1 (posterior al MVP), C4N2 1.0.0 | ADR-001 | RNF-ESC-01 | — |
| HN-23 Pronóstico de demanda | C | SVC-06, SVC-03 | EVT-37 | INT-16 | SEC-FB 1.0.0, ACT-FB 1.0.0, C4N3-03 0.2.0, CLS-rep 0.1.0 | ADR-006 | RNF-MAN-01 | Flujo B 1.1.0, Patrones 0.1.1 |

---

## 5. Matriz inversa · artefacto → HN (CA3)

Comprueba que ningún artefacto de diseño queda sin HN.

### 5.1 Diagramas

| Artefacto | HN que traza |
|---|---|
| C4N1 0.1.1 | HN-01, HN-24, HN-22 |
| C4N2 1.0.0 | HN-01 a HN-04, HN-20, HN-21, HN-24 (y las S/C HN-05, HN-06, HN-22) |
| C4N3-02 0.1.0 | HN-01 a HN-04, HN-07, HN-13, HN-14, HN-19 |
| C4N3-03 0.2.0 | HN-01, HN-08 a HN-12 |
| C4N3-05 1.0.0 | HN-02, HN-15, HN-17, HN-18, HN-21 |
| ACT-FA · SEC-FA 1.0.0 | HN-01, HN-02, HN-03, HN-12, HN-20 |
| ACT-FB · SEC-FB 1.0.0 | HN-08 a HN-12, HN-24 |
| ACT-FC · SEC-FC 1.0.0 | HN-13 a HN-15, HN-17 a HN-19 |
| EST-lote 1.0.0 | HN-03, HN-07, HN-13 |
| EST-reserva 1.0.0 | HN-13, HN-14, HN-19 |
| EST-sugerencia 1.0.0 | HN-10, HN-11 |
| EST-oc 1.0.0 | HN-11, HN-12 |
| EST-pedido 1.0.0 | HN-13, HN-14, HN-15, HN-19 |
| EST-mision 1.0.0 | HN-15, HN-17 |
| EST-amr 1.0.0 | HN-17 |
| EST-visita 1.0.0 | HN-18, HN-19 |
| EST-muelle 1.0.0 | HN-18 |
| CLS-inv 0.1.0 | HN-03, HN-04, HN-13 |
| CLS-rep 0.1.0 | HN-08, HN-09, HN-10 |
| CLS-flota 0.1.0 | HN-19 (contratación del transportista, INT-27) |
| DSH-int 1.1.0 | HN-04, HN-07, HN-09, HN-11, HN-17, HN-20, HN-21 |
| DSH-ext 1.1.0 | HN-04, HN-07, HN-09, HN-18, HN-20, HN-21 |

### 5.2 Documentos, ADR y RNF

| Artefacto | HN que traza |
|---|---|
| Flujo A 1.0.1 | HN-01, HN-02, HN-03, HN-12 |
| Flujo B 1.1.0 | HN-08 a HN-11, HN-24 |
| Flujo C 1.0.0 | HN-13 a HN-15, HN-17 a HN-19 |
| Propiedad 0.1.0 | HN-01, HN-02, HN-04 (dueño de cada dato y réplicas) |
| Matriz INT 1.0.0 | HN-04, HN-19 (y todas las HN por sus INT) |
| Estados P4 1.0.0 | HN-07, HN-11, HN-12 |
| Seguridad 1.0.0 | HN-01, HN-02, HN-08, HN-11, HN-24 |
| Disponib. 1.0.0 | HN-03, HN-11, HN-14, HN-17, HN-18, HN-20 |
| Patrones 0.1.1 | HN-11, HN-13, HN-20 |
| SOLID 0.1.0 | HN-10, HN-13 |
| Dashboard 1.1.0 | HN-07, HN-09, HN-20, HN-21 |
| ADR-001 | HN-01 a HN-04 |
| ADR-002 | HN-01, HN-02, HN-04, HN-24 |
| ADR-003 | HN-13, HN-14, HN-19 |
| ADR-004 | HN-03, HN-07, HN-11 a HN-15, HN-19 |
| ADR-005 | HN-03, HN-04, HN-07, HN-09, HN-20, HN-21 |
| ADR-006 | HN-08, HN-09, HN-10 |
| ADR-007 | HN-15, HN-17, HN-18, HN-19, HN-21 |
| ADR-009 | HN-01, HN-02, HN-03, HN-12 |
| RNF-REN-01 | HN-04 |
| RNF-REN-02 | HN-13 |
| RNF-REN-03 | HN-03, HN-20 |
| RNF-REN-04 | HN-17, HN-21 |
| RNF-DIS-01 | HN-03, HN-12, HN-19 |
| RNF-DIS-02 | HN-13, HN-14 |
| RNF-DIS-03 | HN-17 |
| RNF-DIS-04 | HN-01 a HN-04, HN-11, HN-15 |
| RNF-ESC-01 | HN-09, HN-17, HN-18, HN-20, HN-21 |
| RNF-ESC-02 | HN-13, HN-15, HN-19, HN-20 |
| RNF-CON-01 | HN-13, HN-14 |
| RNF-CON-02 | HN-03, HN-07, HN-09, HN-18, HN-20, HN-21 |
| RNF-CON-03 | HN-16 (prioridad S) |
| RNF-SEG-01 | HN-08, HN-11, HN-24 |
| RNF-SEG-02 | Transversal de proceso (revisión de PR, sin secretos); ver §7 |
| RNF-SEG-03 | HN-01, HN-02, HN-24 |
| RNF-MAN-01 | HN-08, HN-10, HN-13 |
| RNF-MAN-02 | HN-01, HN-02, HN-10 |

---

## 6. Verificación de los criterios

| Criterio | Resultado | Evidencia |
|---|---|---|
| **CA1** · Cadena completa HN → SVC → EVT/INT → diagramas → ADR → RNF → versión | Cumple | La matriz de §3 tiene las ocho columnas en cada fila. HN-08 y HN-24 no publican eventos de negocio: su columna EVT explica por qué (auditoría y autorización). |
| **CA2** · 100 % de las HN de prioridad M trazadas | Cumple: **19 de 19** | HN-01 a HN-04, HN-07 a HN-15, HN-17 a HN-21 y HN-24. Cada una tiene al menos un diagrama, un ADR y un RNF. |
| **CA3** · Ningún huérfano | Cumple | Toda HN (M, S y C) tiene artefacto (§3 y §4). Todo diagrama, documento de diseño, ADR de producto y RNF tiene al menos una HN (§5). Los artefactos de gestión y RNF-SEG-02 se tratan aparte, con su justificación (§7). |

---

## 7. Artefactos de gestión (fuera de la cadena HN)

Estos artefactos no describen el producto, sino el proceso del equipo. Por eso no se trazan a una HN y no cuentan como huérfanos:

| Artefacto | Por qué no se traza a una HN |
|---|---|
| `docs/ARQUITECTURA_BASE.md` · `docs/BACKLOG.md` | Son la fuente de la matriz: el contrato y el backlog que definen las HN. |
| `CONTRIBUTING.md` · `docs/VERSIONAMIENTO.md` · `docs/versionamiento/politica-versionamiento.md` · ADR-008 | Reglas de trabajo y versionamiento del repositorio. |
| `docs/gestion/sprints/sprint-3.md` · actas y plan en `equipo/P3-RichJ-100/` | Ceremonias Scrum. |
| `equipo/P5-JessPCL/HU-08-riesgos.md` · `equipo/P7-luisdavidborjap-web/HU-06-revision-arquitectura.md` | Gestión de riesgos y revisión del contrato. |
| `equipo/P2-EstebanEsp07/HU-02` a `HU-04` | Visión, épicas e historias: son el origen de las HN. |
| RNF-SEG-02 | Requisito de calidad del repositorio (cero secretos en el PR), no de una historia de negocio. |

---

## 8. Hallazgos para la auditoría de versionamiento (HU-36)

Inconsistencias encontradas al armar la matriz. No se corrigen aquí porque cada artefacto tiene su dueño; se reportan para que P3 las resuelva en HU-36.

| ID | Hallazgo | Dónde | Responsable sugerido |
|---|---|---|---|
| T-01 | La versión del CHANGELOG no coincide con la de la cabecera: Dashboard (CHANGELOG 2.0.0, archivo 1.1.0), Patrones (1.0.0 frente a 0.1.1), SOLID (1.0.0 frente a 0.1.0). Esta matriz usa la versión del archivo. | `CHANGELOG.md` §diseno-v0.3.0 | P3 con P2 y P7 |
| T-02 | El CHANGELOG declara ADR-005 y ADR-009 como aceptados, pero los archivos siguen en estado *Propuesto*. Ningún ADR de producto figura como *Aceptado* en su archivo. | `docs/adr/` | P1 y P3 |
| T-03 | El documento de dashboard menciona el diagrama `DSH-frescura-estados` v1.1.0, pero no existe en el repositorio. | `docs/dashboard/propuesta-dashboard.md` | P2 |
| T-04 | Los diagramas siguen dos estructuras de carpetas: `docs/diagramas/fuente|export` (C4 y EST de HU-26) y `diagramas/<tipo>/src|export` (las demás). | `docs/diagramas/`, `diagramas/` | P3 |
| T-05 | §7.2 no tiene una INT propia para los comandos de configuración: catálogo y ubicaciones (HN-01, HN-02, OPS-06), política de reposición (HN-08, OPS-04), transferencia (HN-05) y ajuste (HN-06). Solo viajan por INT-01. | `ARQUITECTURA_BASE.md` §7.2 | P1 (§0.3) |
| T-06 | Siguen en borrador (0.x) C4N1, C4N3-SVC02, C4N3-SVC03, Propiedad de datos, los CLS-SOLID, Patrones y SOLID. Deben subir a 1.0.0 antes del release final. | §2 de este documento | Dueño de cada artefacto |

---

## Historial de cambios

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-27 | P4 | Matriz HN → SVC → EVT/INT → diagramas → ADR → RNF → versión, matriz inversa, verificación de CA1 a CA3 y hallazgos T-01 a T-06 | HU-35 |
