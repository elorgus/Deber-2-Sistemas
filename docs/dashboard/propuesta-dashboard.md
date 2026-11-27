# Smart Inventory · Dashboard v2: vistas interna y externa

| Campo | Valor |
|---|---|
| Documento | `equipo/P2-EstebanEsp07/HU-32-dashboard-v2.md` (versión final de `docs/dashboard/propuesta-dashboard.md`) |
| Versión | **1.1.0** (MINOR: agrega, de forma compatible, modos, indicador de frescura y capturas anotadas sobre la v1.0.0 de HU-22) |
| Estado | Propuesto para revisión (PR de HU-32) |
| Autor | P2 · Product Owner (`EstebanEsp07`) |
| Revisor | P3 · Scrum Master y gestor de versiones (`docs/BACKLOG.md` §1.2) |
| Fecha | 2026-11-24 |
| HU | **HU-32** Dashboard v2: vistas interna y externa · Sprint 3 · rúbrica: Trazabilidad y presentación |
| Épica | **EP-06** Dashboard operativo (TRX-01, TRX-02, TRX-04) |
| Historias | HN-20 y HN-21 (Must) |
| Diagramas | `DSH-planta-interna`, `DSH-planta-externa` y `DSH-frescura-estados`, v1.1.0 |
| Depende de | HU-22 (`HU-22-dashboard-v1.md`) y HU-28 (`HU-28-seguridad-roles.md`) |
| Contrato de referencia | `docs/ARQUITECTURA_BASE.md` v1.0.0 (§6.2, §7.2, §8, §9, §11.3) |

> Todos los identificadores son los del contrato. No se crearon identificadores nuevos. Lo que el contrato no cubre aparece como **PROPUESTA** en §7. Los datos de los bocetos son ilustrativos (enunciado §10).

**Propósito.** Explicar cómo se supervisa la operación en la planta interna y en la externa, y cómo se informa al usuario que lo que ve es consistente solo de forma eventual.

**Cambios respecto a la v1.0.0 (HU-22)**

| Cambio | Dónde |
|---|---|
| Modos Planta interna y Planta externa, y vistas compartidas | §1 |
| Indicador de frescura con sus 3 estados, en cada vista | §3 |
| Bocetos actualizados con el indicador y capturas anotadas (VL e INT en cada elemento) | §4 |
| Nuevo diagrama `DSH-frescura-estados` | §3.3 |
| Visibilidad por rol alineada con HU-28 | §5 |

---

## 1. Modos y vistas compartidas (CA1)

### 1.1 Composición de cada modo

| | **Planta interna** | **Planta externa** |
|---|---|---|
| Mapa central | **VL-04**: racks, ocupación por zona, lotes, AMR (posición y estado) y misiones | **VL-05**: muelles, camiones en acceso, patio y zona de espera |
| Qué se supervisa | Interior de la planta: almacenamiento y flota AMR | Exterior: llegada y salida de camiones y uso de muelles |
| Acción propia del mapa | Reasignar misión → SVC-05 (INT-33) | Registrar llegada o asignar muelle → SVC-05 (INT-33) |
| Eventos que lo alimentan | EVT-03, EVT-05, EVT-29 a EVT-32 | EVT-26, EVT-33 a EVT-36 |
| Frescura objetivo (p95) | ≤ 2 s posiciones AMR; ≤ 5 s ocupación | ≤ 5 s |

### 1.2 Vistas compartidas por ambos modos

| Vista | Contenido | Frescura objetivo | Acciones → dueño |
|---|---|---|---|
| **VL-01** KPIs operativos | Unidades disponibles, productos bajo mínimo, lotes por vencer, pedidos en preparación | ≤ 5 s | — |
| **VL-02** Alertas y acciones | Riesgos de agotamiento, sugerencias pendientes, lotes por vencer, pedidos en espera, AMR detenidos | ≤ 5 s | Aprobar o rechazar → SVC-03 (INT-12); ver detalle de lote → SVC-02 (INT-15) |
| **VL-03** Flujo de pedidos | Pedidos por estado (§9) | ≤ 5 s | Cancelar pedido → SVC-04 (OPS-05) |
| **VL-06** Actividad reciente | Últimos hechos de negocio relevantes | ≤ 5 s | — |

