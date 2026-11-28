[requisitos_de_calidad_verificables.md](https://github.com/user-attachments/files/35535486/requisitos_de_calidad_verificables.md)
# Smart Inventory · Requisitos de Calidad Verificables

| Campo | Valor |
|---|---|
| Documento | `docs/arquitectura/requisitos-calidad.md` |
| Versión | **1.0.0** |
| Estado | Propuesto para revisión (Sprint 3 · HU-27) |
| Responsable | P5 · Diseñador de procesos y calidad (asumida en suplencia) |
| Revisor | P4 · Diseñador de dominio |
| Fecha | 2026-11-27 |
| Contrato de referencia | [`docs/ARQUITECTURA_BASE.md`](../ARQUITECTURA_BASE.md) v1.0.0 (§2, §7, §8, §10, §11, §12) |
| Historia vinculada | HU-27 |

---

## 1. Validación y justificación de supuestos de carga (`SUP-01` a `SUP-03`)

Los umbrales cuantitativos de latencia, concurrencia, disponibilidad y escalamiento en este documento se validan frente a los supuestos operativos del contrato (`ARQUITECTURA_BASE.md` §11.1):

| ID | Supuesto del contrato | Validación / ajuste para HU-27 | Justificación de umbrales en los RNF |
|---|---|---|---|
| **SUP-01** | Un centro de distribución, 4 muelles, 20 AMR, ~5.000 SKU activos y ~30.000 combinaciones SKU · ubicación · lote. | **Validado.** | Un catálogo de ~5.000 SKU y ~30.000 registros SKU · ubicación · lote permite consultas indexadas por lote/FEFO sub-segundo en `SVC-02` y un espacio de estados acotado para 4 muelles y 20 AMR en `SVC-05`. |
| **SUP-02** | **Carga normal:** 60 usuarios concurrentes, 1.500 movimientos/hora (~0,42 mov/s), 300 pedidos/hora (~0,083 pedidos/s), telemetría de 20 AMR a 1 muestra por segundo (20 muestras/s). | **Validado.** | Con ~0,42 mov/s y ~0,083 pedidos/s promedio, las transacciones locales (`TX-01` a `TX-06`) y las llamadas síncronas (`INT-15`, `INT-19`) no saturan `UD-1`, mientras que las 20 muestras/s de telemetría justifican separar `SVC-05` (`UD-2`) y muestrear `EVT-32`. |
| **SUP-03** | **Carga pico:** 3× la carga normal en temporadas altas (180 usuarios concurrentes, 4.500 movimientos/hora, 900 pedidos/hora, 20 AMR a 1 muestra/s). | **Validado.** | Un factor de 3× (1,25 mov/s y 0,25 pedidos/s sostenidos, con ráfagas de lectura y reserva) justifica tolerar hasta 2 s p95 en `INT-19` bajo carga pico y activar escalamiento horizontal en `TRX-04`, consumidores de `SVC-06`, `UD-2` y `UD-4` (`RNF-ESC-01`). |

---

## 2. Conservación de requisitos semilla (`ARQUITECTURA_BASE.md` §11.2)

Se conservan los **15 RNF semilla** definidos en `ARQUITECTURA_BASE.md` §11.2 (`RNF-REN-01` a `RNF-REN-04`, `RNF-DIS-01` a `RNF-DIS-04`, `RNF-CON-01` a `RNF-CON-02`, `RNF-SEG-01` a `RNF-SEG-02`, `RNF-ESC-01`, `RNF-MAN-01` a `RNF-MAN-02`) y se incorporan **RNF complementarios** en atributos con un solo requisito semilla o donde conviene explicitar umbrales adicionales (como `RNF-ESC-02`, `RNF-CON-03` y `RNF-SEG-03`), sin alterar los identificadores ni el sentido del contrato base.

---

## 3. Requisitos de calidad por atributo

### 3.1 Seguridad

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Vínculo (SVC/INT/VL) | Patrón/ADR |
|---|---|---|---|---|---|---|---|
| **RNF-SEG-01** | Usuario interno o sistema (`ROL-01` a `ROL-07`) | Solicitud de ejecución de una operación sensible (`OPS-01` a `OPS-08`) | Operación normal o pico (`SUP-02`, `SUP-03`) en `UD-1` o `UD-2` | La capacidad dueña valida el rol autorizado y registra la auditoría (quién, cuándo, qué, motivo) | 100 % de las operaciones sensibles verificadas en el dueño y auditadas sin omisiones | `SVC-02`, `SVC-03`, `SVC-04`, `SVC-05`, `INT-02` | Puertos y adaptadores; ADR-002 |
| **RNF-SEG-02** | Integrante del equipo o pipeline de documentación | Inclusión o actualización de archivos en el repositorio de diseño | Revisión de Pull Request (`BACKLOG.md` §6.3) | Se inspecciona el PR para impedir credenciales, claves o secretos en código o diagramas | 0 secretos en el repositorio en el 100 % de los PR aprobados | `TRX-02`, `EXT-04` | Revisión cruzada obligatoria; ADR-002 |
| **RNF-SEG-03** | Usuario o módulo invocante con token/identidad de módulo | Intento de ejecutar operación sin rol requerido o acceso a esquema ajeno | Operación normal o pico (`SUP-02`, `SUP-03`) | Se rechaza la operación de negocio y se bloquea el acceso directo entre esquemas | 100 % de intentos no autorizados rechazados y 0 accesos cruzados a datos | `SVC-01` a `SVC-05`, `INT-02`, `INT-06`, `INT-12` | PRI-02; ADR-002 |

### 3.2 Disponibilidad

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Vínculo (SVC/INT/VL) | Patrón/ADR |
|---|---|---|---|---|---|---|---|
| **RNF-DIS-01** | Broker de eventos (`TRX-03`) | Caída o indisponibilidad temporal del broker (`FAL-01`) | Carga normal o pico (`SUP-02`, `SUP-03`) | `SVC-02` (y demás módulos) sigue aceptando recepciones y reservas persistiendo en *Outbox*; al recuperarse el broker se publica todo | 100 % de eventos pendientes publicados sin pérdida ni efectos duplicados al restablecerse `TRX-03` | `SVC-02`, `INT-07`, `VL-01` | Outbox e idempotencia (PRI-06); ADR-004 |
| **RNF-DIS-02** | `SVC-02 Inventario` | Falta de respuesta o indisponibilidad de `SVC-02` ante solicitud de reserva (`FAL-02`) | Operación normal o pico (`SUP-02`, `SUP-03`) | `SVC-04` marca el pedido en *PendienteReserva* sin perderlo y ejecuta reintentos idempotentes | 0 pedidos perdidos; reintentos automáticos activos hasta resolver la reserva | `SVC-04`, `SVC-02`, `INT-19`, `VL-03` | Timeout, Circuit Breaker y Retry idempotente; ADR-003, ADR-004 |
| **RNF-DIS-03** | Telemetría AMR o controlador `EXT-03` | Pérdida de señal de telemetría de un AMR por más de 10 s (`FAL-03`) | Operación de flota con hasta 20 AMR (`SUP-01`, `SUP-02`) | `VL-04` marca el AMR "sin señal", deja de recibir misiones y sus misiones pasan a *Fallida* para reasignación o picking manual, sin afectar inventario ni pedidos | Detección en `> 10 s` sin señal; 0 impactos en saldos ni pedidos | `SVC-05`, `INT-22`, `INT-23`, `VL-04` | Puertos y adaptadores (`PuertoControladorAmr`); ADR-007 |
| **RNF-DIS-04** | Entorno de ejecución de `UD-1` | Operación continua del núcleo transaccional (`SVC-01` a `SVC-04`) | Horario operativo mensual (`SUP-01` a `SUP-03`) | `UD-1` permanece disponible y respondiendo comandos y consultas críticas | Disponibilidad `>= 99,5 %` del horario operativo mensual | `SVC-01`, `SVC-02`, `SVC-03`, `SVC-04`, `INT-06`, `INT-19` | Monolito modular evolutivo; ADR-001 |

### 3.3 Rendimiento

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Vínculo (SVC/INT/VL) | Patrón/ADR |
|---|---|---|---|---|---|---|---|
| **RNF-REN-01** | Usuario (`ROL-01`, `ROL-02`, `ROL-03`, `ROL-05`) | Consulta de detalle de existencias por lote para un SKU | Carga normal (`SUP-02`: 60 usuarios, ~5.000 SKU, ~30.000 registros SKU·ubicación·lote) | `SVC-02` retorna saldo físico, reservado y disponible por lote y ubicación | Tiempo de respuesta `<= 500 ms` p95 | `SVC-02`, `INT-15` | CQRS / consulta directa al dueño; ADR-002, ADR-005 |
| **RNF-REN-02** | `SVC-04 Pedidos y Fulfillment` | Solicitud de reserva de stock con regla FEFO para un pedido | Carga normal (`SUP-02`: 300 pedidos/h) y pico (`SUP-03`: 900 pedidos/h) | `SVC-02` evalúa disponibilidad FEFO y confirma o rechaza la reserva de forma atómica (`TX-02`) | `<= 1 s` p95 en carga normal; `<= 2 s` p95 en carga pico | `SVC-02`, `SVC-04`, `INT-19` | Strategy (`PoliticaFEFO`) y transacción local; ADR-003, ADR-004 |
| **RNF-REN-03** | Operador de bodega (`ROL-01`) | Registro de una recepción válida de mercadería (`TX-01` + `EVT-04`/`EVT-05`) | Carga normal (`SUP-02`: 1.500 movimientos/h) | `TRX-04` procesa los eventos y actualiza el indicador de unidades disponibles en el dashboard | `>= 95 %` de las recepciones reflejadas en `VL-01` en `< 5 s` | `SVC-02`, `INT-07`, `INT-08`, `VL-01` | Outbox + Proyección de lectura CQRS; ADR-004, ADR-005 |
| **RNF-REN-04** | AMR en operación (`EXT-03` → `SVC-05`) | Emisión de muestra de posición/estado de flota (`EVT-32`) | Hasta 20 AMR a 1 muestra/s (`SUP-01`, `SUP-02`) | `SVC-05` publica la señal muestreada y `TRX-04` actualiza la posición del AMR en el mapa de planta interna | Retraso de actualización en `VL-04` `<= 2 s` p95 | `SVC-05`, `INT-23`, `INT-32`, `VL-04` | Señal muestreada en canal separado; ADR-005, ADR-007 |

### 3.4 Escalabilidad

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Vínculo (SVC/INT/VL) | Patrón/ADR |
|---|---|---|---|---|---|---|---|
| **RNF-ESC-01** | Broker (`TRX-03`), carga de usuarios o flota | Aumento del retraso de consumo de eventos (`> 5 s` por 2 min), crecimiento de AMR activos o más usuarios conectados (`SUP-03`) | Carga pico (`SUP-03`) o expansión operativa | `TRX-04` y consumidores de `SVC-06` escalan horizontalmente; `UD-2` escala con AMR activos y `UD-4` con usuarios conectados | Escalamiento horizontal activo cuando retraso `> 5 s` durante `2 min`, sosteniendo carga pico (`3×` normal) | `TRX-04`, `SVC-05`, `SVC-06`, `INT-01`, `INT-05`, `INT-08`, `VL-01` a `VL-07` | Consumidores idempotentes y separación de UD; ADR-001, ADR-005, ADR-007 |
| **RNF-ESC-02** | Flujo de pedidos (`SVC-04`) o consultas de lectura (`TRX-04`) | Incremento de pedidos/hora hasta `3×` la carga normal (`SUP-03`) | Temporadas de alta demanda (`SUP-03`) | Las lecturas de supervisión se sirven desde `TRX-04` sin saturar las escrituras transaccionales de `UD-1` | Soporta hasta 900 pedidos/h y 4.500 movimientos/h manteniendo los tiempos de `RNF-REN-02` y `RNF-REN-03` | `SVC-02`, `SVC-04`, `TRX-04`, `INT-03`, `INT-19`, `VL-01`, `VL-03` | CQRS y réplicas locales; ADR-001, ADR-005, ADR-009 |

### 3.5 Consistencia

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Vínculo (SVC/INT/VL) | Patrón/ADR |
|---|---|---|---|---|---|---|---|
| **RNF-CON-01** | `SVC-04` o solicitudes concurrentes de reserva (`INT-19`) | Solicitudes de reserva FEFO sobre lotes de un SKU | Carga normal o pico (`SUP-02`, `SUP-03`) con concurrencia | `SVC-02` ejecuta `TX-02` verificando `disponible >= cantidad` sobre lotes no vencidos y no en cuarentena | `0` reservas que superen el disponible; saldo disponible nunca `< 0` | `SVC-02`, `INT-19`, `VL-01` | Transacción local y control de concurrencia; ADR-002, ADR-003 |
| **RNF-CON-02** | `TRX-04 Proyección de lectura` | Transcurso del tiempo desde el último evento aplicado a una vista | Operación normal o pico (`SUP-02`, `SUP-03`) o ante retraso (`FAL-01`, `FAL-06`) | Cada vista muestra "Actualizado hace X s" y cambia a *Con retraso* (`> objetivo` y `<= 60 s`) o *Desactualizada* (`> 60 s`) con banner rojo | 100 % de vistas muestran antigüedad; aviso de desactualización activado cuando retraso `> 60 s` | `TRX-04`, `INT-03`, `VL-01` a `VL-07` | Consistencia eventual informada; ADR-005 |
| **RNF-CON-03** | `SVC-04` al cancelar un pedido (`EVT-28`) | Cancelación de un pedido en estado previo a *EnDespacho* | Operación normal o pico (`SUP-02`, `SUP-03`) | `SVC-02` consume `EVT-28` y libera la reserva (`EVT-10`), restaurando el saldo disponible de forma idempotente | Reserva liberada y disponible recuperado en `<= 60 s` (`CE-05`), con `0` liberaciones duplicadas | `SVC-04`, `SVC-02`, `INT-31`, `VL-01`, `VL-03` | Coreografía con compensación e idempotencia; ADR-004, ADR-006 |

### 3.6 Mantenibilidad

| ID | Fuente | Estímulo | Entorno | Respuesta | Medida | Vínculo (SVC/INT/VL) | Patrón/ADR |
|---|---|---|---|---|---|---|---|
| **RNF-MAN-01** | Diseñador de software o dominio (`P4`, `P5`, `P7`) | Incorporación de una nueva política de reposición o estrategia de selección de lotes | Evolución del diseño en el repositorio de arquitectura | Se agrega una nueva implementación de `EstrategiaReposicion` o `PoliticaSeleccionLotes` sin modificar el flujo principal | 0 modificaciones al flujo principal del núcleo al agregar una estrategia | `SVC-02`, `SVC-03`, `INT-10`, `INT-19` | Strategy y Open/Closed Principle (OCP); ADR-003, ADR-006 |
| **RNF-MAN-02** | Diseñadores de arquitectura (`P1` a `P7`) | Modelado o actualización de diagramas C4 de contenedores (`C4N2`) o componentes (`C4N3`) | Revisión de diseño y checklist de PR (`BACKLOG.md` §6.3) | Cada módulo accede únicamente a su propio esquema lógico y se comunica con otros mediante puertos o eventos | `0` dependencias de datos cruzadas en diagramas C4 N2/N3 aprobados | `SVC-01` a `SVC-06`, `TRX-04`, `TRX-05`, `INT-04`, `INT-16` | Aislamiento por esquema y réplicas locales (PRI-02, PRI-04); ADR-002, ADR-009 |

---

## 4. Trazabilidad resumida hacia `ARQUITECTURA_BASE.md` §11.2

| RNF semilla | Estado en este documento | Observación |
|---|---|---|
| `RNF-REN-01` a `RNF-REN-04` | Conservados | Expresados con fuente, estímulo, entorno, respuesta y medida, validados con `SUP-01` a `SUP-03`. |
| `RNF-DIS-01` a `RNF-DIS-04` | Conservados | Vinculados a `FAL-01` a `FAL-03`, `UD-1`, `INT-06`, `INT-07`, `INT-19`, `INT-22`, `INT-23` y `ADR-001`, `ADR-003`, `ADR-004`, `ADR-007`. |
| `RNF-CON-01` a `RNF-CON-02` | Conservados + ampliados con `RNF-CON-03` | Cubren invariante de saldo no negativo, indicador de frescura de vistas y compensación por cancelación (`CE-05`). |
| `RNF-SEG-01` a `RNF-SEG-02` | Conservados + ampliados con `RNF-SEG-03` | Cubren autorización/auditoría en el dueño, ausencia de secretos en el repositorio y rechazo de accesos indebidos. |
| `RNF-ESC-01` | Conservado + ampliado con `RNF-ESC-02` | Detallan escalamiento horizontal por retraso (`> 5 s` por `2 min`), AMR activos, usuarios y factor `3×` de `SUP-03`. |
| `RNF-MAN-01` a `RNF-MAN-02` | Conservados | Evidencian extensibilidad por estrategias (`Strategy`/`OCP`) y aislamiento estricto de esquemas (`PRI-02`, `ADR-002`). |
