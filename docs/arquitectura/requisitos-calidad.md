# Smart Inventory · Requisitos de calidad verificables

| Campo | Valor |
|---|---|
| Documento | `docs/arquitectura/requisitos-calidad.md` |
| Versión | **1.0.1** |
| Estado | Aprobado: PR #59 fusionado el 2026-11-27 |
| HU | HU-27 · Requisitos de calidad verificables (Sprint 3 · 5 pts) |
| Dueño / Revisor | P5 / P4 |
| Contrato de referencia | [`docs/ARQUITECTURA_BASE.md`](../ARQUITECTURA_BASE.md) v1.0.0 (§7.3, §8.2, §11) |
| Insumos | `docs/gestion/riesgos.md` (H-01, H-02 y H-04), `docs/arquitectura/flujos/flujo-a-recepcion.md`, `docs/arquitectura/flujos/flujo-b-reposicion.md` |
| Criterio de la rúbrica | Requisitos de calidad |
| Fecha | 2026-11-27 |

---

## 1. Propósito y criterios de aceptación

Convierte la base de calidad del contrato (§11) en requisitos **verificables**. Cada requisito se escribe como un escenario con cinco partes: **fuente · estímulo · entorno · respuesta · medida**.

| CA de HU-27 | Dónde se cumple |
|---|---|
| CA1. Al menos 2 RNF por atributo, en formato de escenario | §3: 33 RNF en total (15 semilla y 18 nuevos), con al menos 3 por atributo |
| CA2. Umbrales justificados con SUP-01 a SUP-03 | §2: volúmenes derivados de los supuestos y justificación de cada umbral |
| CA3. Cada RNF vinculado a un SVC, INT o VL y al patrón o ADR que lo hace posible | §4: matriz de trazabilidad |
| CA4. Los RNF semilla se conservan o su cambio queda registrado | §5: los 15 se conservan sin cambio de texto; las aclaraciones y las propuestas quedan registradas |

También cierra tres hallazgos del registro de riesgos:
- **H-01:** latencia de la alerta del flujo B.
- **H-02:** FAL-04 a FAL-09 sin RNF.
- **H-04:** escalabilidad con un solo RNF y "horario operativo" sin definir.

> **Alcance.** Este documento define requisitos de diseño y cómo se miden. La verificación con pruebas de software corresponde a Desarrollo de Software Moderno (ARQUITECTURA_BASE §14). Aquí cada RNF se verifica en el diseño, revisando el artefacto que muestra el mecanismo (§4).

---

## 2. Supuestos de carga y justificación de umbrales (CA2)

### 2.1 Supuestos del contrato (§11.1): se validan sin cambios

| ID | Supuesto | Qué se deriva para este documento |
|---|---|---|
| SUP-01 | 1 centro de distribución, 4 muelles, 20 AMR, ~5.000 SKU activos y ~30.000 combinaciones SKU · ubicación · lote | Tamaño de las vistas VL-01, VL-02 y VL-04 y de las réplicas de catálogo |
| SUP-02 | Carga normal: 60 usuarios concurrentes, 1.500 movimientos/h, 300 pedidos/h, telemetría de 20 AMR a 1 muestra/s | Volúmenes de la tabla 2.2 |
| SUP-03 | Carga pico: 3× la carga normal | Volúmenes pico de la tabla 2.2 |

### 2.2 Volúmenes derivados (estimación de orden de magnitud)

| Magnitud | Carga normal (SUP-02) | Carga pico (SUP-03) | Cómo se estima |
|---|---|---|---|
| Movimientos de inventario | 1.500/h ≈ **0,4/s** | 4.500/h ≈ **1,25/s** | Dato directo |
| Pedidos | 300/h ≈ **0,08/s** | 900/h ≈ **0,25/s** | Dato directo |
| Eventos de negocio por movimiento | ≈ 2 | ≈ 2 | El evento del movimiento (EVT-04, EVT-06, EVT-07 o EVT-11) más EVT-05 |
| Eventos de negocio por pedido | ≈ 12 | ≈ 12 | EVT-21 a EVT-28, EVT-08, EVT-29, EVT-30, EVT-34 y EVT-35 |
| **Eventos de negocio en total** | ≈ 7.000/h ≈ **2/s** | ≈ 21.000/h ≈ **6/s** | Movimientos × 2 + pedidos × 12 |
| Señales de telemetría (EVT-32) | **20/s** | 20/s (la flota no cambia con la demanda) | 20 AMR × 1 muestra/s |
| Usuarios concurrentes | 60 | 180 | Dato directo |