**VL-07** (indicadores históricos; EVT-38, INT-34, corte diario) no pertenece a ningún modo: se consulta aparte.

### 1.3 Qué ocurre al cambiar de modo (HN-21)

- **Cambia solo el mapa central** (VL-04 ↔ VL-05).
- **Se mantienen** KPIs, alertas, flujo de pedidos y actividad reciente (VL-01, VL-02, VL-03 y VL-06).
- Cambiar de modo **no es un comando**: TRX-01 lee el snapshot de la otra vista (INT-01 → INT-03) y se suscribe a sus actualizaciones.
- Las vistas compartidas conservan su estado de frescura; no se reinician al cambiar de modo.

---

## 2. Origen de los datos y acciones (resumen de HU-22)

El detalle completo de eventos de origen, dueños y acciones está en `HU-22-dashboard-v1.md` §2 y §4. En este documento solo se repite lo necesario para leer los bocetos.

| Acción | Vista | Rol | SVC dueño | INT |
|---|---|---|---|---|
| Aprobar o rechazar sugerencia | VL-02 | ROL-03 | SVC-03 | INT-12 |
| Ver detalle de lote | VL-02 | Ver §5 | SVC-02 | INT-15 |
| Cancelar pedido | VL-03 | ROL-05 | SVC-04 | sin INT en el contrato (OPS-05, §7 obs. 1) |
| Reasignar misión | VL-04 | ROL-04 | SVC-05 | INT-33 |
| Registrar llegada o asignar muelle | VL-05 | ROL-04 | SVC-05 | INT-33 |

Toda acción viaja de TRX-01 a TRX-02 por INT-01; la sesión y los roles se obtienen con INT-02; el snapshot y las actualizaciones en vivo llegan por INT-03.

---

## 3. Indicador de frescura (CA2)

### 3.1 Estados

Cada vista muestra **«Actualizado hace X s»**, calculado con el momento del último evento aplicado (`ARQUITECTURA_BASE` §8.2, RNF-CON-02).

| Estado | Condición | Qué ve el usuario | Acciones |
|---|---|---|---|
| ● **En vivo** | Retraso ≤ objetivo de la vista | Indicador verde y «Actualizado hace X s» | Disponibles |
| ● **Con retraso** | Retraso > objetivo y ≤ 60 s | Indicador ámbar y hora del último dato | Disponibles |
| ● **Desactualizada** | Retraso > 60 s o conexión perdida | Banner rojo «Los datos pueden no estar al día» | Disponibles: siempre se validan en el dueño (PRI-07) |

### 3.2 Reglas de diseño

- **Cada vista tiene su propio indicador**, porque cada una tiene su objetivo de frescura (§8.1). Una vista puede estar *En vivo* y otra *Con retraso*.
- El objetivo de cada estado es el de la vista: 5 s para VL-01, VL-02, VL-03, VL-05 y VL-06; 2 s para las posiciones de AMR y 5 s para la ocupación en VL-04.
- La frescura **no bloquea acciones**: la pantalla nunca autoriza ni decide (HU-28 §6).
- Al recuperarse los eventos, la vista vuelve a *En vivo* sola; no requiere acción del usuario.

