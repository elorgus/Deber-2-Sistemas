# Smart Inventory · Dashboard v1: vistas y datos

| Campo | Valor |
|---|---|
| Documento | `equipo/P2-EstebanEsp07/HU-22-dashboard-v1.md` (borrador de `docs/dashboard/propuesta-dashboard.md`) |
| Versión | **1.0.0** |
| Estado | Propuesto para revisión (PR de HU-22) |
| Autor | P2 · Product Owner (`EstebanEsp07`) |
| Revisor | P3 · Scrum Master y gestor de versiones (`docs/BACKLOG.md` §1.2) |
| Fecha | 2026-11-24 |
| HU | **HU-22** Dashboard v1: vistas y datos · Sprint 2 · rúbrica: Trazabilidad y presentación |
| Épica | **EP-06** Dashboard operativo (TRX-01, TRX-02, TRX-04) |
| Historias | HN-20 y HN-21 (Must); HN-09 aporta la alerta de VL-02 |
| Diagramas | `DSH-planta-interna` y `DSH-planta-externa`, v1.0.0 |
| Contrato de referencia | `docs/ARQUITECTURA_BASE.md` v1.0.0 (§4.1, §6.2, §7.2, §8, §9) |

> Todos los identificadores son los del contrato. No se crearon identificadores nuevos. Lo que el contrato no cubre aparece como **PROPUESTA** en §6 y no se usa como oficial. Los datos de los bocetos son ilustrativos (enunciado §10).

---

## 1. Propósito y reglas

El dashboard (TRX-01 Dashboard, «LogistPulse · Centro de operaciones») es la interfaz de supervisión de Smart Inventory. Tiene dos modos, **Planta interna** y **Planta externa**, que comparten KPIs, alertas, flujo de pedidos y actividad reciente.

| Regla | Qué implica para el diseño | Fuente |
|---|---|---|
| El dashboard no guarda datos de negocio ni decide | Todo lo que muestra sale de la proyección de lectura (TRX-04); toda alerta la crea y resuelve su dueño | TRX-01, PRI-05, §8.3 |
| La proyección no es fuente de verdad | Para el dato exacto se consulta al dueño (INT-15) | PRI-07, FAL-06 |
| Sus botones solo envían comandos al dueño, a través de TRX-02 | Toda acción pasa por INT-01 y la autorización final la hace el módulo dueño | TRX-01, TRX-02, §11.4 |
| Consistencia eventual informada | Cada vista muestra «Actualizado hace X s» y su estado de frescura | §8.2, RNF-CON-02, CE-01 |

---

## 2. Catálogo de vistas VL-01 a VL-07 (CA1)

### 2.1 Resumen

| Vista | Contenido | Eventos de origen | Frescura objetivo (p95) | Acciones → dueño |
|---|---|---|---|---|
| **VL-01** KPIs operativos | Unidades disponibles, productos bajo mínimo, lotes por vencer, pedidos en preparación | EVT-05, EVT-12, EVT-13, EVT-14, EVT-15, EVT-21 a EVT-28 | ≤ 5 s | — |
| **VL-02** Alertas y acciones | Riesgos de agotamiento, sugerencias pendientes, lotes por vencer, pedidos en espera, AMR detenidos | EVT-09, EVT-12, EVT-14 a EVT-18, EVT-23, EVT-31 | ≤ 5 s | Aprobar o rechazar sugerencia → SVC-03 (INT-12); ver detalle de lote → SVC-02 (INT-15) |
| **VL-03** Flujo de pedidos | Cantidad de pedidos por estado (§9) | EVT-21 a EVT-28 | ≤ 5 s | Cancelar pedido → SVC-04 (OPS-05) |
| **VL-04** Planta interna | Racks y ocupación por zona, lotes, AMR (posición y estado), misiones | EVT-03, EVT-05, EVT-29 a EVT-32 | ≤ 2 s posiciones AMR; ≤ 5 s ocupación | Reasignar misión → SVC-05 (INT-33) |
| **VL-05** Planta externa | Muelles (ocupado o disponible), camiones en acceso, patio y zona de espera | EVT-26, EVT-33 a EVT-36 | ≤ 5 s | Registrar llegada o asignar muelle → SVC-05 (INT-33) |
| **VL-06** Actividad reciente | Últimos hechos de negocio relevantes | Todos los eventos de tipo *Negocio* | ≤ 5 s | — |
| **VL-07** Indicadores históricos | Rotación, quiebres, vencimientos y puntualidad | EVT-38; consulta INT-34 | Corte diario (≤ 24 h) | — |

Composición por modo (§8.1): **Planta interna** = VL-04 al centro + VL-01, VL-02, VL-03 y VL-06. **Planta externa** = VL-05 al centro + las mismas vistas compartidas. VL-07 se consulta aparte y no pertenece a ninguno de los dos modos.

