# Flujo vertical · Recorrido integral: recepción → actualización de stock → alerta → reposición

| Campo | Valor |
|---|---|
| Documento | `docs/arquitectura/flujos/recorrido-integral.md` |
| Versión | **1.0.0** |
| Fecha | 2026-11-27 |
| Autor | P6 · Diseñador de operación (Pedidos y Flota) |
| Revisor | P5 |
| Historia | HU-38 · Recorrido integral del flujo vertical |
| Contrato de referencia | `docs/ARQUITECTURA_BASE.md` v1.0.0 (§3, §5, §6.3, §7.2, §8.1) |

---

## 1. Propósito y alcance

Este documento describe de extremo a extremo la cadena exigida por el contrato (§6.3): el operador registra una **recepción**, el **stock** se actualiza, la baja del disponible dispara una **alerta** de riesgo de agotamiento y la **reposición** se resuelve con una orden de compra (OC) que, al recibirse, cierra el riesgo.

Es la base visual de la presentación final (HU-40) y reúne en un solo recorrido los flujos A (recepción), B (reposición) y C (reserva y despacho). Para cada paso se indica el dueño, el modo de interacción (S o A), la interacción (`INT`), los eventos (`EVT`) y lo que ve el dashboard.

---

## 2. Recorrido paso a paso

| Paso | Qué ocurre | Dueño | Modo | Interacción | Eventos | Qué ve el dashboard |
|---|---|---|---|---|---|---|
| 1 | El operador (`ROL-01`) registra la recepción del SKU X (lote, vencimiento, ubicación). | `ROL-01` → `SVC-02` | S | `INT-01`, `INT-06` | — | — |
| 2 | `SVC-02` guarda lote, movimiento, saldo y outbox en una sola transacción (`TX-01`) y publica los hechos. | `SVC-02` | A | `INT-07`, `INT-08` | `EVT-04` `RecepcionRegistrada`, `EVT-05` `SaldoActualizado` | `VL-01` unidades, `VL-04` y `VL-06` (`INT-08`) |
| 3 | Los pedidos consumen stock: la reserva FEFO baja el disponible de X (`TX-02`) y, al despachar, la salida descuenta el saldo (`TX-03`). | `SVC-04` → `SVC-02` | S (reserva) y A (resultado y salida) | `INT-19`, `INT-20`, `INT-30` | `EVT-08`, `EVT-11`, `EVT-05` (`EVT-27` dispara `INT-30`) | `VL-01` (`INT-08`) y `VL-03` (INT por definir) |
| 4 | `SVC-03` evalúa `EVT-05` contra la política de X: el disponible queda bajo el mínimo. | `SVC-03` | A | `INT-10`, `INT-11` | `EVT-05` (entrada), `EVT-14` `RiesgoAgotamientoDetectado` | `VL-02` alerta (INT por definir); `VL-01` «Productos bajo mínimo» (`INT-08`) |
| 5 | `SVC-03` calcula la sugerencia de reposición (mínimo, demanda, lead time). | `SVC-03` | A | `INT-11` | `EVT-16` `SugerenciaReposicionGenerada` | `VL-02` botón «Aprobar» (INT por definir) |
| 6 | El comprador (`ROL-03`) aprueba; `SVC-03` valida el estado vigente y crea la OC (`TX-04`). | `ROL-03` → `SVC-03` | S (aprobación) y A (publicación) | `INT-01`, `INT-12`, `INT-13` | `EVT-17` `SugerenciaReposicionAprobada`, `EVT-19` `OrdenCompraEmitida` | `VL-02` actualizada (INT por definir) |
| 7 | La OC se envía al ERP mediante adaptador. | `SVC-03` → `EXT-01` | A (con reintentos) | `INT-14` | — | — |
| 8 | Llega el camión del proveedor (garita `EXT-05`) y se recibe la OC: `SVC-02` repite `TX-01` con la OC como referencia. | `SVC-05` y `SVC-02` | A (llegada y publicación) y S (recepción) | `INT-28`, `INT-01`, `INT-06`, `INT-07`, `INT-32`, `INT-08` | `EVT-33` `VehiculoArribado`, `EVT-04`, `EVT-05` | `VL-05` (`INT-32`) y `VL-01` (`INT-08`) |
| 9 | `SVC-03` cierra la OC y, al recuperarse el disponible, resuelve el riesgo. | `SVC-03` | A | `INT-09`, `INT-10`, `INT-11` | `EVT-20` `OrdenCompraCerrada`, `EVT-15` `RiesgoAgotamientoResuelto` | La alerta desaparece de `VL-02` (INT por definir) |

