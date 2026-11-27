# HU-25 · Diagramas de estado: Lote, Reserva, Sugerencia y Orden de compra

| Campo | Valor |
|---|---|
| Documento | `equipo/P4-joshdavr/HU-25-diagramas-estado.md` |
| Versión | **1.0.0** |
| Fecha | 2026-11-27 |
| Autor | P4 (`joshdavr`) · Diseño de Catálogo e Inventario |
| Revisor | P6 (`CamGranda`) |
| Historia | HU-25 · issue #69 |
| Historias de negocio | HN-07, HN-11, HN-12 |
| Rúbrica | Flujos e interacciones arquitectónicas |
| Contrato de referencia | `docs/ARQUITECTURA_BASE.md` v1.0.0 (§3.2, §3.3, §6.2, §7.2, §7.3, §9, §10, §11.3, §11.4) |

## Entregables

| ID | Entidad | Dueño | Fuente | Exportación |
|---|---|---|---|---|
| EST-lote | Lote | SVC-02 | `diagramas/estados/src/EST-lote.puml` | `diagramas/estados/export/EST-lote.svg` |
| EST-reserva | Reserva | SVC-02 | `diagramas/estados/src/EST-reserva.puml` | `diagramas/estados/export/EST-reserva.svg` |
| EST-sugerencia | Sugerencia de reposición | SVC-03 | `diagramas/estados/src/EST-sugerencia.puml` | `diagramas/estados/export/EST-sugerencia.svg` |
| EST-orden-compra | Orden de compra | SVC-03 | `diagramas/estados/src/EST-orden-compra.puml` | `diagramas/estados/export/EST-orden-compra.svg` |

Cada diagrama lleva la leyenda de `BACKLOG.md` §7.3: `ID · v1.0.0 · 2026-11-27 · joshdavr (P4) · HU-25`.

## Cómo se cumplen los criterios de aceptación

- **CA1 (estados y transiciones idénticos a §9).** Los estados, los estados finales y las transiciones son los de `ARQUITECTURA_BASE.md` §9, sin estados nuevos. La única transición que no está en §9 (Enviada → Cerrada) aparece con línea punteada, marcada «PROPUESTA» y registrada como Q-2; no se usa como oficial.
- **CA2 (cada transición indica qué la provoca).** Cada flecha indica el comando (con su INT y su ROL u OPS) o el evento consumido que la dispara, y el evento que se publica. Cuando el contrato no define el disparador, el diagrama lo dice y remite a la pregunta abierta, en lugar de inventarlo.
- **CA3 (transiciones inválidas).** Cada diagrama tiene una nota «Transiciones inválidas» y las tablas de abajo las detallan. Incluye el ejemplo del criterio: aprobar una sugerencia *Reemplazada* (HN-11).

---

## 1. Lote (SVC-02) · EST-lote

**Estados:** Disponible, EnCuarentena, Vencido (final).

| Desde | Hacia | Qué la provoca | Se publica | Fuente en el contrato |
|---|---|---|---|---|
| (inicio) | Disponible | Recepción de un lote nuevo (INT-06, S) en TX-01 | EVT-04 RecepcionRegistrada, EVT-05 SaldoActualizado | §7.3 TX-01, §6.2 |
| Disponible | EnCuarentena | OPS-02 Cambiar estado de lote (cuarentena), ejecutada por ROL-02 | Se refleja en EVT-05 | §9, §11.4 |
| EnCuarentena | Disponible | OPS-02 Cambiar estado de lote (liberación), ejecutada por ROL-02 | Se refleja en EVT-05 | §9, §11.4 |
| Disponible | Vencido | Proceso periódico de vencimientos de SVC-02 | EVT-13 LoteVencido | §9, §3.2 |

EVT-12 LoteProximoAVencer es una alerta (HN-07): no cambia el estado del lote.

**Transiciones inválidas**

