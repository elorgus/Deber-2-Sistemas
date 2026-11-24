# Matriz detallada de propiedad de datos

| Campo | Valor |
|---|---|
| Documento | `docs/arquitectura/propiedad-de-datos.md` |
| Versión | 0.1.0 |
| Fecha | 2026-11-24 |
| Autor | P4 (`joshdavr`) |
| Revisor | P1 (`elorgus`) |
| HU | HU-13 · issue #44 |
| Base | `docs/ARQUITECTURA_BASE.md` v1.0.0, §3 (capacidades), §5 (propiedad de datos), §6 (eventos) y §7 (interacciones) |
| Estado | Borrador para revisión |

> **Regla de lectura.** Cada dato tiene **un solo dueño**, que es el único que lo crea, lo modifica y publica sus eventos. Cualquier otro componente lo obtiene por uno de tres mecanismos: **réplica local** alimentada por eventos del dueño, **evento** al que reacciona, o **consulta síncrona** al dueño. Nadie lee ni escribe el esquema de otro módulo (PRI-01, PRI-02).

---

## 1. Mecanismos de acceso

| Código | Mecanismo | Cuándo se usa | Principio |
|---|---|---|---|
| **D** | Dueño: crea, modifica y publica | Solo un componente por dato | PRI-01 |
| **R** | Réplica local de solo lectura alimentada por eventos | Datos de referencia que se leen a menudo | PRI-04, ADR-009 |
| **E** | Reacciona a eventos o los consolida, sin mantener una copia del dato | Hechos de negocio que disparan una acción o una vista | PRI-05 |
| **C** | Copia de solo lectura recibida en la respuesta de una interacción síncrona | Resultado que el solicitante necesita conservar | PRI-03 |
| **S** | Consulta síncrona al dueño | Se necesita el dato exacto en ese momento (§7.1, S3) | PRI-02 |

---

## 2. Matriz detallada (CA1)

Una fila por entidad de la §5. "Lectores" incluye a todos los que la §5 declara en "Quién más lo usa".

| # | Dato | Dueño | Lector | Mecanismo | Evento o interacción de origen |
|---|---|---|---|---|---|
| 1 | Producto / SKU, categoría | SVC-01 | SVC-02, SVC-03, SVC-04 | R | EVT-01 ProductoRegistrado, EVT-02 ProductoActualizado · INT-04 (A) |
| | | | TRX-04, SVC-06 | E | EVT-01, EVT-02 · INT-05 (A) para SVC-06 |
| 2 | Bodega, zona, ubicación | SVC-01 | SVC-02, SVC-05 | R | EVT-03 UbicacionActualizada · INT-04 (A) |
| | | | TRX-04 (VL-04), SVC-06 | E | EVT-03 |
| 3 | Saldo (físico, reservado, disponible) | SVC-02 | SVC-03 | R (disponible agregado por SKU) | EVT-05 SaldoActualizado · INT-10 (A) |
| | | | TRX-04 (VL-01, VL-04), SVC-06 | E | EVT-05 · INT-07, INT-20 (A) |
| | | | TRX-02 (detalle exacto por lote) | S | INT-15 Consultar detalle exacto de existencias y lotes |
| 4 | Lote y fecha de vencimiento | SVC-02 | SVC-04 | C (copia de la asignación) | Respuesta de INT-19 (S) |
| | | | TRX-04 (VL-01, VL-02), SVC-06 | E | EVT-04, EVT-08, EVT-12 LoteProximoAVencer, EVT-13 LoteVencido |
| 5 | Movimiento (recepción, transferencia, ajuste, salida) | SVC-02 | TRX-04 (VL-06), SVC-06 | E | EVT-04 RecepcionRegistrada, EVT-06, EVT-07, EVT-11 |
| 6 | Reserva de stock | SVC-02 | SVC-04 | C (solo identificador y copia de asignación) | Respuesta de INT-19 (S) |
| | | | TRX-04 | E | EVT-08 ReservaConfirmada, EVT-09 ReservaRechazada, EVT-10 ReservaLiberada |
| 7 | Política de reposición (mínimos) | SVC-03 | TRX-04 (indirecto) | E | EVT-14, EVT-15 (la vista muestra la alerta, no la política) |
| 8 | Alerta de riesgo de agotamiento | SVC-03 | TRX-04 (VL-01, VL-02), SVC-06 | E | EVT-14 RiesgoAgotamientoDetectado, EVT-15 RiesgoAgotamientoResuelto |
| 9 | Sugerencia de reposición | SVC-03 | TRX-04 (VL-02), SVC-06 | E | EVT-16, EVT-17, EVT-18 |
| | | | TRX-02 (detalle y aprobación) | S | Consulta al dueño; aprobar o rechazar por INT-12 (S) |
| 10 | Proveedor | SVC-03 | — | S | Solo se consulta en SVC-03 |
| 11 | Orden de compra (OC) | SVC-03 | SVC-02 (recepciones esperadas) | R | EVT-19 OrdenCompraEmitida, EVT-20 OrdenCompraCerrada |
| | | | SVC-05 (llegadas esperadas) | R | EVT-19 |
| | | | TRX-04, SVC-06 | E | EVT-19, EVT-20 |
| 12 | Pedido y su estado | SVC-04 | TRX-04 (VL-01, VL-03), SVC-06 | E | EVT-21 a EVT-28 |
| 13 | Tarea de preparación | SVC-04 | SVC-05 | E | EVT-24 PreparacionSolicitada · INT-21 (A) |
| 14 | AMR, misión, ruta | SVC-05 | SVC-04 (estado de la misión) | E | EVT-29 MisionAmrAsignada, EVT-30 MisionAmrCompletada · INT-24 (A) |
| | | | TRX-04 (VL-04), SVC-06 | E | EVT-29, EVT-30, EVT-31 |
| 15 | Telemetría (posición, batería) | SVC-05 | TRX-04 (VL-04) | E (señal muestreada) | EVT-32 PosicionFlotaActualizada · la telemetría cruda no sale de SVC-05 |
| 16 | Visita de vehículo, muelle, patio | SVC-05 | SVC-04 (muelle asignado) | R | EVT-34 MuelleAsignado · INT-29 (A) |
| | | | SVC-04 (despacho) | E | EVT-35 VehiculoDespachado · INT-29 (A) |
| | | | TRX-04 (VL-05), SVC-06 | E | EVT-33 a EVT-36 |
| 17 | Pronóstico de demanda | SVC-06 | SVC-03 | R (copia local) | EVT-37 PronosticoDemandaActualizado · INT-16 (A) |
| 18 | Indicadores históricos | SVC-06 | TRX-04 (VL-07) | E | EVT-38 IndicadoresHistoricosActualizados |
| | | | TRX-02 | S | INT-34 Consultar indicadores históricos |
| 19 | Vistas del dashboard | TRX-04 (**derivado, no es fuente de verdad**) | TRX-02 | S + push | INT-03 |
| 20 | Usuarios y roles | EXT-04 | TRX-02 y cada módulo dueño | S | INT-02 |
| 21 | Auditoría de operaciones sensibles | El módulo dueño de cada operación | SVC-06 | E | Evento correspondiente de la operación |

