# Smart Inventory · Matriz de Interacciones Completa

| Campo | Valor |
|---|---|
| Documento | `docs/arquitectura/matriz-interacciones.md` |
| Versión | **1.1.0** |
| Estado | Propuesto para revisión (Sprint 3 · HU-24) |
| Responsable | P6 · Diseñador de operación (asumida en suplencia) |
| Revisor | P5 · Diseñador de procesos y calidad |
| Fecha | 2026-11-27 |
| Contrato de referencia | [`docs/ARQUITECTURA_BASE.md`](../ARQUITECTURA_BASE.md) v1.0.0 (§6, §7.2) |
| Historia vinculada | HU-24 (HN-04, HN-05, HN-19) |

---

## 1. Introducción

En la arquitectura de **Smart Inventory**, una **Interacción (`INT-nn`)** representa el contrato conceptual de comunicación entre dos o más componentes (capacidades de negocio `SVC`, componentes transversales `TRX` o sistemas externos `EXT`). Cada `INT` establece **quién inicia la comunicación (origen)**, **quién la recibe (destino)**, **el propósito funcional** y **el modo de comunicación**:

* **Síncrono (`S`):** El emisor espera una respuesta inmediata para continuar su ejecución (utilizado para comandos de usuario, consultas de datos exactos a la fuente de verdad o validación de invariantes críticas como la reserva FEFO). Incluye protección mediante tiempos límite y *Circuit Breaker*.
* **Asíncrono (`A`):** El emisor publica un hecho de negocio ya ocurrido mediante eventos (`EVT-nn`) a través del broker (`TRX-03`) aplicando el patrón *Outbox* e idempotencia en los consumidores, desacoplando los tiempos de respuesta y garantizando consistencia eventual acotada.

Esta matriz consolida las 34 interacciones canónicas definidas en `ARQUITECTURA_BASE.md` §7.2, garantizando total coherencia con el diagrama de contenedores (`C4N2-contenedores`) y los diagramas de secuencia de los flujos de negocio (`SEC-FA-recepcion`, `SEC-FB-reposicion` y `SEC-FC-reserva-despacho`). Las flechas que todavía no tienen INT se listan en §3.1.

---

## 2. Matriz de Interacciones por Flujo de Negocio

Para facilitar la lectura y verificación contra los diagramas de secuencia, las interacciones se agrupan según el flujo de negocio principal al que pertenecen: **Transversal (T)**, **Flujo A (Recepción)**, **Flujo B (Reposición y Compras)** y **Flujo C (Reserva FEFO, Preparación y Despacho)**. *(Nota: Algunas interacciones transversales o compartidas participan en múltiples flujos, lo cual se especifica en la columna "Flujo").*

### 2.1 Interacciones Transversales y de Infraestructura (`T` / `Todos`)

Comprenden el acceso de usuarios, la autenticación, la actualización de vistas en vivo, la sincronización de réplicas de datos maestros y la ingesta analítica.

| ID | Origen | Destino | Propósito | Modo | Efecto en el flujo de negocio | Efecto si falla | Flujo | EVT relacionado |
|---|---|---|---|---|---|---|---|---|
| **INT-01** | TRX-01 | TRX-02 | Canalizar toda consulta y comando operativo iniciado por el usuario en el dashboard. | S | Habilita la interacción humana con las capacidades del sistema y la visualización de datos. | La interfaz de usuario queda inoperativa; se mitiga replicando horizontalmente `UD-4`. | Todos | — |
| **INT-02** | TRX-02 | EXT-04 | Autenticar la identidad del usuario y obtener sus roles (`ROL-01` a `ROL-07`). | S | Garantiza que solo usuarios autenticados y con roles válidos accedan a las operaciones. | Impide nuevos inicios de sesión; las sesiones activas continúan operando hasta expirar (`FAL-07`). | Todos | — |
| **INT-03** | TRX-02 | TRX-04 | Leer el *snapshot* inicial de las vistas y suscribirse a actualizaciones en vivo. | S + A (push) | Mantiene al operador informado en tiempo real sobre KPIs, alertas y mapas de planta. | Las vistas muestran el último estado conocido marcado con banner rojo "Desactualizada" (`FAL-06`). | Todos | Todos los `EVT` |
| **INT-04** | SVC-01 | TRX-03 → SVC-02, SVC-03, SVC-04, SVC-05 | Propagar altas y cambios de productos (SKU), categorías, bodegas y ubicaciones hacia réplicas locales. | A | Permite que los módulos validen recepciones, políticas y rutas localmente sin llamadas síncronas a `SVC-01`. | La réplica local se atrasa; un SKU o ubicación recién creados tardan unos segundos en estar operativos (`CE-02`). | T | EVT-01, EVT-02, EVT-03 |
| **INT-05** | Todos los SVC | TRX-03 → SVC-06 / TRX-05 | Ingestar todos los hechos de negocio (y señales muestreadas) hacia el almacén histórico. | A | Alimenta la zona cruda y curada del *Data Lakehouse* para cálculos históricos y predicciones. | Retraso en la actualización de indicadores analíticos sin impacto en la operación transaccional (`CE-07`). | T | EVT-01 a EVT-36 |
| **INT-34** | TRX-02 | SVC-06 | Consultar indicadores históricos consolidados (rotación, quiebres, vencimientos, puntualidad). | S | Provee métricas estratégicas de desempeño para la vista `VL-07` del dashboard. | `VL-07` continúa mostrando el último corte diario disponible sin afectar la operación (`FAL-04`). | T | EVT-38 |

