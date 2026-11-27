# Disponibilidad y escalabilidad · Smart Inventory

| Campo | Valor |
|---|---|
| Documento | `docs/arquitectura/disponibilidad-escalabilidad.md` |
| Versión | **1.0.0** |
| Estado | Propuesto para revisión (Sprint 3 · HU-29) |
| Dueño | P1 · Líder de arquitectura |
| Revisión | P7 |
| Fecha | 2026-11-27 |
| Contrato de referencia | [`ARQUITECTURA_BASE.md`](../ARQUITECTURA_BASE.md) v1.0.0 (§2.3, §7, §8, §11) |
| Rúbrica | Requisitos de calidad |
| ADR relacionado | [ADR-009](../adr/ADR-009-replicas-locales-de-referencia.md) |

> **Alcance.** Este documento describe cómo se degrada, se recupera y escala el diseño. No define endpoints, mensajes, configuración de broker o colas, despliegue ni CI/CD. Todo identificador es del contrato. Lo que el contrato no define se marca **[PROPUESTA]** y se reúne en la sección 4 para decidir si se eleva como cambio al contrato (§0.3).

**Elementos referenciados** (ID + nombre, primera vez): UD-1 Núcleo transaccional · UD-2 Flota y Operación · UD-3 Analítica · UD-4 Acceso · UD-5 Proyección de lectura · SVC-01 Catálogo y Ubicaciones · SVC-02 Inventario · SVC-03 Reposición y Compras · SVC-04 Pedidos y Fulfillment · SVC-05 Flota y Operación · SVC-06 Analítica · TRX-02 Capa de Aplicación · TRX-03 Broker de eventos · TRX-04 Proyección de lectura · EXT-01 ERP corporativo · EXT-02 Sistemas de transportistas · EXT-03 Controlador de flota AMR · EXT-04 Proveedor de identidad.

**Cómo se informa la frescura al usuario (§8.2).** Los textos "qué ve el usuario" usan los tres estados del contrato: **En vivo** (indicador verde), **Con retraso** (ámbar y hora del último dato, retraso > objetivo y ≤ 60 s) y **Desactualizada** (banner rojo "Los datos pueden no estar al día", retraso > 60 s o conexión perdida). Las acciones siguen disponibles porque siempre se validan en el dueño (PRI-07).

---

## 1. Matriz de fallos y degradación (CA1)

### 1.1 Matriz FAL-01 a FAL-09