### 2.3 Por qué los umbrales son pertinentes

- **El volumen es bajo.** Con unos 2 eventos/s (6 en pico), el tiempo de procesamiento no es lo que limita. Lo que limita es la **cadena de saltos asíncronos** (outbox → broker → consumidor → vista). Por eso los umbrales de frescura están en segundos y se reparten por salto.
- **Usuario esperando (S1, §7.1):** las operaciones síncronas tienen objetivos de 0,5 a 1 s en carga normal, y el doble en pico, para no interrumpir el trabajo en bodega.
- **Decisiones de horas o días:** reponer y comprar toleran retrasos de segundos (CE-03) y el envío al ERP tolera minutos (RNF-DIS-06).
- **Visualización de AMR:** a 1 muestra/s, ≤ 2 s equivale como máximo a una o dos muestras de atraso en VL-04 (RNF-REN-04).

### 2.4 Supuesto nuevo («PROPUESTA DE CAMBIO»)

| ID | Supuesto propuesto | Por qué | Valida |
|---|---|---|---|
| SUP-04 | **Horario operativo:** dos turnos, de 06:00 a 22:00, de lunes a sábado (16 h × 6 días ≈ 416 h al mes) | RNF-DIS-04 usa "horario operativo" sin definirlo (H-04). Con este supuesto, 99,5 % equivale a **un máximo de unos 125 minutos de indisponibilidad de UD-1 al mes** | P2 (PO) · contrato §11.1 (PC-10) |

---

## 3. Requisitos por atributo (CA1)

**Origen:** *Semilla* = texto de ARQUITECTURA_BASE §11.2, sin cambios. *Nuevo* = definido en esta HU. Los IDs nuevos continúan la numeración del contrato.

### 3.1 Rendimiento

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Origen |
|---|---|---|---|---|---|---|
| RNF-REN-01 | ROL-01 en TRX-01 | Consulta el detalle por lote de un SKU (INT-15) | Carga normal (SUP-02) | SVC-02 devuelve físico, reservado y disponible por lote | **≤ 500 ms p95** | Semilla |
| RNF-REN-02 | SVC-04 | Solicita una reserva FEFO (INT-19) | Carga normal y carga pico | SVC-02 confirma o rechaza la reserva | **≤ 1 s p95** en normal y **≤ 2 s p95** en pico | Semilla |
| RNF-REN-03 | ROL-01 | Registra una recepción (INT-06) | Carga normal | La recepción se refleja en VL-01 | **95 % en < 5 s** desde la confirmación | Semilla |
| RNF-REN-04 | EXT-03 | Envía la posición de un AMR (INT-23) | Carga normal, 20 AMR | La posición se refleja en VL-04 (EVT-32) | **≤ 2 s p95** | Semilla |
| RNF-REN-05 | SVC-02 | Confirma un cambio de saldo que deja un SKU bajo su mínimo (EVT-05) | Carga normal, broker disponible | SVC-03 detecta el riesgo (EVT-14) y la alerta aparece en VL-02 | **95 % en ≤ 5 s** desde el commit del cambio de saldo. Presupuesto: ≤ 3 s hasta EVT-14 en el outbox de SVC-03 (CE-03 propuesto, PC-09) + ≤ 2 s hasta VL-02 | **Nuevo (H-01)** |
| RNF-REN-06 | ROL-03 en TRX-01 | Aprueba o rechaza una sugerencia (INT-12) | Carga normal | SVC-03 valida el estado vigente y, si aprueba, ejecuta TX-04 | **≤ 1 s p95** | Nuevo |
| RNF-REN-07 | ROL-01 en TRX-01 | Registra una recepción (INT-06) | Carga normal y carga pico | SVC-02 valida (V1 a V5 de HU-16) y confirma TX-01 | **≤ 1 s p95** en normal y **≤ 2 s p95** en pico | Nuevo |

