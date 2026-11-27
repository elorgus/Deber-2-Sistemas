# ADR-003 · La reserva de stock y la selección FEFO pertenecen a Inventario

| Campo | Valor |
|---|---|
| Estado | Propuesto |
| Fecha | 2026-11-24 |
| Responsable · Revisor | P4 · P6 |
| HU relacionada | HU-14 |
| Elementos del contrato afectados | `SVC-02`, `SVC-04`, `INT-19`, `INT-20`, `INT-30`, `INT-31`, `TX-02`, `TX-03`, `EVT-05`, `EVT-08`, `EVT-09`, `EVT-10`, `EVT-11`, `EVT-22`, `EVT-23`, `EVT-27`, `EVT-28`, `CE-04`, `CE-05`, `RNF-CON-01`, `RNF-REN-02`, `RNF-DIS-02`, `RNF-MAN-01`, `FAL-02` |
| Diagramas relacionados | `C4N3-SVC02-inventario` |

## 1. Contexto

En el Flujo C, al registrar un pedido hay que **bloquear stock por lote** y elegir **qué lotes** se entregan, aplicando FEFO (*First Expired, First Out*). El enunciado deja abierto en qué módulo vive esa responsabilidad: en Pedidos (`SVC-04`), que inicia la operación, o en Inventario (`SVC-02`), que guarda las existencias.

La decisión depende de dos datos que, según `ARQUITECTURA_BASE.md` §5, tienen dueño:
- el **saldo** (físico, reservado y disponible) y el **lote con su fecha de vencimiento** son de `SVC-02`;
- el **pedido y su estado** son de `SVC-04`.

Principios del contrato que influyen:
- **PRI-01:** un dueño por dato.
- **PRI-02:** ningún módulo lee ni escribe el esquema de otro.
- **PRI-03:** entre módulos se comparte el identificador, no el objeto.
- **PRI-09:** el mismo contrato debe servir en el monolito y al extraer módulos; `SVC-02` es el primer candidato a extracción (§2.4).

Requisitos del negocio y de calidad:
- **HN-13:** con L1 (vence el 10-dic) y L2 (vence el 20-dic) disponibles, se asigna primero L1; los lotes *Vencido* o *EnCuarentena* nunca se asignan.
- **HN-14:** si el disponible no alcanza, no se reserva nada en forma parcial (MVP) y el pedido queda *EnEspera*.
- **RNF-CON-01:** el disponible nunca es negativo (0 reservas que lo superen).
- **RNF-REN-02:** una reserva se resuelve en ≤ 1 s p95 con carga normal y ≤ 2 s p95 en pico.
- **RNF-DIS-02 / FAL-02:** si Inventario no responde, ningún pedido se pierde.
- **RNF-MAN-01:** agregar una política de selección de lotes no debe cambiar el flujo.

## 2. Opciones consideradas (mínimo 2)

### Opción A · Reserva y FEFO en SVC-04 Pedidos y Fulfillment
- **Descripción:** Pedidos consulta los lotes disponibles, aplica FEFO y registra la reserva en su propio esquema. Inventario solo se entera después, por un evento, y descuenta.
- **Ventajas:**
  - El pedido y su reserva quedan en el mismo módulo, así que el estado del pedido y la reserva cambian juntos.
  - Pedidos no depende de una llamada síncrona a Inventario para avanzar.
- **Desventajas:**
  - La **reserva tendría dos dueños de hecho**: `SVC-04` la decide y `SVC-02` tiene que reflejarla en el saldo reservado. Eso contradice PRI-01 y la §5 (la reserva es de `SVC-02`).
  - Para aplicar FEFO, `SVC-04` necesitaría leer saldos y lotes con su vencimiento, que son de `SVC-02`. Tendría que leer su esquema (viola PRI-02) o mantener una réplica, que siempre llega con retraso.
  - **No puede garantizar RNF-CON-01.** Con una réplica atrasada, dos pedidos simultáneos pueden reservar el mismo stock y el disponible real queda negativo (sobreventa).
  - Cada cambio en las reglas de lote (vida útil mínima, cuarentena, FIFO para productos sin vencimiento) obligaría a tocar Pedidos y también Inventario.

### Opción B · Reserva y FEFO en SVC-02 Inventario, solicitada de forma síncrona
- **Descripción:** `SVC-04` solicita la reserva por `INT-19` (S) a través de `PuertoReservasInventario` → `AdaptadorReservasInventario`. En `SVC-02`, `GestionReservas` ordena los lotes elegibles con `PoliticaSeleccionLotes` (`PoliticaFEFO`, o `PoliticaFIFO` si el producto no tiene vencimiento), verifica el disponible y guarda reserva, saldo reservado y outbox en `TX-02`. Responde con el resultado y la asignación por lote, y publica `EVT-08` o `EVT-09` y `EVT-05` (`INT-20`). `SVC-04` guarda solo el identificador de la reserva y una copia de solo lectura de la asignación.
- **Ventajas:**
  - Un solo dueño para saldo, lote y reserva (PRI-01). Nadie más escribe el saldo reservado.
  - **RNF-CON-01 se cumple por construcción:** la verificación del disponible y la reserva ocurren en la misma transacción local (`TX-02`), sobre la fuente de verdad.
  - La regla FEFO vive junto a los datos que necesita (vencimiento, estado del lote, vida útil mínima de la réplica de catálogo).
  - Agregar otra política es agregar una estrategia (`PoliticaSeleccionLotes`, Strategy/OCP, RNF-MAN-01), sin tocar Pedidos.
  - Al extraer `SVC-02` (§2.4) solo cambia el adaptador: `INT-19` pasa de llamada interna a remota con el mismo puerto `GestionReservas` (PRI-09).
