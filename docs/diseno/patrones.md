# HU-30 · Informe de patrones de Smart Inventory

| Campo | Valor |
|---|---|
| Versión del informe | 0.1.1 |
| Fecha | 2026-11-27 |
| Estado | Borrador para revisión |
| Responsable | P7 · `luisdavidborjap-web` |
| Revisor | P1 · `elorgus` |
| Base de diseño | [Contrato de arquitectura v1.0.0](../ARQUITECTURA_BASE.md) y [backlog de HU-30](../BACKLOG.md#hu-30--informe-de-patrones-y-adr-005) |

El informe justifica seis decisiones de diseño. Cada apartado identifica el problema, la ubicación, el beneficio, el costo, el límite y el diagrama que permite revisar el patrón.

## 1. Strategy

**Problema.** Una política de reposición puede priorizar el mínimo y máximo, el punto de reorden o el pronóstico. La cantidad debe ajustarse a la política del SKU sin reescribir la evaluación de riesgo ni la gestión de sugerencias.

**Ubicación.** En **SVC-03 Reposición y Compras**, `EvaluadorRiesgoAgotamiento` evalúa el saldo frente a la política. `EstrategiaReposicion` encapsula el cálculo mediante `ReposicionMinMax`, `ReposicionPuntoReorden` y `ReposicionPorPronostico`. Cuando la política necesita un pronóstico, consume la copia local alimentada por **EVT-37 PronosticoDemandaActualizado**. Si ese dato falta, se usa `ReposicionMinMax`.

**Beneficio.** Agregar o sustituir un cálculo deja estable el flujo que genera y presenta la sugerencia. Esto desarrolla RNF-MAN-01 y HN-23.

**Costo y límite.** Cada estrategia necesita entradas y reglas de selección coherentes con la política. Un cálculo devuelve una recomendación. La aprobación sigue a cargo del comprador y se valida en SVC-03.

**Evidencia.** El [C4N3-SVC03-reposicion (HU-15)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/C4N3-SVC03-reposicion.svg) muestra la separación entre evaluación, estrategia y gestión de sugerencias, y la copia local del pronóstico. La secuencia [SEC-FB-reposicion (HU-17)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/SEC-FB-reposicion.svg) muestra cuándo se selecciona la estrategia y se calcula la sugerencia.

## 2. Puertos y adaptadores

**Problema.** La forma de enviar una orden de compra puede variar si cambia la integración con el ERP.

**Ubicación.** En SVC-03, la gestión de órdenes usa `PuertoErp`. `AdaptadorErp` realiza ese puerto y envía la orden a **EXT-01 ERP corporativo** mediante **INT-14 Enviar OC al ERP / proveedor (A)**. La emisión de la orden y su envío son pasos distintos.

**Beneficio.** La lógica de compra conserva su dependencia del puerto mientras cambia la integración externa. El límite respeta PRI-08.

**Costo y límite.** El puerto y el adaptador requieren mantener sus responsabilidades y significados alineados. Si el ERP está caído, la orden puede quedar *Emitida* pendiente de envío, como prevé FAL-05.

**Evidencia.** El [C4N3-SVC03-reposicion (HU-15)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/C4N3-SVC03-reposicion.svg) distingue el uso de `PuertoErp`, su implementación por `AdaptadorErp` y la interacción INT-14 con EXT-01.

## 3. Outbox e idempotencia

**Problema.** Si la recepción se registra y el aviso se publica en operaciones separadas, un fallo intermedio puede dejar a los consumidores sin conocer el cambio. Reentregar un aviso también puede duplicar efectos.

**Ubicación.** **SVC-02 Inventario** registra el lote si corresponde, el movimiento, el saldo y los eventos pendientes en **TX-01 Recepción**. `PublicadorEventos` y `PublicadorOutbox` permiten publicar **EVT-04 RecepcionRegistrada** y **EVT-05 SaldoActualizado** mediante **INT-07 Publicar EVT-04 y EVT-05 vía Outbox (A)** hacia **TRX-03 Broker de eventos**. Los receptores aplican `ConsumidorIdempotente`.

**Beneficio.** Una recepción confirmada conserva el aviso pendiente ante una caída del broker. La entrega repetida puede tratarse sin repetir efectos. Responde a PRI-06, FAL-01 y RNF-DIS-01.

**Costo y límite.** Se mantienen eventos pendientes y se puede acumular retraso durante una caída. La transacción protege los datos del módulo y el registro de eventos. La propagación a otros módulos permanece asíncrona.

**Evidencia.** La secuencia [SEC-FA-recepcion (HU-16)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/SEC-FA-recepcion.svg) muestra que TX-01 guarda el cambio y sus eventos, y que el Outbox los publica después mediante INT-07. El [ADR-004 (HU-20)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/adr/ADR-004-sincrono-y-eventos-con-outbox.md) documenta la publicación posterior, la entrega al menos una vez y el consumo idempotente.

## 4. CQRS y proyección de lectura

**Problema.** Las consultas del dashboard reúnen datos de varias capacidades, mientras que operaciones como aprobar una sugerencia necesitan el estado vigente del módulo dueño.

**Ubicación.** **TRX-04 Proyección de lectura** usa `ActualizadorVista` para preparar las vistas a partir de eventos del broker. **TRX-02 Capa de Aplicación** obtiene el snapshot y las actualizaciones mediante **INT-03 (S + A)**. Las acciones se validan en el dueño. Por ejemplo, **INT-12 Aprobar o rechazar sugerencia (S)** llega a SVC-03 desde TRX-02.

**Beneficio.** La carga de consulta se separa de las escrituras del negocio. El usuario ve la antigüedad del dato conforme a RNF-CON-02. La proyección se puede reconstruir al reprocesar eventos.

**Costo y límite.** Las vistas pueden retrasarse y hay que observar su frescura. Una vista puede mostrar una sugerencia que ya cambió de estado. SVC-03 confirma su vigencia al procesar la decisión del usuario.

**Evidencia.** El [C4N2-contenedores (HU-12)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/C4N2-contenedores.svg) sitúa TRX-04 en UD-5 y distingue INT-03 (S + A), para consultar la proyección, de INT-12 (S), que lleva la aprobación a SVC-03. La secuencia [SEC-FA-recepcion (HU-16)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/SEC-FA-recepcion.svg) muestra INT-08 y la actualización asíncrona de VL-01, VL-04 y VL-06. La decisión se desarrolla en [ADR-005, Propuesto](../adr/ADR-005-proyeccion-de-lectura-cqrs.md).

## 5. State

**Problema.** Una sugerencia puede quedar reemplazada entre su visualización y la decisión del comprador. Aprobarla después ocasionaría una orden basada en una revisión que ya perdió vigencia.

**Ubicación.** SVC-03 usa `EstadoSugerencia` para distinguir *Generada*, *Aprobada*, *Rechazada* y *Reemplazada*. La aprobación válida crea la orden *Emitida*, representada por `EstadoOrdenCompra`, en **TX-04 Aprobación de sugerencia**. La operación parte de INT-12 y se alinea con HN-11.

**Beneficio.** Las acciones permitidas se determinan por el estado vigente. Una sugerencia reemplazada se rechaza al intentar aprobarla.

**Costo y límite.** Cada transición y su disparador necesitan una revisión explícita. El estado de la sugerencia protege su propio ciclo. El envío y la recepción de la orden siguen el ciclo de la orden de compra.

**Evidencia.** El [C4N3-SVC03-reposicion (HU-15)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/C4N3-SVC03-reposicion.svg) identifica `EstadoSugerencia` y `EstadoOrdenCompra`. La secuencia [SEC-FB-reposicion (HU-17)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/SEC-FB-reposicion.svg) muestra que se rechaza la aprobación de una sugerencia reemplazada y que una aprobación válida deja la sugerencia *Aprobada* y la orden *Emitida* dentro de TX-04.

## 6. Circuit Breaker y reintentos

**Problema.** Si Inventario deja de responder, insistir con llamadas síncronas puede acumular esperas. Repetir la reserva también puede producir una segunda acción si el primer intento sí se ejecutó.

**Ubicación.** **SVC-04 Pedidos y Fulfillment** llama a SVC-02 mediante **INT-19 Solicitar reserva FEFO (S)**. `PuertoReservasInventario` y `AdaptadorReservasInventario` aíslan la llamada a `GestionReservas`. El adaptador limita el tiempo de espera, aplica un Circuit Breaker y reintenta operaciones idempotentes. El contrato usa el identificador del pedido para la idempotencia.

**Beneficio.** Ante FAL-02, el pedido conserva el estado *PendienteReserva* y puede reintentarse. La protección reduce el impacto de la indisponibilidad en el resto del flujo.

**Costo y límite.** Se debe recuperar la operación cuando Inventario vuelva a responder. Durante el fallo, el pedido permanece pendiente. La existencia disponible y el resultado de la reserva se confirman en SVC-02.

**Evidencia.** La secuencia [SEC-FC-reserva-despacho (HU-18)](https://github.com/elorgus/Deber-2-Sistemas/blob/main/diagramas/secuencia/export/SEC-FC-reserva-despacho.svg) muestra la reserva síncrona INT-19. En la rama de FAL-02, el tiempo límite o la apertura del Circuit Breaker deja el pedido en *PendienteReserva* y permite el reintento idempotente.

## Validaciones pendientes del Sprint 3

- **HU-25.** Incorporar `EST-sugerencia` y `EST-orden-compra` cuando se publiquen. La evidencia de State ya está en el C4 de Reposición y en la secuencia del flujo B.
- **HU-24 y HU-29.** Contrastar INT-19 y FAL-02 con la matriz de interacciones y el análisis de disponibilidad cuando se integren.
- **HU-27 y HU-32.** Contrastar los objetivos de frescura con los requisitos refinados y comprobar cómo se muestran los tres estados de frescura en el dashboard v2.
- **ADR-005.** Registrar la decisión de aceptación después de la revisión de P1.