| Intento | Por qué es inválido |
|---|---|
| Vencido → Disponible o EnCuarentena | Vencido es final (§9). |
| EnCuarentena → Vencido | §9 solo define Disponible → Vencido (ver Q-1). |
| Asignar a una reserva un lote EnCuarentena o Vencido | No cuentan como disponibles (§3.2); PoliticaFEFO solo asigna lotes Disponible. |
| Cambiar el estado sin ROL-02 | OPS-02 es una operación sensible reservada a ROL-02 (§11.4). |

---

## 2. Reserva (SVC-02) · EST-reserva

**Estados:** Activa, Consumida (final), Liberada (final).

| Desde | Hacia | Qué la provoca | Se publica | Fuente en el contrato |
|---|---|---|---|---|
| (inicio) | Activa | Solicitar reserva FEFO (INT-19, S) desde SVC-04 vía `GestionReservas`, con disponible suficiente, en TX-02 | EVT-08 ReservaConfirmada, EVT-05 | §9, §7.2, §7.3 |
| Activa | Consumida | Consumo de EVT-27 PedidoDespachado (INT-30, A): registrar salida en TX-03 | EVT-11 SalidaRegistrada, EVT-05 | §9, §3.2, §7.2 |
| Activa | Liberada | Consumo de EVT-28 PedidoCancelado (INT-31, A) o expiración de la reserva | EVT-10 ReservaLiberada | §9, §3.2, §6.2 |

Si el disponible no alcanza para todo el pedido, la reserva no se crea (todo o nada) y se publica EVT-09 ReservaRechazada.

**Transiciones inválidas**

| Intento | Por qué es inválido |
|---|---|
| Consumida → Liberada o Activa | Consumida es final (§9). Un pedido ya no se cancela desde EnDespacho (§3.4). |
| Liberada → Consumida o Activa | Liberada es final: no se registra salida contra una reserva liberada. |
| Crear una reserva parcial | La reserva es todo o nada en el MVP (§3.4). |
| Repetir el cambio por un EVT-27 o EVT-28 duplicado | La entrega es al menos una vez; `ConsumidorIdempotente` descarta el duplicado (§6.1). |

---

## 3. Sugerencia de reposición (SVC-03) · EST-sugerencia

**Estados:** Generada, Aprobada (final), Rechazada (final), Reemplazada (final). Patrón State: `EstadoSugerencia` (§10).

| Desde | Hacia | Qué la provoca | Se publica | Fuente en el contrato |
|---|---|---|---|---|
| (inicio) | Generada | Evaluación de la política tras EVT-05 (INT-10, A) con `EstrategiaReposicion` | EVT-16 SugerenciaReposicionGenerada | §9, §3.3, §7.2 |
| Generada | Aprobada | Aprobar sugerencia (INT-12, S), ROL-03, OPS-03; se valida contra el estado vigente; TX-04 crea la OC Emitida | EVT-17 SugerenciaReposicionAprobada (y EVT-19 de la OC) | §9, §3.3, §7.3, HN-11 |
| Generada | Rechazada | Rechazar sugerencia (INT-12, S), ROL-03 | EVT-18 SugerenciaReposicionRechazada | §9, §6.2 |
| Generada | Reemplazada | Nueva EVT-16 para el mismo SKU (una sola sugerencia vigente por SKU) | La nueva sugerencia publica EVT-16 | §9, §3.3 |

**Transiciones inválidas**

| Intento | Por qué es inválido |
|---|---|
| Aprobar o rechazar una sugerencia Reemplazada | Reemplazada es final; la acción se rechaza con un mensaje claro (HN-11). |
| Aprobar una Rechazada o rechazar una Aprobada | Ambos son estados finales (§9). |
| Aprobar dos veces la misma sugerencia | Aprobada es final; no se emite una segunda OC. |
| Reemplazar una sugerencia Aprobada o Rechazada | Solo la sugerencia vigente (Generada) puede ser reemplazada (§3.3). |
| Aprobar o rechazar sin ROL-03 | §11.4 (OPS-03). |

---

## 4. Orden de compra (SVC-03) · EST-orden-compra