| ID | Fallo | Detección | Degradación (qué deja de funcionar y qué sigue) | Recuperación | Qué ve el usuario (UI / Dashboard) |
|---|---|---|---|---|---|
| **FAL-01** | TRX-03 Broker de eventos no disponible | `PublicadorOutbox` no logra publicar y los eventos quedan pendientes en el outbox **[PROPUESTA: señal operativa por eventos pendientes y umbral, a definir en HU-27]**. Para el usuario, la antigüedad del último evento aplicado supera el objetivo de la vista (§8.2). | **Deja de funcionar:** toda interacción A que pasa por TRX-03 (INT-04 réplicas, INT-05 ingesta, INT-08 y INT-32 vistas, INT-10 evaluación de reposición, INT-21 y INT-26 asignación en flota, INT-30 y INT-31 salida y liberación de reserva). Las vistas VL-01 a VL-06 dejan de actualizarse. Las salidas y liberaciones se difieren sin sobreventa (CE-04, CE-05). **Sigue:** SVC-02 acepta recepciones y reservas (RNF-DIS-01); INT-19 es síncrona e interna a UD-1; todas las TX-01 a TX-06 siguen siendo locales. | Los eventos esperaban en el outbox (PRI-06). Al volver TRX-03 se publican **en orden por entidad**, el 100 % sin pérdida y **sin efectos duplicados**: los consumidores descartan duplicados con `ConsumidorIdempotente` (RNF-DIS-01). TRX-04, SVC-03, SVC-06 y las réplicas se ponen al día al aplicar los eventos. | Vistas pasan de **En vivo** a **Con retraso** y a **Desactualizada** (> 60 s). Las posiciones de AMR en VL-04 (objetivo ≤ 2 s) lo muestran primero. Recepciones y pedidos siguen operando con normalidad. |
| **FAL-02** | SVC-02 Inventario no disponible | `AdaptadorReservasInventario` agota el tiempo límite o abre el Circuit Breaker (INT-19, §7.1). TRX-02 no obtiene respuesta en INT-06 e INT-15. | **Deja de funcionar:** registrar recepciones (INT-06), consultar detalle exacto de existencias (INT-15) y confirmar reservas. Los pedidos nuevos quedan en *PendienteReserva* (§9, sin evento). Salidas y liberaciones por INT-30 e INT-31 esperan en el broker (si la espera supera los reintentos acotados, aplica FAL-08). **Sigue:** el registro de pedidos en SVC-04 y el resto de los módulos. Ningún pedido se pierde (RNF-DIS-02). | Reintento automático de la reserva con el **identificador del pedido como clave de idempotencia**; cierra el Circuit Breaker cuando SVC-02 responde. El pedido avanza a *Reservado* (EVT-22) o *EnEspera* (EVT-23). | La UI **conserva el formulario** de recepción y avisa que no se pudo registrar. El dashboard muestra la **última vista con su marca de frescura**. El detalle de lote (INT-15) se muestra desde la proyección marcada como "vista". El pedido en *PendienteReserva* sigue contado como *Registrado* en VL-03 (EVT-21 ya se emitió y esa transición no emite evento). |
| **FAL-03** | Telemetría o EXT-03 Controlador de flota AMR sin señal | SVC-05 detecta que un AMR no envía telemetría por más de **10 s** (RNF-DIS-03, §3.5; INT-23). Si falla el envío de misión, INT-22 (S) no obtiene respuesta. | **Deja de funcionar:** el AMR no recibe misiones; sus misiones *EnCurso* pasan a *Fallida* (§9). Las posiciones en VL-04 se congelan. Si cae EXT-03 completo, aplica a toda la flota. **Sigue:** inventario, pedidos y el resto de los módulos (RNF-DIS-03, ADR-007). | Las misiones *Fallida* vuelven a *Pendiente* y se **reasignan** a otro AMR, o se **hacen manualmente** (picking manual, INT-21). Al reanudarse la telemetría, el AMR vuelve a ser elegible según su estado **[PROPUESTA: criterio exacto de reincorporación]**. | VL-04 marca el AMR **"sin señal"**. ROL-04 puede reasignar o abortar la misión manualmente (INT-33). Los demás indicadores no se afectan. |
| **FAL-04** | SVC-06 Analítica no disponible | TRX-02 no obtiene respuesta de INT-34; SVC-03 no recibe un EVT-37 nuevo (INT-16) y el corte de VL-07 no se renueva **[PROPUESTA: señal operativa de ausencia de EVT-37 y EVT-38]**. | **Deja de funcionar:** pronóstico nuevo (EVT-37), indicadores históricos nuevos (EVT-38) y la ingesta de INT-05. **Sigue:** SVC-03 usa la **última copia local del pronóstico** o `ReposicionMinMax`. **Sin impacto operativo.** | SVC-06 retoma los eventos pendientes y recalcula (sus datos son derivados y recalculables, §3.6); vuelve a publicar EVT-37 y EVT-38 y SVC-03 actualiza su copia local. | VL-07 muestra el **último corte** (frescura de hasta 24 h, CE-07). Sin aviso operativo ni bloqueo de acciones. |
| **FAL-05** | EXT-01 ERP corporativo no disponible | `AdaptadorErp` no obtiene respuesta al enviar la OC (INT-14) o al importar pedidos (INT-18). Alerta en VL-02 **tras un umbral** **[PROPUESTA: valor del umbral, a definir en HU-27]**. | **Deja de funcionar:** el envío de OC (queda *Emitida* sin pasar a *Enviada*) y la importación de pedidos de venta (se retrasan). **Sigue:** aprobar sugerencias y emitir OC (TX-04) y el **registro manual de pedidos** (INT-17). | **Reintentos idempotentes** de INT-14; al volver EXT-01, la OC pasa a *Enviada* y se reanuda la importación (INT-18). | Alerta en **VL-02** tras el umbral. La OC se ve *Emitida* pendiente de envío. ROL-05 y ROL-03 pueden seguir operando con registro manual. |
| **FAL-06** | TRX-04 Proyección de lectura caída o corrupta | INT-03 falla (snapshot o suscripción en vivo); TRX-01 lo detecta por pérdida de conexión o por la marca de último evento aplicado (§8.2). | **Deja de funcionar:** actualización de VL-01 a VL-07. **Sigue:** todos los módulos, comandos y reglas de negocio (TRX-04 no es fuente de verdad, PRI-07). | **Reconstrucción** reprocesando eventos y, si hace falta, snapshots solicitados a los dueños (§8.3); consumidor idempotente y con orden por entidad. UD-5 puede escalar para recuperar el retraso (RNF-ESC-01). | **El dashboard avisa** que la vista está **Desactualizada**. El detalle exacto se **consulta al dueño** (INT-15). Las acciones siguen disponibles y se validan en el dueño. |
| **FAL-07** | EXT-04 Proveedor de identidad no disponible | TRX-02 no obtiene respuesta en INT-02. | **Deja de funcionar:** nuevos inicios de sesión. **Sigue:** las **sesiones vigentes** hasta que expiren. | Al volver EXT-04, se retoman los inicios de sesión; quienes perdieron la sesión por expiración se autentican de nuevo. | No puede iniciar sesión (mensaje de servicio no disponible **[PROPUESTA: texto del aviso, a definir en HU-32]**). Quien ya tiene sesión sigue operando con normalidad. |
| **FAL-08** | Un evento falla repetidamente al procesarse | El consumidor agota los **reintentos acotados** y TRX-03 envía el evento a la **DLQ** (§6.1). | **Deja de funcionar:** solo la aplicación de ese evento en ese consumidor. El efecto sobre los eventos posteriores de la misma entidad **no está definido** en el contrato **[PROPUESTA: definir en HU-24 / HU-27]**. | Se **corrige la causa** y se **reprocesa** desde la DLQ **sin duplicar efectos** (`ConsumidorIdempotente`, PRI-06). | **Alerta operativa** (§6.1). Si el consumidor es TRX-04, la vista afectada se ve **Con retraso** o **Desactualizada**. El destinatario de la alerta no está definido **[PROPUESTA: rol destinatario]**. |
| **FAL-09** | EXT-02 Sistemas de transportistas sin respuesta | El adaptador (`PuertoTransportista`) no obtiene confirmación del vehículo tras los reintentos (INT-27). | **Deja de funcionar:** la confirmación automática del vehículo. El pedido **permanece *EnDespacho*** (no llega a *Despachado*, no se emite EVT-27). **Sigue:** el resto de las operaciones. | Se **reintenta** (con reintentos, INT-27). Mientras tanto, **ROL-04 puede asignar el vehículo manualmente** (INT-33). | El pedido se ve en *EnDespacho* (VL-03). En VL-05 el vehículo no aparece confirmado y ROL-04 dispone de la acción de asignación manual. |