### 3.2 Disponibilidad

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Origen |
|---|---|---|---|---|---|---|
| RNF-DIS-01 | TRX-03 | El broker deja de estar disponible (FAL-01) | Operación normal | SVC-02 sigue aceptando recepciones y reservas; los eventos esperan en el outbox | Al volver: **100 % de los eventos publicados**, **0 pérdidas** y **0 efectos duplicados** | Semilla |
| RNF-DIS-02 | SVC-02 | No responde a INT-19 (FAL-02) | Pedido en registro | El pedido queda *PendienteReserva* y se reintenta automáticamente | **0 pedidos perdidos** | Semilla (ver aclaración en §5) |
| RNF-DIS-03 | EXT-03 / telemetría | Un AMR deja de enviar señal por más de 10 s (FAL-03) | Operación normal | VL-04 lo marca "sin señal" y no recibe misiones | **0 misiones** a AMR sin señal; inventario y pedidos **sin impacto** | Semilla |
| RNF-DIS-04 | — | Uso mensual de UD-1 | Horario operativo (SUP-04 propuesto) | UD-1 disponible | **≥ 99,5 %**, unos 125 min de indisponibilidad máxima al mes con SUP-04 | Semilla (ver aclaración en §5) |
| RNF-DIS-05 | SVC-06 | Analítica no está disponible (FAL-04) | Llega EVT-05, carga normal | SVC-03 evalúa con el último pronóstico o con `ReposicionMinMax`; VL-07 muestra el último corte | **100 % de los EVT-05** evaluados dentro del objetivo de CE-03; **0 fallas** de INT-12 causadas por SVC-06 | **Nuevo (H-02)** |
| RNF-DIS-06 | EXT-01 | El ERP no está disponible (FAL-05) | Se aprueba una sugerencia (TX-04); el ERP puede estar caído hasta 4 h | La OC queda *Emitida* pendiente de envío, con reintentos idempotentes por INT-14 | **0 OC perdidas o duplicadas** en el ERP; **alerta en VL-02** si una OC lleva más de 30 min sin enviarse; **100 % enviadas ≤ 15 min** después de que vuelva el ERP | **Nuevo (H-02)** |
| RNF-DIS-07 | TRX-04 | La proyección cae o se corrompe (FAL-06) | Horario operativo, carga normal | El dashboard avisa "Desactualizada"; el detalle se consulta al dueño (INT-15); la proyección se reconstruye | Vistas **reconstruidas en ≤ 30 min** (hasta 24 h de eventos más snapshots de los dueños); durante la falla, INT-15 cumple RNF-REN-01 y **0 acciones bloqueadas**, porque se validan en el dueño (PRI-07) | **Nuevo (H-02)** |
| RNF-DIS-08 | EXT-04 | El proveedor de identidad no responde (FAL-07) | Usuarios con sesión vigente | Las sesiones vigentes siguen hasta expirar y los nuevos inicios de sesión se rechazan con un mensaje claro | **0 sesiones vigentes interrumpidas** por la falla | **Nuevo (H-02)** |
| RNF-DIS-09 | Cualquier consumidor (`ConsumidorIdempotente`) | Un evento falla repetidamente (FAL-08) | Carga normal | Tras reintentos acotados, el evento pasa a la DLQ y se genera una alerta operativa; se reprocesa al corregir la causa | **100 % a la DLQ** (0 eventos descartados en silencio); **alerta ≤ 1 min**; **0 efectos duplicados** al reprocesar | **Nuevo (H-02)** |
| RNF-DIS-10 | EXT-02 | El transportista no responde (FAL-09) | Pedido *EnDespacho* | El pedido permanece *EnDespacho*, se reintenta INT-27 y ROL-04 puede asignar el vehículo manualmente (INT-33) | **0 pedidos** que cambien de estado por la falla; asignación manual **disponible el 100 %** del tiempo de la falla | **Nuevo (H-02)** |

