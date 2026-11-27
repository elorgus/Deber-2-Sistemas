# Smart Inventory · Épicas EP-01 a EP-08 (validación y prioridad MoSCoW)

| Campo | Valor |
|---|---|
| Documento | `equipo/P2-EstebanEsp07/HU-03-epicas.md` (insumo de `docs/BACKLOG.md` §2.1) |
| Versión | **1.0.0** |
| Estado | Propuesto para revisión (PR de HU-03) |
| Autor | P2 · Product Owner (`EstebanEsp07`) |
| Revisor | P3 · Scrum Master y gestor de versiones (`docs/BACKLOG.md` §1.2) |
| Fecha | 2026-11-23 |
| HU | **HU-03** según la asignación del equipo a P2. Criterios de referencia: `docs/BACKLOG.md` §4.1, HU-04 CA1, CA2 y CA4 (épicas) |
| Épicas | EP-01 a EP-08 |
| Contrato de referencia | `docs/ARQUITECTURA_BASE.md` v1.0.0 y `docs/BACKLOG.md` v1.0.0 |

> Los IDs (`EP`, `HN`, `SVC`, `TRX`, `EXT`, `EVT`, `INT`, `VL`, `ROL`, `OPS`, `TX`, `CE`) son los del contrato y del BACKLOG. No se crearon identificadores nuevos.

**Regla de prioridad de una épica:** hereda la prioridad más alta de sus HN (M > S > C). Una épica es **Must** si al menos una de sus HN entra al MVP.

---

## 1. Resumen de épicas

| ID | Épica | Módulos | Objetivo | MoSCoW | HN (M / S / C) | Diseñador de referencia |
|---|---|---|---|---|---|---|
| EP-01 | Catálogo y ubicaciones | SVC-01 | Tener un maestro confiable de productos y de la estructura física | **M** | 2 (2 / 0 / 0) | P4 |
| EP-02 | Inventario y lotes | SVC-02 | Saber qué hay, dónde, de qué lote y cuándo vence | **M** | 5 (3 / 2 / 0) | P4 |
| EP-03 | Reposición y compras | SVC-03 | Reponer antes del quiebre con decisiones trazables | **M** | 5 (5 / 0 / 0) | P5 |
| EP-04 | Pedidos y fulfillment | SVC-04 | Reservar con FEFO, preparar y despachar sin perder pedidos | **M** | 4 (3 / 1 / 0) | P6 |
| EP-05 | Flota y operación | SVC-05 | Coordinar AMR, camiones, muelles y patio | **M** | 3 (3 / 0 / 0) | P6 |
| EP-06 | Dashboard operativo | TRX-01, TRX-02, TRX-04 | Supervisar en vivo la planta interna y la externa | **M** | 2 (2 / 0 / 0) | P2 (TRX-01), P1 (TRX-02), P7 (TRX-04) |
| EP-07 | Analítica | SVC-06, TRX-05 | Aprender del histórico y anticipar la demanda | **C** | 2 (0 / 0 / 2) | P7 |
| EP-08 | Seguridad y acceso | EXT-04, todos los SVC | Que cada usuario haga solo lo que le corresponde | **M** | 1 (1 / 0 / 0) | P2 (HU-28), P1 |
| | **Total** | | | | **24 (19 / 3 / 2)** | |

Los diseñadores de referencia salen de `docs/BACKLOG.md` §1.3.

---

## 2. Validación de las épicas

| # | Verificación | Resultado |
|---|---|---|
| 1 | Existen 8 épicas con IDs consecutivos EP-01 a EP-08 | ✔ |
| 2 | Cada una de las 24 HN pertenece a exactamente una épica (2 + 5 + 5 + 4 + 3 + 2 + 2 + 1 = 24) | ✔ |
| 3 | Los módulos de cada épica existen con el mismo nombre en `ARQUITECTURA_BASE` §3 y §4 | ✔ |
| 4 | Cada capacidad SVC-01 a SVC-06 es cubierta por al menos una épica | ✔ |
| 5 | La prioridad de cada épica es coherente con sus HN (regla de arriba) | ✔ |
| 6 | El orden respeta dependencias: catálogo y roles (órdenes 1 a 3) antes que inventario (desde el 4); inventario antes que reposición (desde el 6) | ✔ |
| 7 | El MVP (19 HN M) recorre los flujos A, B y C en su camino principal | ✔ |
| 8 | Todo interesado de `HU-02-vision.md` §2.1 está vinculado a al menos una épica | ✔ |

### Observaciones para el refinamiento