### 1.2 Garantías que se mantienen en todos los escenarios

| Garantía | Por qué se sostiene | Referencia |
|---|---|---|
| El disponible nunca es negativo | La reserva se verifica y se confirma en SVC-02 dentro de TX-02, no en vistas ni réplicas | RNF-CON-01, PRI-05, PRI-07 |
| Ningún pedido se pierde | El pedido queda en *PendienteReserva* y se reintenta con clave de idempotencia | RNF-DIS-02, FAL-02 |
| Ningún evento se pierde ni se duplica | Outbox en la misma transacción del cambio, consumidores idempotentes y DLQ para fallos persistentes | PRI-06, RNF-DIS-01, FAL-01, FAL-08 |
| El retraso se resuelve del lado conservador | Liberar una reserva tarde nunca provoca sobreventa | A4, CE-04, CE-05 |
| El usuario sabe qué tan viejo es su dato | Cada vista muestra "Actualizado hace X s" y el estado de frescura | RNF-CON-02, §8.2 |

---

## 2. Tabla de escalabilidad (CA2 · RNF-ESC-01)

Fuente: columna "Escala según" de §2.3, RNF-ESC-01 y supuestos de carga SUP-01 a SUP-03. La carga pico es 3× la normal (SUP-03), es decir, **180 usuarios concurrentes, 4.500 movimientos/hora y 900 pedidos/hora**.