### 2.2 Flujo A · Recepción y Actualización de Existencias (`A`)

Comprende el ingreso de mercadería a bodega, la actualización transaccional de saldos y lotes, y la notificación a las vistas y órdenes de compra asociadas.

| ID | Origen | Destino | Propósito | Modo | Efecto en el flujo de negocio | Efecto si falla | Flujo | EVT relacionado |
|---|---|---|---|---|---|---|---|---|
| **INT-06** | TRX-02 | SVC-02 | Registrar una recepción de mercadería validando SKU, lote, vencimiento, ubicación y OC opcional. | S | Ejecuta `TX-01`: crea el lote (si es nuevo), registra el movimiento inmutable y actualiza el saldo disponible. | La recepción no se registra; la UI conserva los datos ingresados y notifica al operador (`FAL-02`). | A | — |
| **INT-07** | SVC-02 | TRX-03 | Publicar los hechos de recepción y cambio de saldo desde la tabla *Outbox* hacia el broker. | A | Comunica al resto de la arquitectura que el inventario físico y disponible ha aumentado. | Los eventos permanecen persistidos en el *Outbox* local y se publican al restablecerse `TRX-03` (`FAL-01`). | A | EVT-04, EVT-05 |
| **INT-08** | TRX-03 | TRX-04 | Consumir eventos de recepción y saldo para actualizar las vistas operativas del dashboard. | A | Refleja las nuevas existencias y movimientos recientes en `VL-01`, `VL-04` y `VL-06`. | El dashboard muestra el indicador de frescura en ámbar o rojo con retraso visible (`CE-01`). | A | EVT-04, EVT-05 |
| **INT-09** | TRX-03 | SVC-03 | Notificar recepciones vinculadas a una Orden de Compra para actualizar su estado de cumplimiento. | A | Permite a `SVC-03` marcar la OC como *RecibidaParcial* o *Cerrada* al completarse la cantidad esperada. | El avance o cierre de la OC se difiere hasta que el evento sea procesado, sin afectar el saldo (`CE-06`). | A, B | EVT-04, EVT-20 |

### 2.3 Flujo B · Detección de Riesgo de Agotamiento y Reposición (`B`)

Comprende la evaluación continua de saldos contra políticas de inventario, la generación y aprobación de sugerencias de compra y el envío de órdenes al ERP.

