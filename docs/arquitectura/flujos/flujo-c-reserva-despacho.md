# Flujo C · Reserva FEFO, preparación y despacho

| Campo | Valor |
|---|---|
| Documento | `docs/arquitectura/flujos/flujo-c-reserva-despacho.md` |
| Versión | **1.0.0** |
| Fecha | 2026-11-24 |
| Autor | P6 · Diseñador de operación (Pedidos y Flota) |
| Revisor | P5 |
| Historia | HU-18 · Flujo C: reserva FEFO, preparación y despacho |
| Historias de negocio | HN-13, HN-14, HN-15, HN-16, HN-17, HN-18, HN-19 |
| Contrato de referencia | `docs/ARQUITECTURA_BASE.md` v1.0.0 (§3.2, §3.4, §3.5, §6.2, §7, §9) |

---

## 1. Propósito y alcance

Este documento describe el flujo de extremo a extremo desde el ingreso de un pedido hasta su despacho y el descuento definitivo de existencias. Coordina tres capacidades con propiedad de datos estrictamente separada (`PRI-01`, `PRI-02`, `PRI-03`):

- **`SVC-04` Pedidos y Fulfillment:** dueño del `Pedido` (y su `EstadoPedido`), la `Tarea de preparación` y la `Solicitud de despacho`.
- **`SVC-02` Inventario:** fuente única de verdad de `Saldo`, `Lote`, `fecha de vencimiento`, `Movimiento` y `Reserva` (aplica `PoliticaFEFO`).
- **`SVC-05` Flota y Operación:** dueño de `AMR`, `Misión AMR`, `Ruta`, `Telemetría`, `Visita de vehículo`, `Muelle` y `Patio`.

---

## 2. Flujo principal (camino feliz)

| Paso | Qué ocurre | Origen → Destino | Interacción | Modo | Eventos / Transacción |
|---|---|---|---|---|---|
| 1 | Ingresa un pedido manual (`ROL-05`) o importado desde `EXT-01`. `SVC-04` registra el pedido en estado `Registrado`. | `TRX-02` → `SVC-04` / `EXT-01` → `SVC-04` | `INT-17` / `INT-18` | S / A | `TX-05`, `EVT-21` (`PedidoRegistrado`) |
| 2 | `SVC-04` solicita la reserva FEFO a `SVC-02` mediante `PuertoReservasInventario` → `GestionReservas`. | `SVC-04` → `SVC-02` | `INT-19` | **S** (timeout + Circuit Breaker) | `TX-02` en `SVC-02` |
| 3 | `SVC-02` verifica disponible, asigna lotes con `PoliticaFEFO`, guarda la reserva y publica el hecho vía Outbox. | `SVC-02` → `TRX-03` | `INT-20` | A | `EVT-08` (`ReservaConfirmada`), `EVT-05` (`SaldoActualizado`) |
| 4 | `SVC-04` guarda el ID de reserva y una copia de solo lectura de la asignación por lote, pasa el pedido a `Reservado` y luego a `EnPreparacion`. | `SVC-04` → `TRX-03` → `SVC-05` | `INT-21` | A | `TX-05`, `EVT-22` (`PedidoReservado`), `EVT-24` (`PreparacionSolicitada`) |
| 5 | `SVC-05` selecciona un AMR disponible (`EstrategiaAsignacionAmr`), calcula la ruta usando su réplica de ubicaciones (`EVT-03`) y envía la misión a `EXT-03`. | `SVC-05` → `EXT-03` | `INT-22` | S | `EVT-29` (`MisionAmrAsignada`) |
| 6 | `EXT-03` emite telemetría continua hacia `SVC-05`; `SVC-05` publica la señal muestreada `EVT-32` y cambios de estado a `TRX-04` (`VL-04`). | `EXT-03` → `SVC-05` → `TRX-03` → `TRX-04` | `INT-23`, `INT-32` | A | `EVT-31`, `EVT-32` (Señal) |
| 7 | Al finalizar el AMR, `SVC-05` notifica la finalización de la misión a `SVC-04`. | `SVC-05` → `TRX-03` → `SVC-04` | `INT-24` | A | `EVT-29`, `EVT-30` (`MisionAmrCompletada`) |
| 8 | El operador (`ROL-01`) confirma la preparación en `SVC-04`. El pedido pasa a `Preparado` y luego a `EnDespacho`. | `TRX-02` → `SVC-04` → `TRX-03` → `SVC-05` | `INT-25`, `INT-26` | S / A | `TX-05`, `EVT-25` (`PreparacionCompletada`), `EVT-26` (`DespachoSolicitado`) |
| 9 | `SVC-05` coordina el vehículo con `EXT-02`, registra la llegada en garita (`EXT-05`), asigna muelle (`TX-06`) y actualiza `VL-05`. | `SVC-05` ↔ `EXT-02`, `EXT-05` → `SVC-05` → `TRX-04` | `INT-27`, `INT-28`, `INT-32` | A | `TX-06`, `EVT-33` (`VehiculoArribado`), `EVT-34` (`MuelleAsignado`) |
| 10 | El camión cargado sale del patio (`EXT-05`), `SVC-05` libera el muelle y notifica el despacho a `SVC-04`. | `SVC-05` → `TRX-03` → `SVC-04` | `INT-29`, `INT-32` | A | `EVT-35` (`VehiculoDespachado`), `EVT-36` (`MuelleLiberado`) |
| 11 | `SVC-04` cierra el pedido en estado `Despachado` y notifica a `SVC-02`. | `SVC-04` → `TRX-03` → `SVC-02` | `INT-30` | A | `TX-05`, `EVT-27` (`PedidoDespachado`) |
| 12 | `SVC-02` consume la reserva, registra el movimiento de salida y actualiza el saldo en una transacción local. | `SVC-02` → `TRX-03` | `INT-30` | A | `TX-03`, `EVT-11` (`SalidaRegistrada`), `EVT-05` (`SaldoActualizado`) |