| Causa de pérdida de frescura | Escenario | Efecto en el dashboard |
|---|---|---|
| Broker no disponible | FAL-01 | Las vistas pasan a *Desactualizada*; los eventos esperan en el outbox y se publican en orden al volver |
| Proyección caída o corrupta | FAL-06 | El dashboard avisa y el detalle se consulta al dueño (INT-15); la proyección se reconstruye reprocesando eventos |
| Inventario no disponible | FAL-02 | El dashboard muestra la última vista con su marca de frescura |
| Analítica no disponible | FAL-04 | VL-07 muestra el último corte; sin impacto operativo |
| Telemetría de un AMR perdida (> 10 s) | FAL-03, RNF-DIS-03 | VL-04 marca el AMR «sin señal»; ver §7 obs. 2 |

### 3.3 Diagrama de estados

`DSH-frescura-estados` v1.1.0 · HU-32

![DSH-frescura-estados](diagramas/dashboard/HU-32/export/DSH-frescura-estados.svg)

---

## 4. Capturas anotadas (CA3)

Convención de las anotaciones: **(n)** marca cada elemento visual; cada vista indica su **VL** y cada botón su **INT** (u **OPS** cuando el contrato aún no define la interacción). El indicador de frescura aparece en cada vista.

### 4.1 Planta interna

`DSH-planta-interna` v1.1.0 · HU-32

![DSH-planta-interna](diagramas/dashboard/HU-32/export/DSH-planta-interna.svg)

| Marca | Elemento | VL / módulo | Datos (eventos) | Botón → INT |
|---|---|---|---|---|
| (1) | Encabezado, selector de modo y sesión | TRX-01 | — | Planta externa → INT-01, INT-03 · sesión → INT-02 |
| (2) | KPIs operativos | VL-01 | EVT-05, EVT-12 a EVT-15, EVT-21 a EVT-28 | — |
| (3) | Mapa de planta interna: racks, lotes y AMR | VL-04 | EVT-03, EVT-05, EVT-29 a EVT-32 | — |
| (4) | Reasignar misión | VL-04 | — | INT-33 (OPS-08, ROL-04) |
| (5) | Alertas y acciones | VL-02 | EVT-09, EVT-12, EVT-14 a EVT-18, EVT-23, EVT-31 | Aprobar y Rechazar → INT-12 · Ver detalle → INT-15 |
| (6) | Flujo de pedidos por estado | VL-03 | EVT-21 a EVT-28 | — |
| (7) | Cancelar pedido | VL-03 | — | OPS-05 (ROL-05); **INT pendiente** (§7 obs. 1) |
| (8) | Actividad reciente | VL-06 | Eventos de tipo *Negocio* | — |

### 4.2 Planta externa

`DSH-planta-externa` v1.1.0 · HU-32

![DSH-planta-externa](diagramas/dashboard/HU-32/export/DSH-planta-externa.svg)

En este boceto VL-05 se muestra *Con retraso* (hace 18 s, objetivo ≤ 5 s) para ilustrar un estado distinto de *En vivo* en una vista del mismo tablero.

| Marca | Elemento | VL / módulo | Datos (eventos) | Botón → INT |
|---|---|---|---|---|
| (1) | Encabezado, selector de modo y sesión | TRX-01 | — | Planta interna → INT-01, INT-03 · sesión → INT-02 |
| (2) | KPIs operativos | VL-01 | EVT-05, EVT-12 a EVT-15, EVT-21 a EVT-28 | — |
| (3) | Mapa de planta externa: muelles, patio, acceso y espera | VL-05 | EVT-26, EVT-33 a EVT-36 | — |
| (4) | Registrar llegada y Asignar muelle | VL-05 | — | INT-33 (ROL-04) |
| (5) | Alertas y acciones | VL-02 | EVT-09, EVT-12, EVT-14 a EVT-18, EVT-23, EVT-31 | Aprobar y Rechazar → INT-12 · Ver detalle → INT-15 |
| (6) | Flujo de pedidos por estado | VL-03 | EVT-21 a EVT-28 | — |
| (7) | Cancelar pedido | VL-03 | — | OPS-05 (ROL-05); **INT pendiente** (§7 obs. 1) |
| (8) | Actividad reciente | VL-06 | Eventos de tipo *Negocio* | — |