---

## 3. Vista cruzada: dato × componente

`D` = dueño · `R` = réplica · `E` = evento · `C` = copia de respuesta · `S` = consulta síncrona · vacío = no lo usa.

| Dato | SVC-01 | SVC-02 | SVC-03 | SVC-04 | SVC-05 | SVC-06 | TRX-02 | TRX-04 |
|---|---|---|---|---|---|---|---|---|
| Producto / SKU | **D** | R | R | R | | E | | E |
| Ubicación | **D** | R | | | R | E | | E |
| Saldo | | **D** | R | | | E | S | E |
| Lote y vencimiento | | **D** | | C | | E | | E |
| Movimiento | | **D** | | | | E | | E |
| Reserva | | **D** | | C | | | | E |
| Política de reposición | | | **D** | | | | | E |
| Alerta de riesgo | | | **D** | | | E | | E |
| Sugerencia | | | **D** | | | E | S | E |
| Proveedor | | | **D** | | | | | |
| Orden de compra | | R | **D** | | R | E | | E |
| Pedido | | | | **D** | | E | | E |
| Tarea de preparación | | | | **D** | E | | | |
| AMR, misión, ruta | | | | E | **D** | E | | E |
| Telemetría | | | | | **D** | | | E |
| Vehículo, muelle, patio | | | | R / E | **D** | E | | E |
| Pronóstico de demanda | | | R | | | **D** | | |
| Indicadores históricos | | | | | | **D** | S | E |

**Control:** cada fila tiene exactamente una `D`.

---

## 4. Réplicas locales y su evento de origen (CA2)

Todas las réplicas declaradas en la §3 del contrato. Son de solo lectura, las actualiza el `ConsumidorIdempotente` de cada módulo y se pueden reconstruir reprocesando eventos (ADR-009).