---

## 3. Ejemplo concreto de selección FEFO (CA2)

Supóngase un pedido de **80 unidades** del SKU `LEC-1L` (controla lote y vencimiento, con vida útil mínima de despacho de 5 días) registrado el **1 de diciembre**:

| Lote | Fecha de vencimiento | Estado del lote | Saldo disponible | ¿Elegible para FEFO? | Cantidad asignada |
|---|---|---|---|---|---|
| `L-001` | 28-nov | `Vencido` | 0 (excluido) | No (lote vencido) | 0 |
| `L-002` | 15-dic | `EnCuarentena` | 0 (excluido) | No (en cuarentena) | 0 |
| `L-003` | 10-dic | `Disponible` | 50 u. | Sí (vence primero y cumple vida útil mínima) | **50 u.** |
| `L-004` | 20-dic | `Disponible` | 100 u. | Sí (siguiente vencimiento más próximo) | **30 u.** |

- `SVC-02` aplica `PoliticaFEFO` dentro de `TX-02`, asigna **50 u. de `L-003`** y **30 u. de `L-004`** (total 80 u.), y devuelve la asignación a `SVC-04` en la respuesta de `INT-19`.
- `SVC-04` no elige lotes ni modifica saldos: solo conserva el identificador de la reserva y la copia de solo lectura de esta asignación para armar la lista de preparación.

---

## 4. Caminos alternativos y manejo de fallos (CA3)

### 4.1 Sin stock suficiente (`EnEspera`)
- En el MVP la reserva es **todo o nada**. Si el disponible total de lotes elegibles es menor a lo solicitado, `SVC-02` rechaza la reserva en `INT-19` y publica `EVT-09` (`ReservaRechazada`) vía `INT-20` (consumido por `SVC-03` como demanda no atendida).
- `SVC-04` transiciona el pedido a estado `EnEspera` y publica `EVT-23` (`PedidoEnEspera`), visible en `VL-02` y `VL-03`. No se realiza ninguna reserva parcial.