- **Desventajas:**
  - `SVC-04` depende de la disponibilidad de `SVC-02` para reservar.
  - La llamada síncrona suma latencia al registro del pedido y debe cumplir RNF-REN-02.
  - La liberación por cancelación (`EVT-28`, `INT-31`) y la salida por despacho (`EVT-27`, `INT-30`) son eventuales (`CE-05`, `CE-04`).

### Opción C · Reserva en SVC-02, pero solicitada de forma asíncrona por evento
- **Descripción:** `SVC-04` publica que se registró un pedido. `SVC-02` reacciona, reserva con FEFO y publica el resultado, y `SVC-04` actualiza el estado del pedido al recibirlo.
- **Ventajas:**
  - Mantiene un solo dueño de la reserva (como B).
  - `SVC-04` no queda bloqueado si `SVC-02` está caído.
- **Desventajas:**
  - El usuario y el pedido no conocen el resultado al momento. El pedido queda en un estado intermedio más tiempo, y la confirmación depende del broker (`FAL-01`).
  - Va contra el criterio **S2** de la §7.1: una operación que protege un invariante (disponible ≥ 0) debe confirmarse antes de seguir.
  - Agrega más estados y eventos para coordinar sin beneficio para la carga del MVP (SUP-01 a SUP-03).

## 3. Decisión

Se elige la **Opción B**. La reserva de stock y la selección de lotes con FEFO pertenecen a `SVC-02`, porque es el dueño del saldo, del lote y de la reserva, y es el único lugar donde se puede garantizar que el disponible nunca sea negativo (`TX-02`, RNF-CON-01). `SVC-04` la solicita en forma síncrona por `INT-19`, protegida con tiempo límite y Circuit Breaker, y solo guarda el identificador de la reserva y una copia de solo lectura de la asignación.

## 4. Consecuencias

**Positivas**
- Responsabilidad clara: quien es dueño del stock decide qué lote se entrega y cuánto queda disponible (PRI-01, PRI-02).
- Sin sobreventa: la verificación y la reserva son una sola transacción local (`TX-02`).
- `GestionReservas` y `PoliticaSeleccionLotes` quedan como componentes separados en el `C4N3-SVC02-inventario`, lo que sirve de evidencia de SRP, OCP y DIP para HU-31.
- La extracción futura de `SVC-02` no cambia el flujo C, solo el adaptador (PRI-09).

**Negativas o riesgos**
- **Dependencia síncrona de SVC-04 hacia SVC-02.** Se mitiga con tiempo límite y Circuit Breaker en `AdaptadorReservasInventario`. Si `SVC-02` no responde, el pedido queda en *PendienteReserva* y se reintenta con el identificador del pedido como clave de idempotencia (FAL-02, RNF-DIS-02).
- **Latencia en el registro del pedido.** Se acota con RNF-REN-02 y con un máximo de dos saltos síncronos por operación (§7.1). Si se incumple de forma sostenida, es la señal para extraer `SVC-02` (§2.4).
- **Liberación y salida eventuales** (`CE-05`, `CE-04`). El retraso queda del lado conservador: mientras no llega `EVT-28` o `EVT-27`, el stock sigue reservado y no hay sobreventa (§7.1, A4).
- **Reserva todo o nada.** Pedidos parcialmente atendibles quedan *EnEspera* (HN-14). Se acepta en el MVP; permitir reservas parciales requeriría un ADR nuevo.

## 5. Notas

- **Cómo se verifica:**
  - En el `C4N3-SVC02-inventario`, `GestionReservas` y `PoliticaSeleccionLotes` están dentro del límite de `SVC-02`, y la única entrada de `SVC-04` es `INT-19` (S).
  - Ningún diagrama muestra a `SVC-04` escribiendo saldo, lote o reserva, ni leyendo el esquema de `SVC-02` (RNF-MAN-02).
  - El diagrama de secuencia del Flujo C (HU-18) muestra `TX-02` dentro de `SVC-02` y la publicación de `EVT-08` o `EVT-09` por outbox.
- **Relacionados:** HN-13, HN-14, HN-16, HN-19 · PRI-01, PRI-02, PRI-03, PRI-09 · ADR-001, ADR-002, ADR-004 · HU-14, HU-18, HU-31.
- Un ADR *Aceptado* no se edita: si la decisión cambia, se crea un ADR nuevo que lo reemplaza (`docs/VERSIONAMIENTO.md` §6).
- Fuera de alcance: endpoints, payloads, CI/CD y pruebas.