| Módulo | Réplica local | Dueño del dato | Evento de origen | Interacción | Para qué la usa |
|---|---|---|---|---|---|
| SVC-01 | — (no tiene réplicas) | — | — | — | — |
| SVC-02 | Productos (`ReplicaCatalogo`) | SVC-01 | EVT-01, EVT-02 | INT-04 | Exigir lote y vencimiento en la recepción; vida útil mínima para FEFO |
| SVC-02 | Ubicaciones (`ReplicaCatalogo`) | SVC-01 | EVT-03 | INT-04 | Rechazar recepciones en ubicaciones bloqueadas |
| SVC-02 | OC emitidas pendientes de recepción | SVC-03 | EVT-19, EVT-20 | — (ver O3) | Validar la recepción contra una OC |
| SVC-03 | Catálogo (`ReplicaCatalogo`) | SVC-01 | EVT-01, EVT-02 | INT-04 | Políticas por SKU activo |
| SVC-03 | Disponible agregado por SKU | SVC-02 | EVT-05 | INT-10 | Evaluar riesgo de agotamiento (ADR-006) |
| SVC-03 | Demanda no atendida | SVC-02 | EVT-09 | — (ver O3) | Calcular la sugerencia |
| SVC-03 | Pronóstico de demanda | SVC-06 | EVT-37 | INT-16 | `ReposicionPorPronostico` |
| SVC-04 | Catálogo (`ReplicaCatalogo`) | SVC-01 | EVT-01, EVT-02 | INT-04 | Validar líneas del pedido |
| SVC-04 | Muelle asignado | SVC-05 | EVT-34 | INT-29 | Preparar el despacho |
| SVC-05 | Ubicaciones y layout (`ReplicaCatalogo`) | SVC-01 | EVT-03 | INT-04 | Calcular rutas de AMR |
| SVC-05 | OC emitidas | SVC-03 | EVT-19 | — (ver O3) | Anticipar llegadas de proveedores |
| SVC-06 | No aplica: todo su contenido es derivado de eventos | — | Todos los de tipo *Negocio* | INT-05 | Histórico e indicadores |

**Copia que no es réplica.** SVC-04 guarda el identificador de la reserva y una copia de solo lectura de la asignación por lote, recibidas en la **respuesta** de INT-19, no por un evento. El dueño sigue siendo SVC-02 (§3.4).

---

## 5. Verificación (CA3)

| Comprobación | Resultado |
|---|---|
| Ningún dato tiene dos dueños | **Cumple.** Las 21 filas de la §2 y las 18 de la §3 tienen un único dueño. |
| Ninguna réplica tiene como origen un evento de alguien que no es el dueño | **Cumple.** Todos los eventos de origen de la §4 los emite el dueño del dato (§6.2). |
| Ningún lector escribe el dato | **Cumple.** Los lectores solo usan R, E, C o S; las acciones del dashboard van al dueño (PRI-07). |
| Coherencia con el C4 N2 (HU-12) | **Pendiente.** El diagrama `C4N2-contenedores` todavía no está publicado. Cuando exista se debe verificar que cada contenedor dueño tenga su propio esquema y que no haya flechas hacia esquemas ajenos. |
| Coherencia con los C4 N3 publicados | **Cumple.** `C4N3-SVC02-inventario` (HU-14) y `C4N3-SVC03-reposicion` (HU-15, PR #19) muestran los mismos datos propios y las mismas réplicas que esta matriz. |

---

## 6. Observaciones para P1 (posibles ajustes al contrato)

Ninguna cambia un dueño. Son omisiones en la columna "Quién más lo usa" de la §5 frente a lo que dicen la §3 y la §6. Si P1 las acepta, se corrigen con un PR al contrato según la §0.3.

- **O1 · "Solicitud de despacho" no tiene fila en la §5.** La §3.4 la declara dato propio de SVC-04 y SVC-05 reacciona a ella por EVT-26 DespachoSolicitado (INT-26). *Propuesta:* agregar la fila "Solicitud de despacho · SVC-04 · SVC-05, TRX-04, SVC-06 · EVT-26".
- **O2 · Lectores omitidos en la §5.**
  - *Movimiento:* SVC-03 consume EVT-04 para el avance de la OC (INT-09), pero no figura como lector.
  - *Reserva:* SVC-03 consume EVT-09 (demanda no atendida), pero no figura como lector.
  - *Pedido:* SVC-02 (EVT-27, EVT-28) y SVC-05 (EVT-28) reaccionan a eventos del pedido, pero no figuran como lectores.
  - *Propuesta:* agregarlos en "Quién más lo usa".
- **O3 · Réplicas sin interacción en la §7.2.** La réplica de OC en SVC-02 (EVT-19, EVT-20), la de OC en SVC-05 (EVT-19) y la de demanda no atendida en SVC-03 (EVT-09) no tienen un INT propio. *Propuesta:* agregarlas a la matriz de interacciones (HU-24).

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio |
|---|---|---|---|
| 0.1.0 | 2026-11-24 | P4 | Versión inicial: matriz detallada, vista cruzada, réplicas y verificación (HU-13). |