### 3.3 Consistencia

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Origen |
|---|---|---|---|---|---|---|
| RNF-CON-01 | Varios pedidos (SVC-04) | Reservas concurrentes del mismo SKU · ubicación · lote (INT-19) | Carga pico | TX-02 verifica el disponible dentro de la transacción | **0 reservas** que superen el disponible | Semilla |
| RNF-CON-02 | TRX-04 | Una vista acumula retraso | Cualquier carga | La vista muestra su antigüedad con los 3 estados de §8.2 | **100 % de las vistas** con "Actualizado hace X s"; aviso si el retraso supera 60 s | Semilla |
| RNF-CON-03 | Módulo dueño | Confirma una transacción local (TX-01 a TX-06) | Incluso si el proceso falla justo después del commit | El cambio de negocio y su evento quedan guardados juntos (Outbox) | **0 cambios sin evento** y **0 eventos sin cambio** (conciliación entre el outbox y los cambios) | Nuevo |
| RNF-CON-04 | SVC-01, SVC-02, SVC-03, SVC-04 | Publican un hecho que alimenta una réplica (EVT-01 a EVT-03, EVT-05 hacia SVC-03, EVT-19, EVT-20) o una compensación (EVT-28) | Carga normal, broker disponible | Las réplicas se actualizan y las reservas canceladas se liberan | **95 % de las réplicas ≤ 5 s** (CE-02); liberación de reserva por cancelación **≤ 60 s** (CE-05) | Nuevo |
| RNF-CON-05 | SVC-03 | Llegan varios EVT-05 del mismo SKU, o un comprador aprueba desde una vista atrasada | Carga normal | Solo hay una sugerencia vigente por SKU y la aprobación se valida contra el estado vigente (V2 de HU-17) | **0 SKU** con dos sugerencias *Generada* a la vez; **0 aprobaciones** de sugerencias no vigentes | Nuevo |

### 3.4 Seguridad

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Origen |
|---|---|---|---|---|---|---|
| RNF-SEG-01 | Cualquier usuario | Ejecuta una operación sensible (OPS-01 a OPS-08) | Cualquier carga | La capacidad dueña verifica el rol y registra la auditoría | **100 %** de las OPS con rol verificado en el dueño y auditadas (quién, cuándo, qué, motivo) | Semilla |
| RNF-SEG-02 | Integrante del equipo | Abre un PR | Repositorio del equipo | La checklist del PR revisa que no haya secretos | **0 secretos** en el repositorio | Semilla |
| RNF-SEG-03 | ROL-07 (OPS-07) | Asigna roles a un usuario | Gestión de usuarios en EXT-04 | Se impide combinar ROL-03 (aprueba OC) con ROL-01 o ROL-02 (registran recepciones) | **0 usuarios** con ROL-03 y ROL-01 o ROL-02 a la vez (separación de funciones, §11.4) | **Nuevo (H-05)** |
| RNF-SEG-04 | Un módulo | Llama de forma síncrona a otro módulo (por ejemplo, INT-19) | MVP y después de extraer módulos (§2.4) | La llamada usa la identidad propia del módulo que llama (§11.4) | **100 %** de las llamadas entre módulos autenticadas; **0 llamadas anónimas** | Nuevo |

### 3.5 Escalabilidad

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Origen |
|---|---|---|---|---|---|---|
| RNF-ESC-01 | Consumidores de eventos | El retraso de consumo supera 5 s durante 2 min | Cualquier carga | TRX-04 y los consumidores de SVC-06 escalan horizontalmente; UD-2 escala con los AMR activos y UD-4 con los usuarios conectados | Retraso de consumo de vuelta **≤ 5 s** | Semilla |
| RNF-ESC-02 | Temporada alta | La carga pasa de normal a pico (3×, SUP-03) | Pico sostenido | UD-1 agrega instancias; la señal es más de 120 usuarios concurrentes o un p95 de INT-19 mayor a 1,5 s durante 5 min | RNF-REN-01 (≤ 500 ms) y RNF-REN-02 (≤ 2 s en pico) **se mantienen**. Si no se mantienen de forma sostenida, se dispara la extracción de SVC-02 (§2.4) | **Nuevo (H-04)** |
| RNF-ESC-03 | Operación | La flota se duplica (de 20 a 40 AMR, 40 señales/s) | Carga normal en el resto del sistema | Solo escala UD-2 (ADR-007) | RNF-REN-04 **se mantiene**; **0 cambios** en UD-1; el p95 de INT-19 varía **≤ 10 %** | **Nuevo (H-04)** |