Son observaciones del PO; no cambian el contrato ni los IDs. Cualquier ajuste se tramita por PR (y ADR si aplica) según `ARQUITECTURA_BASE` §0.3.

1. **TRX-03 no pertenece a ninguna épica.** El broker es infraestructura habilitadora de todas; se cubre en HU-12 y ADR-004.
2. **Ingesta al Lakehouse (INT-05) y EP-07.** La analítica del MVP se limita a esa ingesta, pero EP-07 es Could porque ninguna de sus HN entra al MVP. Se propone tratar la ingesta como habilitador técnico del MVP, no como HN.
3. **Capacidades del contrato sin HN propia** (candidatas a refinamiento, **PROPUESTA** y no oficiales): cambio de estado de lote a cuarentena o liberación (OPS-02), reasignar o abortar misión AMR (OPS-08, INT-33), gestión de proveedores, cancelación de OC, y registro o importación de pedidos (INT-17, INT-18).
4. **Cronómetro de HN-09.** Pide la alerta en VL-02 en ≤ 5 s, mientras CE-03 (detección ≤ 5 s desde EVT-05) y CE-01 (vista ≤ 5 s p95) podrían sumar hasta ~10 s. Aclarar el punto de inicio de la medición en HU-27.
5. **Dashboard antes de pedidos y flota.** EP-06 (órdenes 11 y 12) precede a EP-04 y EP-05 (13 a 18). Es aceptable: la proyección solo consume eventos (PRI-05) y sus vistas se completan a medida que los dueños los publican.

---

## 3. Detalle por épica

### EP-01 · Catálogo y ubicaciones · **Must**

| Aspecto | Detalle |
|---|---|
| Objetivo | Tener un maestro confiable de productos y de la estructura física |
| Módulo | SVC-01 (UD-1) |
| HN | HN-01 (M, orden 1), HN-02 (M, orden 2) |
| Rol | ROL-07 Administrador (OPS-06) |
| Eventos e interacciones | EVT-01, EVT-02, EVT-03 · INT-04 |
| Diseñada en | HU-07, HU-13 |
| Justificación | Sin SKU y ubicaciones válidos no hay recepción; es la primera dependencia de todo el flujo |

### EP-02 · Inventario y lotes · **Must**

| Aspecto | Detalle |
|---|---|
| Objetivo | Saber qué hay, dónde, de qué lote y cuándo vence |
| Módulo | SVC-02 (UD-1) |
| HN | HN-03 (M, 4), HN-04 (M, 5), HN-07 (M, 19), HN-05 (S, 21), HN-06 (S, 22) |
| Roles | ROL-01 Operador de bodega, ROL-02 Supervisor de bodega (OPS-01, OPS-02) |
| Eventos e interacciones | EVT-04 a EVT-13 · INT-06 a INT-08, INT-15 · TX-01, TX-03 |
| Diseñada en | HU-14, HU-16, HU-24, HU-25, HU-28 |
| Justificación | Es la fuente de verdad de existencias y arranca el flujo vertical; transferencias y ajustes (S) no son necesarios para recorrerlo |

### EP-03 · Reposición y compras · **Must**

| Aspecto | Detalle |
|---|---|
| Objetivo | Reponer antes del quiebre con decisiones trazables |
| Módulo | SVC-03 (UD-1) |
| HN | HN-08 (M, 6), HN-09 (M, 7), HN-10 (M, 8), HN-11 (M, 9), HN-12 (M, 10) |
| Roles | ROL-03 Comprador (OPS-03, OPS-04), ROL-01 en la recepción contra OC |
| Eventos e interacciones | EVT-14 a EVT-20 · INT-09 a INT-14 · TX-04 · CE-03, CE-06 |
| Diseñada en | HU-15, HU-16, HU-17, HU-22, HU-25 |
| Justificación | Es el flujo B completo y el centro del flujo vertical exigido en la presentación final |

### EP-04 · Pedidos y fulfillment · **Must**

| Aspecto | Detalle |
|---|---|
| Objetivo | Reservar con FEFO, preparar y despachar sin perder pedidos |
| Módulo | SVC-04 (UD-1), con la reserva en SVC-02 (ADR-003) |
| HN | HN-13 (M, 13), HN-14 (M, 14), HN-15 (M, 15), HN-16 (S, 20) |
| Roles | ROL-05 Supervisor de operaciones (OPS-05 al cancelar), ROL-01 Operador de bodega (HN-15) |
| Eventos e interacciones | EVT-21 a EVT-28 · INT-17 a INT-20, INT-25, INT-30, INT-31 · TX-02, TX-05 · CE-04, CE-05 |
| Diseñada en | HU-14, HU-18, HU-26 |
| Justificación | Es el flujo C; la cancelación (S) se difiere porque el camino principal no la requiere |