| ID | Origen | Destino | Propósito | Modo | Efecto en el flujo de negocio | Efecto si falla | Flujo | EVT relacionado |
|---|---|---|---|---|---|---|---|---|
| **INT-10** | TRX-03 | SVC-03 | Evaluar cambios de saldo disponible contra la política de mínimos y punto de reorden del SKU. | A | Detecta riesgo de agotamiento (bajo mínimo) o resuelve una alerta activa si el stock se recuperó. | La detección de riesgo o resolución de alerta se difiere temporalmente (`CE-03`). | B | EVT-05, EVT-14, EVT-15 |
| **INT-11** | SVC-03 | TRX-03 | Publicar alertas de riesgo de agotamiento, resoluciones y sugerencias de reposición calculadas. | A | Informa a `VL-02` que existe un riesgo activo y pone a disposición una sugerencia vigente para decisión humana. | Los eventos quedan resguardados en el *Outbox* de `SVC-03` hasta que el broker responda (`FAL-01`). | B | EVT-14, EVT-15, EVT-16 |
| **INT-12** | TRX-02 | SVC-03 | Ejecutar la decisión humana del Comprador (`ROL-03`) de aprobar o rechazar una sugerencia vigente. | S | Valida vigencia de la sugerencia y ejecuta `TX-04`: cambia estado de sugerencia y crea la OC en estado *Emitida*. | La acción no se ejecuta; se informa al comprador para que reintente más tarde. | B | — |
| **INT-13** | SVC-03 | TRX-03 | Publicar la resolución de la sugerencia (aprobada/rechazada) y la emisión de la nueva OC. | A | Actualiza `VL-02` y notifica a `SVC-02` (recepciones esperadas) y `SVC-05` (llegadas esperadas de proveedores). | Los eventos esperan en el *Outbox* local de `SVC-03` sin perder la OC ya creada (`FAL-01`). | B | EVT-17, EVT-18, EVT-19 |
| **INT-14** | SVC-03 | EXT-01 (adaptador) | Transmitir la Orden de Compra emitida al ERP corporativo / proveedor externo. | A (con reintentos) | Formaliza la compra en el sistema externo y transiciona la OC a estado *Enviada*. | La OC permanece en estado *Emitida* pendiente de envío con reintentos idempotentes y alerta en `VL-02` (`FAL-05`). | B | EVT-19 |
| **INT-15** | TRX-02 | SVC-02 | Consultar directamente a la fuente de verdad el detalle exacto de existencias por SKU, ubicación y lote. | S | Provee datos exactos sin retraso de proyección para decisiones críticas de compra o auditoría de lotes. | Se degrada mostrando los datos disponibles en la proyección (`TRX-04`) marcados explícitamente como "vista". | B, C | — |
| **INT-16** | SVC-06 | TRX-03 → SVC-03 | Actualizar la copia local del pronóstico de demanda por SKU en `SVC-03`. | A | Permite que `EstrategiaReposicion` utilice `ReposicionPorPronostico` al calcular futuras sugerencias. | `SVC-03` opera con el último pronóstico válido o conmuta a `ReposicionMinMax` sin detener el flujo (`FAL-04`). | B | EVT-37 |

### 2.4 Flujo C · Reserva FEFO, Preparación y Despacho (`C`)

Comprende el ciclo completo del pedido: ingreso, reserva síncrona con FEFO, picking automatizado con AMR, asignación de muelles/vehículos y salida física de bodega.