### 3.6 Mantenibilidad

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Origen |
|---|---|---|---|---|---|---|
| RNF-MAN-01 | Equipo de diseño | Se agrega una política de reposición o de selección de lotes | Diseño vigente | Solo se agrega una estrategia (`EstrategiaReposicion`, `PoliticaSeleccionLotes`) | **0 cambios** en el flujo principal | Semilla |
| RNF-MAN-02 | Revisor del PR | Revisa un diagrama C4 de contenedores o de componentes | Cualquier sprint | Ningún módulo accede al esquema de otro | **0 dependencias de datos cruzadas** | Semilla |
| RNF-MAN-03 | Negocio | Se cambia de ERP o se agrega un transportista | Diseño vigente | Solo se reemplaza o se agrega un adaptador (`AdaptadorErp`, `AdaptadorTransportistaPropio`, `AdaptadorTransportistaExterno`) | **0 cambios** en los componentes de dominio de SVC-03 y SVC-05 y en sus secuencias | Nuevo |
| RNF-MAN-04 | Equipo de arquitectura | Se extrae SVC-02 a su propia unidad de despliegue (§2.4) | Primera extracción | Solo cambia `AdaptadorReservasInventario`; el puerto `GestionReservas` se mantiene (PRI-09) | **0 cambios** en SEC-FA-recepcion, SEC-FB-reposicion y en la secuencia del flujo C | Nuevo |

---

## 4. Trazabilidad: vínculo y mecanismo (CA3)