### 4.3 Archivos de diagrama

| Diagrama | Versión | Fuente | Exportación |
|---|---|---|---|
| DSH-planta-interna | 1.1.0 | `diagramas/dashboard/HU-32/src/DSH-planta-interna.puml` | `diagramas/dashboard/HU-32/export/DSH-planta-interna.svg` |
| DSH-planta-externa | 1.1.0 | `diagramas/dashboard/HU-32/src/DSH-planta-externa.puml` | `diagramas/dashboard/HU-32/export/DSH-planta-externa.svg` |
| DSH-frescura-estados | 1.1.0 | `diagramas/dashboard/HU-32/src/DSH-frescura-estados.puml` | `diagramas/dashboard/HU-32/export/DSH-frescura-estados.svg` |

---

## 5. Visibilidad por rol (alineada con HU-28)

Según la matriz de `HU-28-seguridad-roles.md` §3 (filas 30 y 31, **PROPUESTA**):

| Vista | ROL-01 | ROL-02 | ROL-03 | ROL-04 | ROL-05 | ROL-06 | ROL-07 |
|---|---|---|---|---|---|---|---|
| VL-01 a VL-06 | L | L | L | L | L | L | — |
| VL-07 | — | — | — | — | L | L | — |

L = solo consulta. Los botones se muestran según el rol, pero la autorización la hace siempre el módulo dueño (HU-28 §6).

---

## 6. Trazabilidad

| Criterio (`docs/BACKLOG.md` §4.3) | Dónde se cumple |
|---|---|
| CA1. Explica los modos Planta interna y Planta externa y las vistas que comparten | §1 |
| CA2. El indicador de frescura con sus 3 estados (§8.2) aparece en el boceto | §3 y §4 (el indicador se ve en cada vista de los bocetos y en `DSH-frescura-estados`) |
| CA3. Capturas anotadas: cada elemento visual indica su VL y cada botón su INT | §4 |

HN-20 (antigüedad del dato) y HN-21 (alternar modos) · EP-06 · Rúbrica *Trazabilidad y presentación*. Insumo de HU-34, HU-35 y HU-39.

---

## 7. Observaciones y propuestas de cambio

Heredan las de HU-22 (§6) y no cambian el contrato; se tramitan con PR (y ADR si aplica) según `ARQUITECTURA_BASE` §0.3.

1. **Cancelar pedido sin INT.** El contrato no define la interacción TRX-02 → SVC-04 para cancelar. **PROPUESTA:** agregarla en la matriz de HU-24 (P6, con P1). Hasta entonces los bocetos la marcan «INT pendiente».
2. **«Sin señal» de un AMR.** No es un estado de AMR (§9) y la proyección no genera alertas (PRI-05). **PROPUESTA:** que SVC-05 lo comunique con EVT-31 (HU-19, HU-24).
3. **Frescura de VL-07.** Su objetivo es el corte diario, no los 5 s de las demás vistas. **PROPUESTA:** que VL-07 muestre «Corte del <fecha>» y el banner de *Desactualizada* si pasan más de 24 h sin corte; no usa *Con retraso*.
4. **Pedidos en *PendienteReserva*.** Ese estado no publica evento (§9), por lo que VL-03 no puede contarlos. **PROPUESTA:** decidir con P6 y P1 si se muestran mediante consulta al dueño o se aceptan como no visibles.
5. **Cronómetro de HN-09.** La alerta en VL-02 pide ≤ 5 s, mientras CE-03 y CE-01 podrían sumar hasta ~10 s; aclarar el punto de inicio de la medición en HU-27.

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-24 | P2 | Versión inicial: vistas VL-01 a VL-07, acciones y bocetos | HU-22 |
| 1.1.0 | 2026-11-24 | P2 | Modos y vistas compartidas, indicador de frescura con 3 estados, capturas anotadas y visibilidad por rol | HU-32 |