### 2.2 De dónde sale cada dato

La proyección (TRX-04) solo consolida; el dueño de cada dato es quien publica el evento.

| Vista | Dato que se muestra | Evento(s) | Dueño que lo publica |
|---|---|---|---|
| VL-01 | Unidades disponibles | EVT-05 SaldoActualizado | SVC-02 |
| VL-01 | Productos bajo mínimo | EVT-14, EVT-15 | SVC-03 |
| VL-01 | Lotes por vencer | EVT-12, EVT-13 | SVC-02 |
| VL-01 | Pedidos en preparación | EVT-24, EVT-25 (y demás EVT-21 a EVT-28) | SVC-04 |
| VL-02 | Riesgos de agotamiento | EVT-14, EVT-15 | SVC-03 |
| VL-02 | Sugerencias pendientes | EVT-16, EVT-17, EVT-18 | SVC-03 |
| VL-02 | Lotes por vencer | EVT-12 | SVC-02 |
| VL-02 | Pedidos en espera | EVT-23 (causa: EVT-09) | SVC-04 (SVC-02) |
| VL-02 | AMR detenidos | EVT-31 EstadoAmrCambiado | SVC-05 |
| VL-03 | Pedidos por estado | EVT-21 a EVT-28 | SVC-04 |
| VL-04 | Ocupación por zona y lotes | EVT-03, EVT-05 | SVC-01, SVC-02 |
| VL-04 | Posición de AMR (señal, muestreada) | EVT-32 PosicionFlotaActualizada | SVC-05 |
| VL-04 | Estado de AMR y misiones | EVT-29, EVT-30, EVT-31 | SVC-05 |
| VL-05 | Muelles, camiones, acceso y espera | EVT-26, EVT-33, EVT-34, EVT-35, EVT-36 | SVC-04 (EVT-26), SVC-05 |
| VL-06 | Actividad reciente | Eventos de tipo *Negocio* (no EVT-32, que es señal) | Todos los dueños |
| VL-07 | Indicadores históricos | EVT-38 IndicadoresHistoricosActualizados | SVC-06 |

### 2.3 Cómo llegan los datos a la pantalla

| Tramo | Interacción | Modo |
|---|---|---|
| Usuario → dashboard → capa de aplicación | INT-01 | S |
| Autenticación y roles | INT-02 (TRX-02 → EXT-04) | S |
| Snapshot inicial y actualizaciones en vivo | INT-03 (TRX-02 → TRX-04) | S + A (push) |
| Eventos hacia la proyección | INT-08 (VL-01, VL-04, VL-06) e INT-32 (VL-04, VL-05) | A |
| Detalle exacto de existencias y lotes | INT-15 (TRX-02 → SVC-02) | S |
| Indicadores históricos | INT-34 (TRX-02 → SVC-06) | S |

---

## 3. Estados de frescura (resumen)

Cada vista muestra «Actualizado hace X s», calculado con el momento del último evento aplicado (§8.2). En esta versión el boceto solo reserva el espacio de ese texto; el indicador con sus tres estados (*En vivo*, *Con retraso*, *Desactualizada*) se diseña en HU-32.

---

## 4. Acciones del dashboard y su destino (CA2)

Toda acción viaja de TRX-01 a TRX-02 por INT-01. La columna INT indica la interacción que la lleva hasta el dueño. El rol autorizado se verifica en el dueño, no en la pantalla.

| # | Acción | Vista | Rol | SVC dueño | INT | Modo | Por qué va a ese dueño | Si el destino falla |
|---|---|---|---|---|---|---|---|---|
| 1 | Aprobar sugerencia | VL-02 | ROL-03 | SVC-03 | INT-12 | S | SVC-03 es dueño de sugerencias y OC; aprueba y emite la OC en una sola transacción (TX-04, OPS-03) | Acción no disponible; se informa al usuario |
| 2 | Rechazar sugerencia | VL-02 | ROL-03 | SVC-03 | INT-12 | S | SVC-03 es dueño del estado de la sugerencia (EVT-18) | Acción no disponible; se informa al usuario |
| 3 | Ver detalle de lote | VL-02 | ROL-01, ROL-02, ROL-03, ROL-05, ROL-06 *(propuesta de visibilidad, ver HU-28)* | SVC-02 | INT-15 | S | SVC-02 es dueño de existencias y lotes; la proyección no es fuente de verdad (PRI-07) | Se muestra la proyección marcada como «vista» |
| 4 | Cancelar pedido | VL-03 | ROL-05 | SVC-04 | **sin INT en el contrato** (OPS-05) | S | SVC-04 es dueño del estado del pedido; la cancelación dispara EVT-28 y la compensación (INT-31) | Ver §6, observación 1 |
| 5 | Reasignar misión | VL-04 | ROL-04 | SVC-05 | INT-33 | S | SVC-05 es dueño de misiones y AMR (OPS-08) | Acción no disponible; se informa al usuario |
| 6 | Registrar llegada | VL-05 | ROL-04 | SVC-05 | INT-33 | S | SVC-05 es dueño de visitas de vehículo; el registro manual cubre la falla de EXT-05 (INT-28) | Acción no disponible; se informa al usuario |
| 7 | Asignar muelle | VL-05 | ROL-04 | SVC-05 | INT-33 | S | SVC-05 es dueño de muelles; la asignación es una transacción local (TX-06) | Acción no disponible; se informa al usuario |