| RNF | SVC / INT / VL | Lo hace posible (patrón · ADR · puerto) | Dónde se ve en el diseño | FAL / riesgo |
|---|---|---|---|---|
| RNF-REN-01 | SVC-02 · INT-15 · VL-02 | `ConsultaExistencias` (ISP) · ADR-002 | C4 N3 de SVC-02 (HU-14) | — |
| RNF-REN-02 | SVC-02, SVC-04 · INT-19 | `GestionReservas`, `AdaptadorReservasInventario` (Circuit Breaker) · ADR-003 | Secuencia del flujo C (HU-18) | FAL-02 · RIE-02, RIE-04 |
| RNF-REN-03 | SVC-02, TRX-04 · INT-07, INT-08 · VL-01 | Outbox + CQRS · ADR-004, ADR-005 | SEC-FA-recepcion (mensajes 21 a 25) | RIE-03, RIE-05 |
| RNF-REN-04 | SVC-05, TRX-04 · INT-23 · VL-04 | Canal de señales separado (EVT-32) · ADR-007 | C4 N3 de SVC-05 (HU-19) | FAL-03 |
| RNF-REN-05 | SVC-03, TRX-04 · INT-10, INT-11 · VL-02 | `EvaluadorRiesgoAgotamiento` · ADR-006, ADR-005 | SEC-FB-reposicion (mensajes 3 a 17) | H-01 · RIE-05 |
| RNF-REN-06 | SVC-03 · INT-12 · VL-02 | `EstadoSugerencia` (State) · TX-04 | SEC-FB-reposicion (mensajes 18 a 36) | — |
| RNF-REN-07 | SVC-02 · INT-06 | `RegistroMovimientos`, `ReplicaCatalogo` · ADR-009 | SEC-FA-recepcion (mensajes 4 a 20) | RIE-06 |
| RNF-DIS-01 | SVC-02, TRX-03 · INT-07 | `PublicadorOutbox`, `ConsumidorIdempotente` · ADR-004 | SEC-FA-recepcion (A2) | FAL-01 · RIE-03 |
| RNF-DIS-02 | SVC-04 · INT-19 | Retry idempotente + Circuit Breaker · ADR-003 | Secuencia del flujo C (HU-18) | FAL-02 · RIE-02 |
| RNF-DIS-03 | SVC-05 · INT-23 · VL-04 | `PuertoControladorAmr` · ADR-007 | C4 N3 de SVC-05 (HU-19) | FAL-03 |
| RNF-DIS-04 | UD-1 (SVC-01 a SVC-04) | Monolito modular · ADR-001 | C4 N2 (HU-12) | RIE-02 |
| RNF-DIS-05 | SVC-03, SVC-06 · INT-16 | `EstrategiaReposicion` (Strategy) · ADR-006 | SEC-FB-reposicion (B5) | FAL-04 |
| RNF-DIS-06 | SVC-03, EXT-01 · INT-14 · VL-02 | `PuertoErp` → `AdaptadorErp` (Adapter, reintentos) | SEC-FB-reposicion (B4) | FAL-05 · RIE-07 |
| RNF-DIS-07 | TRX-04 · INT-03, INT-15 · VL-01 a VL-07 | CQRS reconstruible, `ActualizadorVista` · ADR-005 | Informe de patrones y ADR-005 (HU-30) | FAL-06 · RIE-05 |
| RNF-DIS-08 | TRX-02, EXT-04 · INT-02 | Identidad centralizada (§11.4) | C4 N1 y C4 N2 (HU-11, HU-12) | FAL-07 |
| RNF-DIS-09 | TRX-03 y todos los consumidores | `ConsumidorIdempotente` + DLQ · ADR-004 | Los C4 N3 de cada módulo | FAL-08 · RIE-03 |
| RNF-DIS-10 | SVC-05, EXT-02 · INT-27, INT-33 · VL-05 | `PuertoTransportista` (Adapter) · ADR-007 | Secuencia del flujo C (HU-18) | FAL-09 · RIE-07 |
| RNF-CON-01 | SVC-02 · INT-19 | TX-02 · ADR-003 | C4 N3 de SVC-02 (HU-14) | RIE-04 |
| RNF-CON-02 | TRX-04, TRX-01 · VL-01 a VL-07 | Indicador de frescura (§8.2) · ADR-005 | Dashboard v1 y v2 (HU-22, HU-32) | RIE-05 |
| RNF-CON-03 | Todos los SVC · TX-01 a TX-06 | Outbox (PRI-06) · ADR-004 | SEC-FA-recepcion (TX-01), SEC-FB-reposicion (TX-04) | RIE-03 |
| RNF-CON-04 | SVC-01 a SVC-04 · INT-04, INT-10, INT-31 | Réplicas locales (PRI-04) · ADR-009 | SEC-FA-recepcion (A4) | RIE-06 · PC-01, PC-05 |
| RNF-CON-05 | SVC-03 · INT-10, INT-12 · VL-02 | `EstadoSugerencia` (State) · ADR-006 | SEC-FB-reposicion (B2) | PC-06, PC-08 |
| RNF-SEG-01 | Todos los SVC · OPS-01 a OPS-08 | Autorización en la capacidad dueña (§11.4) | Seguridad y roles (HU-28) | RIE-08 |
| RNF-SEG-02 | Repositorio | Checklist del PR (`CONTRIBUTING.md`) | Plantilla de revisión de PR | RIE-08 |
| RNF-SEG-03 | EXT-04 · OPS-07 · ROL-01, ROL-02, ROL-03 | Separación de funciones (§11.4) | Seguridad y roles (HU-28) | H-05 · RIE-08 |
| RNF-SEG-04 | SVC-02, SVC-04 · INT-19 | Identidad entre módulos (§11.4) · PRI-09 | C4 N2 (HU-12) | RIE-08 |
| RNF-ESC-01 | TRX-04, SVC-06, UD-2, UD-4 | Consumidores independientes · ADR-005, ADR-007 | Disponibilidad y escalabilidad (HU-29) | — |
| RNF-ESC-02 | UD-1 · INT-19 | Ruta de evolución (§2.4) · ADR-001 | C4 N2 (HU-12) | H-04 · RIE-02 |
| RNF-ESC-03 | UD-2 (SVC-05) | Unidad de despliegue separada · ADR-007 | C4 N2 (HU-12) y C4 N3 de SVC-05 (HU-19) | H-04 |
| RNF-MAN-01 | SVC-02, SVC-03 | Strategy (OCP) | C4 N3 de SVC-03 (HU-15); evidencia SOLID (HU-31) | — |
| RNF-MAN-02 | Todos los SVC | Esquema por módulo (PRI-02) · ADR-002 | Todos los C4 N2 y N3 | RIE-01 |
| RNF-MAN-03 | SVC-03, SVC-05 · INT-14, INT-27 | Adapter (LSP, DIP) | C4 N3 de SVC-03 (HU-15) y de SVC-05 (HU-19) | RIE-07 |
| RNF-MAN-04 | SVC-02, SVC-04 · INT-19 | Puertos y adaptadores (DIP) · PRI-09 · ADR-001 | C4 N3 de SVC-02 (HU-14) | RIE-01 |

---

## 5. Cambios respecto de la semilla y propuestas (CA4)

### 5.1 Los 15 RNF semilla

Se **conservan sin cambios de texto** (RNF-REN-01 a 04, RNF-DIS-01 a 04, RNF-CON-01 y 02, RNF-SEG-01 y 02, RNF-ESC-01, RNF-MAN-01 y 02). Solo se reescribieron con el formato de escenario de §3. Hay dos aclaraciones que no cambian su texto:

