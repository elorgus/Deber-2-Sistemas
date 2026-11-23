# HU-06 · Revisión de arquitectura de P7

**Versión** 0.1.0 · **Fecha** 2026-11-23 · **Autor** P7 (`luisdavidborjap-web`) · **Revisor** P1 (`elorgus`)

**Resultado.** El [contrato maestro v1.0.0](../../docs/ARQUITECTURA_BASE.md#10-nombres-canónicos-de-puertos-estrategias-y-adaptadores) contiene los límites, las responsabilidades y los elementos canónicos necesarios para los cinco patrones revisados. Esta validación de P7 cubre el encaje de los patrones con las unidades del MVP.

## Unidades de despliegue del MVP

| Unidad | Encaje en la arquitectura |
|---|---|
| **UD-1 Núcleo transaccional** | Agrupa **SVC-01 Catálogo y Ubicaciones**, **SVC-02 Inventario**, **SVC-03 Reposición y Compras** y **SVC-04 Pedidos y Fulfillment**. Cada módulo conserva su esquema, sus puertos y sus transacciones locales. |
| **UD-2 Flota y Operación** | **SVC-05 Flota y Operación** aísla las integraciones de transportistas y AMR, las estrategias de asignación y los estados de su operación. |
| **UD-3 Analítica** | **SVC-06 Analítica** usa **TRX-05 Data Lakehouse** para el histórico. La ingesta por `INT-05` (ingesta histórica de eventos, A) separa esa carga de la operación transaccional. |
| **UD-4 Acceso** | **TRX-01 Dashboard** y **TRX-02 Capa de Aplicación** presentan vistas y dirigen cada comando al dueño del dato. |
| **UD-5 Proyección de lectura** | **TRX-04 Proyección de lectura** mantiene las vistas del dashboard a partir de los eventos que recibe de **TRX-03 Broker de eventos**, infraestructura compartida del MVP. |

## Validación de patrones

| Patrón | Evidencia en el contrato |
|---|---|
| **Puertos y Adaptadores** | `INT-19 Solicitar reserva FEFO` (S) conecta `PuertoReservasInventario` de `SVC-04` con `GestionReservas` de `SVC-02` mediante `AdaptadorReservasInventario`. La interacción es interna en `UD-1`. Al extraer Inventario, cambia el adaptador y se conserva el puerto. En `UD-2`, `PuertoControladorAmr` y `AdaptadorControladorAmr` aíslan la integración con **EXT-03 Controlador de flota AMR (fabricante)**. |
| **Outbox e idempotencia** | En `TX-01 Recepción`, `SVC-02` persiste el lote si es nuevo, el movimiento, el saldo y el registro de Outbox en la misma transacción. `PublicadorEventos` tiene como adaptador `PublicadorOutbox`. La publicación posterior de `EVT-04 RecepcionRegistrada` y `EVT-05 SaldoActualizado` va al broker por `INT-07` (publicación por Outbox, A). `ConsumidorIdempotente` evita efectos duplicados. Esta regla cubre los eventos de negocio durables. `EVT-32 PosicionFlotaActualizada` se mantiene como señal de telemetría de pérdida tolerable. |
| **CQRS** | `ActualizadorVista` actualiza cada vista de `TRX-04`. Por ejemplo, `INT-08` (actualización de vistas, A) alimenta **VL-01 KPIs operativos** con `EVT-05`. `TRX-02` consulta la proyección por `INT-03` (lectura inicial y actualizaciones en vivo, S + A) y dirige las acciones de negocio a los módulos dueños, que las validan con su estado vigente. La interfaz informa la frescura de la vista. |
| **State** | El dueño de cada entidad aplica las transiciones permitidas de la §9. En `UD-1`, `EstadoPedido` pertenece a `SVC-04`, `EstadoSugerencia` y `EstadoOrdenCompra` a `SVC-03`, y el ciclo `Activa → Consumida` o `Activa → Liberada` de la reserva a `SVC-02`. En `UD-2`, `SVC-05` aplica `EstadoMisionAmr` y `EstadoVisitaVehiculo`. |
| **Strategy** | `SVC-02` aplica `PoliticaSeleccionLotes` con `PoliticaFEFO` o `PoliticaFIFO`. `SVC-03` usa `EstrategiaReposicion` con `ReposicionMinMax`, `ReposicionPuntoReorden` o `ReposicionPorPronostico`. `SVC-05` resuelve la asignación con `EstrategiaAsignacionAmr`, `AsignacionPorCercania` o `AsignacionPorCarga`. La variación se concentra en la política de cada módulo. |

Trazabilidad en el [registro de decisiones del contrato](../../docs/ARQUITECTURA_BASE.md#12-registro-de-adr-iniciales) mediante `ADR-001 Monolito modular evolutivo con unidades de despliegue`, `ADR-003 La reserva de stock y la selección FEFO pertenecen a Inventario`, `ADR-004 Comandos síncronos y eventos de negocio con Outbox e idempotencia` y `ADR-005 Proyección de lectura (CQRS) con consistencia eventual informada`.

## Borradores del Sprint 3

- [Esqueleto del informe de patrones](HU-30-esqueleto-informe-patrones.md) para HU-30.
- [Evidencia SOLID inicial](HU-31-evidencia-solid-inicial.md) para HU-31.