Alternar entre Planta interna y Planta externa no es un comando: solo cambia el mapa central y lee el snapshot de la otra vista (INT-01 → INT-03).

---

## 5. Bocetos (CA3)

Basados en las Figuras 3 y 4 del enunciado. Fuente PlantUML y exportación SVG con el mismo nombre; leyenda dentro del diagrama.

**Planta interna** · `DSH-planta-interna` v1.0.0 · HU-22

![DSH-planta-interna](diagramas/dashboard/HU-22/export/DSH-planta-interna.svg)

**Planta externa** · `DSH-planta-externa` v1.0.0 · HU-22

![DSH-planta-externa](diagramas/dashboard/HU-22/export/DSH-planta-externa.svg)

| Diagrama | Fuente | Exportación |
|---|---|---|
| DSH-planta-interna | `diagramas/dashboard/HU-22/src/DSH-planta-interna.puml` | `diagramas/dashboard/HU-22/export/DSH-planta-interna.svg` |
| DSH-planta-externa | `diagramas/dashboard/HU-22/src/DSH-planta-externa.puml` | `diagramas/dashboard/HU-22/export/DSH-planta-externa.svg` |

Elementos tomados del enunciado: KPIs (Unidades disponibles 12.840, Productos bajo mínimo 18, Lotes por vencer 7, Pedidos en preparación 24), Pasillo A, Zona C · 74 %, AMR-03 Picking, muelles 1 a 4, camión T-2205, Acceso principal, Zona de espera y Tractor de patio.

---

## 6. Observaciones y propuestas de cambio

No cambian el contrato. Cualquier ajuste se tramita con PR (y ADR si aplica) según `ARQUITECTURA_BASE` §0.3.

1. **Cancelar pedido no tiene INT.** VL-03 ofrece «Cancelar pedido → SVC-04 (OPS-05)», pero §7.2 no define la interacción TRX-02 → SVC-04 para cancelar (sí existen INT-17 y INT-25 para registrar y confirmar). **PROPUESTA DE CAMBIO:** agregar esa interacción síncrona en la matriz de HU-24 (P6, con P1). Hasta entonces el boceto la marca «INT pendiente».
2. **«Sin señal» de un AMR.** RNF-DIS-03 y FAL-03 piden que VL-04 marque «sin señal» a un AMR tras 10 s sin telemetría, pero «sin señal» no es un estado de AMR (§9) y la proyección no genera alertas (PRI-05). **PROPUESTA:** que SVC-05 lo comunique con EVT-31 y aclararlo en HU-19 y HU-24.
3. **Alimentación de VL-02 y VL-03.** INT-08 e INT-32 citan VL-01, VL-04, VL-05 y VL-06, pero ningún INT cita el tramo broker → proyección para VL-02 y VL-03, aunque §6.2 ya lista a TRX-04 como consumidor de esos eventos. **PROPUESTA:** completarlo en la matriz de HU-24.
4. **Pedidos en *PendienteReserva*.** Ese estado no publica evento (§9), por lo que VL-03 no puede contarlos. **PROPUESTA:** mostrarlos solo como parte de «Registrado» con consulta al dueño, o aceptar que no se ven; decidirlo con P6 y P1.
5. **Cronómetro de HN-09.** La alerta en VL-02 pide ≤ 5 s, mientras CE-03 y CE-01 podrían sumar hasta ~10 s; aclarar el punto de inicio en HU-27.

---

## 7. Trazabilidad

| Criterio (`docs/BACKLOG.md` §4.2) | Dónde se cumple |
|---|---|
| CA1. VL-01 a VL-07 con contenido, eventos de origen y frescura objetivo, coherentes con ARQUITECTURA_BASE §8 | §2.1 y §2.2 |
| CA2. Cada acción indica su SVC e INT | §4 |
| CA3. Boceto de planta interna y externa basado en las Figuras 3 y 4, en `diagramas/dashboard/` | §5 |

Épica EP-06 · HN-20, HN-21 · Rúbrica *Trazabilidad y presentación*. Continúa en HU-32 (`HU-32-dashboard-v2.md`).

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-24 | P2 | Versión inicial: vistas VL-01 a VL-07, acciones y bocetos de planta interna y externa | HU-22 |