| RNF | Aclaración | Por qué |
|---|---|---|
| RNF-DIS-02 | En el MVP, SVC-02 y SVC-04 viven en UD-1, así que "SVC-02 no responde" significa que se excede el tiempo límite del puerto `GestionReservas`. Se cumple completo después de extraer SVC-02 (§2.4) | H-03; se desarrolla en HU-29 (P1) |
| RNF-DIS-04 | "Horario operativo" se mide con SUP-04 (propuesto) | H-04 |

### 5.2 RNF nuevos (18)

RNF-REN-05 a 07 · RNF-DIS-05 a 10 · RNF-CON-03 a 05 · RNF-SEG-03 y 04 · RNF-ESC-02 y 03 · RNF-MAN-03 y 04.

### 5.3 Propuestas de cambio al contrato («PROPUESTA DE CAMBIO», no oficiales)

La numeración continúa la de HU-16 y HU-17 (PC-01 a PC-08). Ninguna se usa como oficial hasta aprobarse por ARQUITECTURA_BASE §0.3.

| ID | Propuesta | Por qué | Tipo | Dueño |
|---|---|---|---|---|
| PC-09 | Ajustar **CE-03 de ≤ 5 s a ≤ 3 s**, para que CE-03 + CE-01 quepan en los 5 s que pide HN-09 (RNF-REN-05) | Hoy suman unos 10 s (H-01). La evaluación de SVC-03 es local, sobre una réplica, con unos 2 eventos/s: 3 s es holgado. *Alternativa descartada:* relajar HN-09 a ≤ 10 s, que cambia un criterio que el PO ya validó | MINOR | P1 (§7.3), P2 (HN-09) |
| PC-10 | Agregar **SUP-04** (horario operativo) a §11.1 | RNF-DIS-04 no se puede medir sin él (H-04) | MINOR | P1, P2 |
| PC-11 | Registrar los **18 RNF nuevos** en §11.2 y completar la columna FAL → RNF de §11.3 | Mantener el contrato como fuente única de IDs (§0.1) | MINOR | P1 |

---

## 6. Hallazgos y riesgos que se cierran o mitigan

| Hallazgo o riesgo | Cómo queda |
|---|---|
| H-01 · Latencia de la alerta del flujo B | Cubierto por RNF-REN-05 y PC-09 |
| H-02 · FAL-04 a FAL-09 sin RNF | Cubierto: RNF-DIS-05 a 10 (un RNF por cada fallo) |
| H-04 · Escalabilidad y horario operativo | Cubierto por RNF-ESC-02, RNF-ESC-03 y SUP-04 (PC-10) |
| H-05 · Separación de funciones | Requisito definido en RNF-SEG-03; los pares de roles se completan en HU-28 (P2) |
| RIE-05 · Frescura del dashboard | Pasa a **En mitigación** (RNF-REN-05, RNF-CON-02) |
| RIE-16 · Requisitos de calidad incompletos | Pasa a **En mitigación** (CA1 a CA4 de esta HU) |

**Cobertura final de FAL → RNF:** FAL-01 → RNF-DIS-01 · FAL-02 → RNF-DIS-02 · FAL-03 → RNF-DIS-03 · FAL-04 → RNF-DIS-05 · FAL-05 → RNF-DIS-06 · FAL-06 → RNF-DIS-07 · FAL-07 → RNF-DIS-08 · FAL-08 → RNF-DIS-09 · FAL-09 → RNF-DIS-10. **Todos los fallos tienen ahora un requisito verificable.**

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-24 | P5 | Versión inicial: 33 RNF en formato de escenario (15 semilla y 18 nuevos), SUP validados, SUP-04 propuesto, trazabilidad y PC-09 a PC-11 | HU-27 |
| 1.0.1 | 2026-11-27 | P5 | Pasa a `docs/arquitectura/requisitos-calidad.md`, la ubicación que pide BACKLOG para HU-27, y reemplaza la versión de suplencia del PR #83, que repetía IDs (RNF-CON-03, RNF-ESC-02, RNF-MAN-02, RNF-SEG-03…) con otro significado. Se actualizan el estado y los enlaces. Sin cambios de contenido | HU-27 |