| ID | Origen | Destino | Propósito | Modo | Efecto en el flujo de negocio | Efecto si falla | Flujo | EVT relacionado |
|---|---|---|---|---|---|---|---|---|
| **INT-17** | TRX-02 | SVC-04 | Registrar manualmente un pedido de venta en el sistema (`ROL-05`). | S | Crea el pedido en estado *Registrado* (`TX-05`) e inicia el proceso de reserva de stock. | El pedido no se registra; la interfaz informa el error al supervisor de operaciones. | C | EVT-21 |
| **INT-18** | EXT-01 | SVC-04 (adaptador) | Importar automáticamente pedidos de venta originados en el ERP corporativo. | A | Incorpora pedidos externos al ciclo de fulfillment sin intervención manual. | Se retrasa la importación automática; el registro manual (`INT-17`) permanece operativo (`FAL-05`). | C | EVT-21 |
| **INT-19** | SVC-04 | SVC-02 | **Solicitar reserva de stock aplicando regla FEFO** (todo o nada) para las líneas de un pedido. | **S** (timeout + Circuit Breaker) | Ejecuta `TX-02`: bloquea lotes disponibles con vencimiento más próximo, evitando sobreventa (`RNF-CON-01`). | El pedido pasa a estado *PendienteReserva* y se reintenta de forma idempotente usando el ID del pedido (`FAL-02`). | C | — |
| **INT-20** | SVC-02 | TRX-03 | Publicar el resultado de la solicitud de reserva (confirmada o rechazada) y el nuevo saldo disponible. | A | Permite actualizar las vistas de disponibilidad y notificar demanda no atendida (`EVT-09`) a `SVC-03`. | Los eventos esperan en el *Outbox* de `SVC-02`; la reserva ya quedó asegurada en `TX-02` (`FAL-01`). | C | EVT-08, EVT-09, EVT-05 |
| **INT-21** | SVC-04 | TRX-03 → SVC-05 | Notificar que un pedido reservado requiere ejecución de tarea de preparación (*picking*). | A | Dispara en `SVC-05` la planificación de ruta y selección de un AMR disponible (`EstrategiaAsignacionAmr`). | La tarea queda en cola; el operador puede ejecutar picking manual sin bloquear la operación. | C | EVT-24 |
| **INT-22** | SVC-05 | EXT-03 | Enviar la instrucción de misión y ruta calculada al controlador físico de flota AMR. | S | Inicia el desplazamiento físico del robot hacia las ubicaciones asignadas por FEFO. | La misión pasa a *Fallida*; `SVC-05` reasigna a otro AMR o deriva a picking manual (`FAL-03`). | C | EVT-29 |
| **INT-23** | EXT-03 | SVC-05 | Recibir flujo continuo de telemetría (posición, nivel de batería, estado operativo) de los AMR. | A (flujo continuo) | Permite supervisar la flota en tiempo real y descartar robots detenidos o con batería baja en nuevas asignaciones. | Si supera 10 s sin señal, el AMR se marca "sin señal" en `VL-04` y deja de recibir misiones (`FAL-03`). | C | EVT-31, EVT-32 |
| **INT-24** | SVC-05 | TRX-03 → SVC-04 | Informar la asignación y finalización de la misión AMR asociada a una tarea de preparación. | A | Mantiene trazabilidad en `SVC-04` sobre el avance físico del *picking* antes de la confirmación final. | La actualización visual del estado de la tarea en `SVC-04` se difiere sin detener el flujo físico. | C | EVT-29, EVT-30 |
| **INT-25** | TRX-02 | SVC-04 | Confirmar el término de la preparación física del pedido por parte del operador (`ROL-01`). | S | Transiciona el pedido a *Preparado* (`EVT-25`) y dispara la solicitud de despacho (`EVT-26`). | No se registra la confirmación; se notifica al operador para reintentar. | C | EVT-25, EVT-26 |
| **INT-26** | SVC-04 | TRX-03 → SVC-05 | Solicitar asignación de muelle y coordinación de vehículo para un pedido preparado. | A | Activa en `SVC-05` la gestión de patio, reserva de muelle (`AsignadorMuelles`) y coordinación con transportista. | El despacho queda en espera hasta que el evento sea consumido por `SVC-05`. | C | EVT-26 |
| **INT-27** | SVC-05 | EXT-02 (adaptador) | Solicitar y confirmar asignación de vehículo y ventana horaria con sistemas de transportistas. | A (con reintentos) | Coordina la llegada programada del transporte externo para el retiro de pedidos. | El pedido permanece *EnDespacho*; se reintenta y `ROL-04` puede asignar vehículo manualmente (`FAL-09`). | C | — |
| **INT-28** | EXT-05 | SVC-05 | Recibir eventos automáticos de acceso y salida de camiones desde garita o sensores de patio. | A | Registra el arribo (`VehiculoArribado`) o salida física del vehículo, actualizando la ocupación del patio. | El operador de patio y flota (`ROL-04`) registra el movimiento manualmente mediante `INT-33`. | A, C | EVT-33, EVT-35 |
| **INT-29** | SVC-05 | TRX-03 → SVC-04 | Notificar la asignación de muelle y la salida efectiva del camión cargado del patio. | A | Permite a `SVC-04` vincular el muelle al pedido y cerrarlo transaccionalmente en estado *Despachado* (`EVT-27`). | El cierre administrativo del pedido en `SVC-04` se difiere hasta procesar el evento. | C | EVT-34, EVT-35, EVT-27 |
| **INT-30** | SVC-04 | TRX-03 → SVC-02 | Comunicar que el pedido fue despachado para consumir la reserva y descontar el saldo físico. | A | Ejecuta `TX-03` en `SVC-02`: consume la reserva, registra la salida inmutable (`EVT-11`) y actualiza saldo (`EVT-05`). | Sin riesgo de sobreventa: el stock ya estaba bloqueado en reserva hasta que se procese la salida (`CE-04`). | C | EVT-27, EVT-11, EVT-05 |
| **INT-31** | SVC-04 | TRX-03 → SVC-02, SVC-05 | Comunicar la cancelación de un pedido (previo a *EnDespacho*) para ejecutar compensación. | A | Libera la reserva de stock en `SVC-02` (`EVT-10`) y aborta misiones AMR pendientes en `SVC-05`. | El stock permanece reservado temporalmente (lado conservador) hasta procesar la liberación (`CE-05`). | C | EVT-28, EVT-10 |
| **INT-32** | SVC-05 | TRX-03 → TRX-04 | Transmitir eventos de estado de flota, muelles, vehículos y señales muestreadas de posición AMR. | A | Actualiza en tiempo real los mapas operativos `VL-04` (planta interna) y `VL-05` (planta externa). | Las vistas muestran retraso en su indicador de frescura; las señales `EVT-32` perdidas se descartan sin impacto. | C | EVT-31 a EVT-36 |
| **INT-33** | TRX-02 | SVC-05 | Ejecutar operaciones manuales de patio y flota (`ROL-04`): registrar llegada, asignar muelle o reasignar AMR. | S | Ejecuta `TX-06` o reasignación directa permitiendo contingencia operativa ante fallos de sensores o robots. | La acción manual no se aplica y se informa al operador de patio y flota. | C | EVT-33, EVT-34, EVT-36 |