| UD / consumidor | Qué escala | Señal que dispara el escalamiento | Umbral | Hasta dónde escala | Qué hacer cuando eso no basta |
|---|---|---|---|---|---|
| **UD-1 Núcleo transaccional** (SVC-01 a SVC-04) | UD-1 completa, como un solo bloque (ADR-001) | Usuarios concurrentes y pedidos/hora | Umbral numérico no definido **[PROPUESTA: definir en HU-27]** | Hasta sostener el pico 3× (SUP-03) cumpliendo RNF-REN-02 (reserva ≤ 2 s p95 en pico) y RNF-DIS-04 (≥ 99,5 %). Tope de instancias no definido **[PROPUESTA]** | **Extraer módulos** según §2.4: SVC-02 si RNF-REN-02 se incumple de forma sostenida en pico; SVC-04 si hay picos ≥ 3× que degradan al núcleo; SVC-03 si crecen las integraciones ERP |
| **UD-2 Flota y Operación** (SVC-05) | UD-2 | Número de AMR y vehículos activos | Sin umbral por instancia **[PROPUESTA: definir en HU-27]** | Hasta los **20 AMR** y **4 muelles** de SUP-01, con telemetría de 1 muestra/s por AMR (SUP-02). Más allá de SUP-01 queda fuera de los supuestos | Revisar los supuestos de carga en HU-27 **[PROPUESTA]** |
| **UD-3 Analítica** (SVC-06 + TRX-05) | Consumidores de SVC-06 (ingesta histórica y cálculos) | Retraso de consumo de eventos **> 5 s durante 2 min** (RNF-ESC-01) y volumen de eventos y cálculos | 5 s sostenidos durante 2 min | Hasta cumplir CE-07 (datos crudos ≤ 15 min; indicadores con corte diario). Tope de instancias no definido **[PROPUESTA]** | La carga analítica no se mueve a UD-1: nunca debe consumir recursos transaccionales (§2.3) |
| **UD-4 Acceso** (TRX-01, TRX-02) | Réplicas de UD-4, **sin tocar el núcleo** (INT-01) | Usuarios conectados | Umbral numérico no definido **[PROPUESTA: definir en HU-27]** | Hasta los 180 usuarios concurrentes del pico (SUP-03) | — |
| **UD-5 Proyección de lectura** (TRX-04) | Consumidores de TRX-04 (`ActualizadorVista`) | Retraso de consumo de eventos **> 5 s durante 2 min** (RNF-ESC-01) | 5 s sostenidos durante 2 min | Hasta recuperar la frescura objetivo: ≤ 5 s p95 (CE-01, RNF-REN-03) y ≤ 2 s en posiciones AMR (RNF-REN-04). **Restricción:** más instancias no deben romper el orden por entidad (§6.1, §8.3) | Reconstruir la proyección reprocesando eventos (FAL-06) |
| *Infraestructura* (TRX-03, EXT-04) | No escala el equipo | — | — | Son servicios gestionados, no código del equipo (§2.3) | — |

**Por qué escalar consumidores es seguro:** todo consumidor es idempotente (PRI-06) y recibe los eventos de una misma entidad en orden. Escalar agrega capacidad de procesamiento sin cambiar las reglas de negocio ni la propiedad de los datos.

---

## 3. Análisis de dependencias