### 4.2 `SVC-02` Inventario no disponible (`PendienteReserva` · `FAL-02`)
- Si `INT-19` excede el tiempo límite o se abre el Circuit Breaker de `AdaptadorReservasInventario`, ningún pedido se pierde (`RNF-DIS-02`).
- `SVC-04` guarda el pedido en estado `PendienteReserva` (sin emitir evento de negocio) y reintenta `INT-19` usando el **identificador del pedido como clave de idempotencia**, evitando reservas duplicadas cuando `SVC-02` se recupere.

### 4.3 Cancelación del pedido con compensación (`Cancelado`)
- Un pedido puede cancelarse únicamente antes de entrar en `EnDespacho` (si ya está en `EnPreparacion`, requiere `ROL-05` y motivo obligatorio según `OPS-05`).
- `SVC-04` transiciona el pedido a `Cancelado` (`TX-05`) y publica `EVT-28` (`PedidoCancelado`).
- Mediante `INT-31`, `SVC-02` libera la reserva (`EVT-10` `ReservaLiberada`, recuperando el disponible en `≤ 60 s` según `CE-05`) y `SVC-05` aborta o cancela las misiones AMR asociadas.

### 4.4 Degradación en Flota (`FAL-03` y `FAL-09`)
- Si un AMR pierde telemetría por más de 10 s (`FAL-03`, `RNF-DIS-03`), se marca "sin señal" en `VL-04`, su misión pasa a `Fallida` y se reasigna (`INT-33`) o se completa por picking manual sin afectar a `SVC-02` ni `SVC-04`.
- Si `EXT-02` no responde (`FAL-09`), el pedido permanece en `EnDespacho` y `ROL-04` puede asignar el vehículo manualmente (`INT-33`).

---

## 5. Justificación de modos de interacción: síncrono vs. asíncrono (CA4)

- **Por qué `INT-19` (Solicitar reserva FEFO) es síncrona (`S`):**
  1. **Protección de invariante de negocio (criterio S2):** el saldo disponible nunca puede ser negativo (`RNF-CON-01`). `SVC-04` necesita confirmación inmediata de `SVC-02` (`TX-02`) antes de generar la tarea de preparación (`EVT-24`) y comprometer recursos de bodega.
  2. **Dato exacto en el instante:** `SVC-04` requiere la asignación exacta por lote y ubicación calculada por `PoliticaFEFO` para iniciar el picking.
  3. **Acotamiento de riesgo:** se protege con tiempo límite, Circuit Breaker y reintento idempotente (`FAL-02`), cumpliendo el límite de máximo dos saltos síncronos (`TRX-02` → `SVC-04` → `SVC-02`) y `RNF-REN-02` (`≤ 1 s` p95 normal).

- **Por qué el resto del flujo es asíncrono (`A`):**
  1. **Desacoplamiento entre capacidades y hardware (criterios A1–A3):** la preparación física con AMR (`INT-21`, `INT-24`), la coordinación con transportistas externos (`INT-27`) y el registro en garita (`INT-28`) ocurren en escalas de minutos u horas y dependen de sistemas externos o hardware.
  2. **Consistencia eventual del lado conservador (criterio A4):** tanto la salida tras despacho (`INT-30`, `CE-04 ≤ 60 s`) como la liberación por cancelación (`INT-31`, `CE-05 ≤ 60 s`) operan sobre stock ya reservado, por lo que un retraso de segundos **nunca provoca sobreventa**.
  3. **Coreografía con compensación:** en lugar de transacciones distribuidas, el flujo se coordina por eventos durables con Outbox (`PRI-06`) y compensación explícita ante cancelación (`EVT-28` → `EVT-10`).

---

## 6. Diagramas asociados

- **Diagrama de actividad:** `diagramas/actividad/src/ACT-FC-reserva-despacho.puml` (exportación en `diagramas/actividad/export/ACT-FC-reserva-despacho.svg`).
- **Diagrama de secuencia:** `diagramas/secuencia/src/SEC-FC-reserva-despacho.puml` (exportación en `diagramas/secuencia/export/SEC-FC-reserva-despacho.svg`).