---

## 3. Cómo se conecta con los flujos A, B y C

| Pasos | Flujo | Diagrama |
|---|---|---|
| 1 y 2 | A · Recepción | `SEC-FA` |
| 3 | C · Reserva FEFO y despacho | `SEC-FC` |
| 4 a 7 | B · Reposición y compras | `SEC-FB` |
| 8 y 9 | A y B (`INT-28` también participa en el flujo C) | `SEC-FA`, `SEC-FB` |

---

## 4. Propiedad de datos en el recorrido

- `SVC-02` es el único dueño de lote, movimiento, saldo y reserva. `SVC-04` solo pide la reserva (`INT-19`) y avisa del despacho (`INT-30`); no escribe en datos de Inventario (`PRI-02`).
- `SVC-03` es el único dueño de la política, la alerta, la sugerencia y la OC. Evalúa con su réplica del disponible (`EVT-05`), no consultando a `SVC-02`.
- `SVC-05` es el único dueño de la visita del vehículo; solo informa la llegada con `EVT-33`.
- `TRX-04` solo consolida las vistas a partir de eventos. No decide ni es fuente de verdad (`PRI-05`, `PRI-07`): el botón «Aprobar» envía el comando a `SVC-03` (`INT-12`), que valida el estado vigente.

---

## 5. Notas de lectura

- La tabla de §6.3 cita una `INT` por paso. Aquí se muestran también las de publicación (`INT-11`, `INT-13`, `INT-20`) y las del dashboard (`INT-08`, `INT-32`) tal como las define §7.2.
- `INT-01` (`TRX-01` → `TRX-02`) aparece en los pasos donde un usuario actúa desde el dashboard (1, 6 y 8).
- En el paso 3, `EVT-27` (`PedidoDespachado`) no figura en la columna de eventos de §6.3, pero es el que dispara `INT-30`; se muestra por coherencia con `SEC-FC`.
- En el paso 6, `EVT-19` también lo consumen `SVC-02` y `SVC-05`, que mantienen una réplica de las OC esperadas (§3.2, §3.5).
- La publicación de `EVT-20` es resultado de `INT-09`: §7.2 no define una `INT` aparte para publicarlo.

---

## 6. Diagramas asociados

- **Secuencia del recorrido integral:** `diagramas/secuencia/src/SEC-VERTICAL-recorrido-integral.puml` (exportación en `diagramas/secuencia/export/SEC-VERTICAL-recorrido-integral.svg`).
- **Flujos que lo componen:** `SEC-FA`, `SEC-FB` y `SEC-FC`.

---

## 7. Puntos abiertos

| ID | Punto abierto | Estado |
|---|---|---|
| INT por definir | Actualización de `VL-02` y `VL-03` en `TRX-04`. En §7.2, `INT-08` cubre `VL-01`, `VL-04` y `VL-06`, e `INT-32` cubre `VL-04` y `VL-05`; ninguna cubre `VL-02` ni `VL-03`. Afecta a los pasos 3, 4, 5, 6 y 9. | Abierto; se propone vía §0.3 (PR + ADR «Propuesto») |

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-27 | P6 | Versión inicial | HU-38 |