La arquitectura contiene los fallos por capas. El **monolito modular** deja a Flota (UD-2) y Analítica (UD-3) fuera del núcleo, así que FAL-03 y FAL-04 no detienen inventario ni pedidos, y la caída de UD-1 no deja sin rutas a Flota si sus réplicas ya están al día. El **Outbox** convierte la caída del broker (FAL-01) en un retraso: SVC-02 sigue aceptando recepciones y reservas, y los eventos salen en orden al volver (RNF-DIS-01). Las **réplicas locales de solo lectura** (PRI-04, [ADR-009](../adr/ADR-009-replicas-locales-de-referencia.md)) permiten validar recepciones y reservas sin llamar a Catálogo, así que SVC-01 o el broker caídos no bloquean el negocio. La **reserva síncrona con tiempo límite, Circuit Breaker e idempotencia** evita perder pedidos cuando SVC-02 falla (FAL-02), y la **proyección reconstruible** (FAL-06) con acciones validadas siempre en el dueño (PRI-07) evita que una vista caída detenga la operación. El punto de mayor concentración de riesgo es **UD-1**, porque SVC-01 a SVC-04 comparten despliegue; por eso RNF-DIS-04 le exige ≥ 99,5 % y la ruta de extracción (§2.4) existe para reducir ese riesgo si la señal aparece.

---

## 4. [PROPUESTA] y vacíos detectados en el contrato

No se usaron como oficiales. Si el equipo las acepta, se elevan como cambio al contrato (§0.3: PR + ADR *Propuesto*; el PR que toca `ARQUITECTURA_BASE.md` exige además la aprobación de P1 y del diseñador del módulo afectado).

| # | Tema | Qué falta o qué se observa | Dónde afecta |
|---|---|---|---|
| 1 | Detección técnica | §11.3 describe el comportamiento esperado pero no cómo se detecta cada fallo. Solo FAL-03 (> 10 s), FAL-08 (reintentos y DLQ) y FAL-05 (umbral sin valor) tienen criterio | FAL-01, FAL-02, FAL-04, FAL-05, FAL-06, FAL-07 |
| 2 | Umbral de alerta del ERP | FAL-05 y INT-14 dicen "tras un umbral" sin valor | FAL-05 |
| 3 | Efecto de la DLQ | No se define qué ocurre con los eventos posteriores de la misma entidad mientras uno está en la DLQ, dado que el orden es por entidad | FAL-08 |
| 4 | Destinatario de la alerta operativa | No se define a qué rol llega | FAL-08 |
| 5 | Falla completa de UD-1 | FAL-02 asume que solo SVC-02 falla. SVC-01 a SVC-04 comparten UD-1, así que una caída de UD-1 no tiene escenario propio. La meta es RNF-DIS-04 | FAL-02, RNF-DIS-04 |
| 6 | Reincorporación de un AMR | No se define cuándo un AMR "sin señal" vuelve a recibir misiones | FAL-03 |
| 7 | Retraso de las réplicas | §8.2 mide la frescura de las vistas, no el retraso de las réplicas (CE-02) | ADR-009 |
| 8 | Umbral de RNF-ESC-01 para SVC-06 | 5 s durante 2 min es mucho más estricto que CE-07 (datos crudos ≤ 15 min). Conviene revisarlo en HU-27 | UD-3 |
| 9 | Umbrales y topes de escalamiento | RNF-ESC-01 da umbral solo para TRX-04 y SVC-06. UD-1, UD-2 y UD-4 solo tienen la métrica, y ninguna UD tiene tope de instancias | Sección 2 |
| 10 | Aviso de EXT-04 caído | El contrato no define el texto que ve el usuario sin sesión | FAL-07 |

---

## Trazabilidad

| Elemento | Contrato |
|---|---|
| FAL-01 a FAL-09 | §11.3 |
| Escalamiento | §2.3, RNF-ESC-01, SUP-01 a SUP-03, §2.4 |
| Frescura y estados de vistas | §8.2, RNF-CON-02 |
| Interacciones citadas | §7.2 (INT-01 a INT-34) |
| Calidad citada | RNF-REN-02 a RNF-REN-04, RNF-DIS-01 a RNF-DIS-04, RNF-CON-01, RNF-ESC-01 |

## Historial de cambios

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-27 | P1 | Versión inicial: matriz de fallos, tabla de escalabilidad y análisis de dependencias | HU-29 |