### EP-05 · Flota y operación · **Must**

| Aspecto | Detalle |
|---|---|
| Objetivo | Coordinar AMR, camiones, muelles y patio |
| Módulo | SVC-05 (UD-2) |
| HN | HN-17 (M, 16), HN-18 (M, 17), HN-19 (M, 18) |
| Roles | ROL-04 Operador de patio y flota (OPS-08); ROL-05 Supervisor de operaciones (HN-19) |
| Eventos e interacciones | EVT-29 a EVT-36 · INT-21 a INT-24, INT-26 a INT-29, INT-33 · TX-06 |
| Diseñada en | HU-18, HU-19, HU-24, HU-26 |
| Justificación | Cierra el flujo C (preparación y despacho) y aporta la llegada de camiones del paso 8 del flujo vertical |

### EP-06 · Dashboard operativo · **Must**

| Aspecto | Detalle |
|---|---|
| Objetivo | Supervisar en vivo la planta interna y la externa |
| Módulos | TRX-01, TRX-02, TRX-04 (UD-4 y UD-5) |
| HN | HN-20 (M, 11), HN-21 (M, 12) |
| Rol | ROL-05 Supervisor de operaciones |
| Vistas, eventos e interacciones | VL-01 a VL-06 · INT-01, INT-03, INT-08, INT-32 · CE-01 |
| Diseñada en | HU-22, HU-32 |
| Justificación | La frescura del dato y los modos interno y externo son parte de la presentación final; es solo lectura (PRI-07) |

### EP-07 · Analítica · **Could**

| Aspecto | Detalle |
|---|---|
| Objetivo | Aprender del histórico y anticipar la demanda |
| Módulos | SVC-06, TRX-05 (UD-3) |
| HN | HN-23 (C, 23), HN-22 (C, 24) |
| Roles | ROL-06 Analista, ROL-03 Comprador (pronóstico) |
| Vistas, eventos e interacciones | VL-07 · EVT-37, EVT-38 · INT-05, INT-16, INT-34 · CE-07 |
| Diseñada en | HU-12, HU-15, HU-30, HU-31 |
| Justificación | Ninguna HN entra al MVP; sin pronóstico se usa `ReposicionMinMax` (FAL-04) y la ingesta al Lakehouse sí se diseña (observación 2) |

### EP-08 · Seguridad y acceso · **Must**

| Aspecto | Detalle |
|---|---|
| Objetivo | Que cada usuario haga solo lo que le corresponde |
| Módulos | EXT-04 y todos los SVC |
| HN | HN-24 (M, 3) |
| Roles | ROL-01 a ROL-07; Administrador gestiona roles (OPS-07) |
| Controles e interacciones | OPS-01 a OPS-08 · INT-02 · RNF-SEG-01 · FAL-07 |
| Diseñada en | HU-28 |
| Justificación | La autorización final vive en la capacidad dueña; es prerrequisito del inventario (orden 3) |

---

## 4. Secuencia de entrega por épica

Según la primera HN de cada épica en el product backlog.

| Posición | Épica | Primera HN (orden) |
|---|---|---|
| 1 | EP-01 | HN-01 (1) |
| 2 | EP-08 | HN-24 (3) |
| 3 | EP-02 | HN-03 (4) |
| 4 | EP-03 | HN-08 (6) |
| 5 | EP-06 | HN-20 (11) |
| 6 | EP-04 | HN-13 (13) |
| 7 | EP-05 | HN-17 (16) |
| 8 | EP-07 | HN-23 (23) |

---

## 5. Trazabilidad

| Criterio | Dónde se cumple |
|---|---|
| HU-04 CA1 (épicas EP-01 a EP-08) | §1 y §3 |
| HU-04 CA2 (módulo, prioridad MoSCoW, orden y HU que diseña) | §1, §3 y `HU-04-historias.md` |
| HU-04 CA4 (orden respeta dependencias) | §2 verificación 6 y §4 |
| Interesados vinculados a épicas (HU-03 CA4 de `docs/BACKLOG.md`) | §2 verificación 8 y `HU-02-vision.md` §2.1 |

Rúbrica: *Gestión del proyecto y sprints*.

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-23 | P2 | Versión inicial: validación y prioridad MoSCoW de EP-01 a EP-08 | HU-03 |
