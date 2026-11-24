# HU-30 · Informe de patrones de Smart Inventory

| Campo | Valor |
|---|---|
| Versión del informe | 0.1.0 |
| Fecha | 2026-11-23 |
| Estado | Borrador para revisión |
| Responsable | P7 · `luisdavidborjap-web` |
| Revisor | P1 · `elorgus` |
| Base de diseño | [Contrato de arquitectura v1.0.0 propuesto](../ARQUITECTURA_BASE.md) y [backlog de HU-30](../BACKLOG.md#hu-30--informe-de-patrones-y-adr-005) |

El informe justifica seis decisiones de diseño. Cada apartado identifica el problema, la ubicación, el beneficio, el costo, el límite y el diagrama que permite revisar el patrón.

## 1. Strategy

**Problema.** Una política de reposición puede priorizar el mínimo y máximo, el punto de reorden o el pronóstico. La cantidad debe ajustarse a la política del SKU sin reescribir la evaluación de riesgo ni la gestión de sugerencias.

**Ubicación.** En **SVC-03 Reposición y Compras**, `EvaluadorRiesgoAgotamiento` evalúa el saldo frente a la política. `EstrategiaReposicion` encapsula el cálculo mediante `ReposicionMinMax`, `ReposicionPuntoReorden` y `ReposicionPorPronostico`. Cuando la política necesita un pronóstico, consume la copia local alimentada por **EVT-37 PronosticoDemandaActualizado**. Si ese dato falta, se usa `ReposicionMinMax`.

**Beneficio.** Agregar o sustituir un cálculo deja estable el flujo que genera y presenta la sugerencia. Esto desarrolla RNF-MAN-01 y HN-23.

**Costo y límite.** Cada estrategia necesita entradas y reglas de selección coherentes con la política. Un cálculo devuelve una recomendación. La aprobación sigue a cargo del comprador y se valida en SVC-03.

**Evidencia.** `C4N3-SVC03-reposicion` muestra la separación entre evaluación, estrategia y gestión de sugerencias, junto con la copia del pronóstico. `SEC-FB-reposicion` muestra la selección de estrategia antes de guardar la sugerencia. [HU-15 · PR #19, borrador](https://github.com/elorgus/Deber-2-Sistemas/pull/19/files) · [HU-17 · PR #20, borrador](https://github.com/elorgus/Deber-2-Sistemas/pull/20/files)

## 2. Puertos y adaptadores

**Problema.** La forma de enviar una orden de compra puede variar si cambia la integración con el ERP.

**Ubicación.** En SVC-03, la gestión de órdenes usa `PuertoErp`. `AdaptadorErp` realiza ese puerto y envía la orden a **EXT-01 ERP corporativo** mediante **INT-14 Enviar OC al ERP / proveedor (A)**. La emisión de la orden y su envío son pasos distintos.

**Beneficio.** La lógica de compra conserva su dependencia del puerto mientras cambia la integración externa. El límite respeta PRI-08.

**Costo y límite.** El puerto y el adaptador requieren mantener sus responsabilidades y significados alineados. Si el ERP está caído, la orden puede quedar *Emitida* pendiente de envío, como prevé FAL-05.

**Evidencia.** El diagrama de HU-15 distingue el uso de `PuertoErp`, la implementación por `AdaptadorErp` y la interacción con EXT-01. [HU-15 · PR #19, borrador](https://github.com/elorgus/Deber-2-Sistemas/pull/19/files)

## 3. Outbox e idempotencia

**Problema.** Si la recepción se registra y el aviso se publica en operaciones separadas, un fallo intermedio puede dejar a los consumidores sin conocer el cambio. Reentregar un aviso también puede duplicar efectos.

**Ubicación.** **SVC-02 Inventario** registra el lote si corresponde, el movimiento, el saldo y los eventos pendientes en **TX-01 Recepción**. `PublicadorEventos` y `PublicadorOutbox` permiten publicar **EVT-04 RecepcionRegistrada** y **EVT-05 SaldoActualizado** mediante **INT-07 Publicar EVT-04 y EVT-05 vía Outbox (A)** hacia **TRX-03 Broker de eventos**. Los receptores aplican `ConsumidorIdempotente`.

**Beneficio.** Una recepción confirmada conserva el aviso pendiente ante una caída del broker. La entrega repetida puede tratarse sin repetir efectos. Responde a PRI-06, FAL-01 y RNF-DIS-01.

**Costo y límite.** Se mantienen eventos pendientes y se puede acumular retraso durante una caída. La transacción protege los datos del módulo y el registro de eventos. La propagación a otros módulos permanece asíncrona.

**Evidencia.** `SEC-FA-recepcion` muestra TX-01 y la publicación posterior. Contrastar la explicación con el ADR-004 de Elías cuando se publique en HU-20. [HU-16 · PR #17, borrador](https://github.com/elorgus/Deber-2-Sistemas/pull/17/files)

## 4. CQRS y proyección de lectura

**Problema.** Las consultas del dashboard reúnen datos de varias capacidades, mientras que operaciones como aprobar una sugerencia necesitan el estado vigente del módulo dueño.

**Ubicación.** **TRX-04 Proyección de lectura** usa `ActualizadorVista` para preparar las vistas a partir de eventos del broker. **TRX-02 Capa de Aplicación** obtiene el snapshot y las actualizaciones mediante **INT-03 (S + A)**. Las acciones se validan en el dueño. Por ejemplo, **INT-12 Aprobar o rechazar sugerencia (S)** llega a SVC-03 desde TRX-02.

**Beneficio.** La carga de consulta se separa de las escrituras del negocio. El usuario ve la antigüedad del dato conforme a RNF-CON-02. La proyección se puede reconstruir al reprocesar eventos.

**Costo y límite.** Las vistas pueden retrasarse y hay que observar su frescura. Una vista puede mostrar una sugerencia que ya cambió de estado. SVC-03 confirma su vigencia al procesar la decisión del usuario.

**Evidencia.** `SEC-FA-recepcion` muestra **INT-08 Actualizar VL-01, VL-04 y VL-06 (A)** tras la recepción. Contrastar la representación general con HU-12 y los indicadores de frescura con HU-32. La decisión se explica en el [ADR-005 propuesto](../adr/ADR-005-proyeccion-de-lectura-cqrs.md). [HU-16 · PR #17, borrador](https://github.com/elorgus/Deber-2-Sistemas/pull/17/files)

## 5. State

**Problema.** Una sugerencia puede quedar reemplazada entre su visualización y la decisión del comprador. Aprobarla después ocasionaría una orden basada en una revisión que ya perdió vigencia.

**Ubicación.** SVC-03 usa `EstadoSugerencia` para distinguir *Generada*, *Aprobada*, *Rechazada* y *Reemplazada*. La aprobación válida crea la orden *Emitida*, representada por `EstadoOrdenCompra`, en **TX-04 Aprobación de sugerencia**. La operación parte de INT-12 y se alinea con HN-11.

**Beneficio.** Las acciones permitidas se determinan por el estado vigente. Una sugerencia reemplazada se rechaza al intentar aprobarla.

**Costo y límite.** Cada transición y su disparador necesitan una revisión explícita. El estado de la sugerencia protege su propio ciclo. El envío y la recepción de la orden siguen el ciclo de la orden de compra.

**Evidencia.** `C4N3-SVC03-reposicion` muestra ambos estados como parte del diseño de SVC-03. `SEC-FB-reposicion` muestra el rechazo de una aprobación cuando la sugerencia está reemplazada. `EST-sugerencia` y `EST-orden-compra` de HU-25 (P4) aportarán el detalle de transiciones. [HU-15 · PR #19, borrador](https://github.com/elorgus/Deber-2-Sistemas/pull/19/files) · [HU-17 · PR #20, borrador](https://github.com/elorgus/Deber-2-Sistemas/pull/20/files)

## 6. Circuit Breaker y reintentos

**Problema.** Si Inventario deja de responder, insistir con llamadas síncronas puede acumular esperas. Repetir la reserva también puede producir una segunda acción si el primer intento sí se ejecutó.

**Ubicación.** **SVC-04 Pedidos y Fulfillment** llama a SVC-02 mediante **INT-19 Solicitar reserva FEFO (S)**. `PuertoReservasInventario` y `AdaptadorReservasInventario` aíslan la llamada a `GestionReservas`. El adaptador limita el tiempo de espera, aplica un Circuit Breaker y reintenta operaciones idempotentes. El contrato usa el identificador del pedido para la idempotencia.

**Beneficio.** Ante FAL-02, el pedido conserva el estado *PendienteReserva* y puede reintentarse. La protección reduce el impacto de la indisponibilidad en el resto del flujo.

**Costo y límite.** Se debe recuperar la operación cuando Inventario vuelva a responder. Durante el fallo, el pedido permanece pendiente. La existencia disponible y el resultado de la reserva se confirman en SVC-02.

**Evidencia pendiente.** `SEC-FC-reserva-despacho` de HU-18 (P6) debe mostrar INT-19 y el caso alternativo de Inventario no disponible. Contrastar ese comportamiento con la matriz de HU-24 y con FAL-02 de HU-29 cuando se publiquen.

## Evidencia que falta confirmar antes del cierre

- **P1 · HU-06, HU-12 y HU-20.** Comparar la versión aprobada del contrato con el informe y verificar la ubicación de TRX-04 y el ADR-004. La base de este borrador es el contrato propuesto v1.0.0.
- **P5 · HU-16, HU-17 y HU-27.** Verificar las versiones aceptadas de `SEC-FA-recepcion` y `SEC-FB-reposicion` y contrastar los requisitos de calidad utilizados en el informe. HU-16 y HU-17 tienen PR en borrador.
- **P4 y P6 · HU-25 y HU-18.** Añadir enlaces a los diagramas de estados y a la secuencia de reserva cuando estén disponibles. Alinear INT-19 con HU-24 (P6) y el escenario FAL-02 con HU-29 (P1).
- **P2 · HU-32.** Comprobar que el dashboard represente las tres marcas de frescura definidas en el contrato.
- **P7 y P1 · HU-15 y ADR-005.** Actualizar los enlaces a los diagramas definitivos y registrar la aceptación de ADR-005 conforme a la revisión de HU-30.