---

## 3. Verificación de Coherencia Arquitectónica

* **Alineación con C4 Nivel 2 (`C4N2-contenedores`):** Todas las interacciones síncronas (`S`) respetan los límites de contenedor y el máximo de dos saltos (`TRX-01 → TRX-02 → SVC-nn`, y únicamente `SVC-04 → SVC-02` en `INT-19`). Ningún módulo accede directamente al esquema de datos de otro (`PRI-02`, `RNF-MAN-02`).
* **Alineación con Secuencias (`SEC-FA-recepcion`, `SEC-FB-reposicion`, `SEC-FC-reserva-despacho`):**
  * **Flujo A:** usa `INT-01`, `INT-06`, `INT-07`, `INT-08` e `INT-09`, más una flecha «INT por definir» (P-01 en §3.1).
  * **Flujo B:** usa `INT-01`, `INT-03`, `INT-10` a `INT-14` e `INT-16`, más cuatro flechas «INT por definir» (P-02 a P-05).
  * **Flujo C:** usa `INT-17` a `INT-32`, más dos flechas «INT por definir» (P-06 y P-07).
  * Todas las demás flechas de las tres secuencias tienen su INT. Las 7 sin ID se listan en §3.1 y quedan pendientes de aprobación como cambio MINOR del contrato (ARQUITECTURA_BASE §0.3). Por eso CA3 de HU-24 se cumple por completo solo cuando P1 las apruebe y se numeren como INT-35 en adelante.
  * El recorrido integral (HU-38) se verificará contra esta matriz cuando exista su secuencia.

### 3.1 Interacciones pendientes de aprobación

Aparecen en las secuencias como «INT por definir». **No son IDs oficiales**: se usan los códigos locales P-nn hasta que se aprueben.

| Código | Origen → destino | Propósito | Modo | EVT | Flujo | Origen del hallazgo |
|---|---|---|---|---|---|---|
| P-01 | SVC-03 → TRX-03 → SVC-02, SVC-05 | Mantener la réplica de OC (recepciones y llegadas esperadas) | A | EVT-19, EVT-20 | A, B | PC-01 (flujo A) |
| P-02 | TRX-03 → TRX-04 | Actualizar VL-01 y VL-02 con la alerta, la sugerencia y la OC | A | EVT-14 a EVT-19 | B | PC-05 (a) (flujo B) |
| P-03 | TRX-03 → SVC-03 | Llevar la demanda no atendida a la réplica de SVC-03 | A | EVT-09 | B, C | PC-05 (b) (flujo B) |
| P-04 | TRX-02 → SVC-03 | Consultar el detalle de la sugerencia vigente | S | — | B | PC-05 (c) (flujo B) |
| P-05 | TRX-02 → SVC-03 | Configurar la política de reposición del SKU (OPS-04) | S | — | B | PC-05 (d) (flujo B) |
| P-06 | TRX-02 → SVC-04 | Cancelar un pedido (OPS-05) | S | EVT-28 | C | Flujo C, §7 |
| P-07 | TRX-03 → TRX-04 | Actualizar VL-03 con el ciclo del pedido | A | EVT-21 a EVT-28 | C | Flujo C, §7 |

---

## 4. Nota de Alcance Técnico

> **Exclusión explícita según `ARQUITECTURA_BASE.md` §14:** Este documento describe exclusivamente la arquitectura conceptual de interacciones (origen, destino, propósito, modo `S`/`A`, impacto en el negocio, resiliencia ante fallos y eventos asociados). **No incluye ni define** diseños detallados de APIs, rutas o *endpoints*, verbos o métodos HTTP, códigos de estado, contratos JSON/OpenAPI, esquemas de mensajes ni configuraciones técnicas de colas o del broker (`TRX-03`), los cuales corresponden al ámbito de *Desarrollo de Software Moderno*.

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-27 | P1 (suplencia de P6) | Versión inicial: 34 INT por flujo y verificación de coherencia | HU-24 |
| 1.1.0 | 2026-11-27 | P5 (revisora) | Corrige §3: las secuencias tienen 7 flechas sin INT, que ahora se listan en §3.1 como pendientes de aprobación (CA3). Quita la referencia a la secuencia del recorrido vertical, que aún no existe (HU-38), y el enlace suelto a un adjunto en la línea 1 | HU-24 |