**Estados:** Emitida, Enviada, RecibidaParcial, Cerrada (final), Cancelada (final). Patrón State: `EstadoOrdenCompra` (§10).

| Desde | Hacia | Qué la provoca | Se publica | Fuente en el contrato |
|---|---|---|---|---|
| (inicio) | Emitida | Aprobación de la sugerencia en TX-04 | EVT-19 OrdenCompraEmitida | §9, §7.3 |
| Emitida | Enviada | Envío al ERP con `PuertoErp` → `AdaptadorErp` (INT-14, A con reintentos) | — | §9, §3.3, §7.2 |
| Enviada | RecibidaParcial | EVT-04 RecepcionRegistrada con referencia a la OC (INT-09, A) y cantidad menor a la de la OC | — | §7.2 INT-09, HN-12 |
| RecibidaParcial | Cerrada | EVT-04 que completa la cantidad (INT-09, A) | EVT-20 OrdenCompraCerrada | §9, §7.2, HN-12 |
| Enviada | Cerrada | «PROPUESTA»: recepción completa en una sola entrega | EVT-20 | HN-12 (ver Q-2) |
| Emitida o Enviada | Cancelada | Cancelar OC: el contrato no define disparador ni evento | — | §9 (ver Q-3) |

Si EXT-01 no está disponible, la OC permanece Emitida con reintentos idempotentes (FAL-05). Una nueva recepción parcial que no completa la cantidad deja la OC en RecibidaParcial.

**Transiciones inválidas**

| Intento | Por qué es inválido |
|---|---|
| Cerrada o Cancelada → cualquier estado | Son estados finales (§9). |
| Registrar una recepción contra una OC Cerrada o Cancelada | La OC ya no espera mercadería. |
| Cancelar una OC RecibidaParcial | §9 solo permite cancelar desde Emitida o Enviada. |
| Emitida → RecibidaParcial sin pasar por Enviada | §9 no define esa transición. |
| Emitir una segunda OC para la misma sugerencia aprobada | La OC nace una sola vez en TX-04. |

---

## 5. Preguntas abiertas (no se inventan; se resuelven vía §0.3)

| ID | Pregunta | Afecta | Propuesta |
|---|---|---|---|
| Q-1 | ¿Qué pasa con un lote EnCuarentena que llega a su fecha de vencimiento? §9 solo define Disponible → Vencido. | EST-lote | Decidir con P1 si se agrega EnCuarentena → Vencido (EVT-13) al contrato. |
| Q-2 | HN-12 dice que una OC de 200 que recibe 200 pasa a Cerrada, pero §9 solo muestra Enviada → RecibidaParcial → Cerrada. | EST-orden-compra | Agregar Enviada → Cerrada (EVT-20) a §9. Se dibuja punteada como «PROPUESTA». Consultar a P5 (diseñador de SVC-03) y P1. |
| Q-3 | §9 permite cancelar una OC desde Emitida o Enviada, pero no dice quién la cancela ni qué evento se publica. Sin evento, la réplica de OC de SVC-02 (EVT-19, EVT-20) y la de SVC-05 (EVT-19) no se enteran de la cancelación, y SVC-02 podría aceptar una recepción contra una OC cancelada. | EST-orden-compra, INT-06 | Definir el comando (ROL-03 gestiona las OC según §11.4) y un evento de cancelación, con ADR Propuesto. |
| Q-4 | EVT-10 cubre la liberación "por expiración", pero el contrato no define qué proceso detecta la expiración de una reserva ni su plazo. | EST-reserva | Definir el proceso en SVC-02 (como el de vencimientos) o quitar la expiración. |
| Q-5 | §10 nombra `EstadoSugerencia` y `EstadoOrdenCompra`, pero no nombra las clases State de Lote y Reserva. | EST-lote, EST-reserva | Que P7 (patrones) decida si se agregan a §10 para el informe de patrones (HU-30). |

---

## Historial de cambios

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-27 | P4 | Diagramas de estado de Lote, Reserva, Sugerencia y Orden de compra, transiciones inválidas y preguntas abiertas | HU-25 |
