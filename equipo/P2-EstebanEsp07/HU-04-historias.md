# Smart Inventory · Historias de negocio HN-01 a HN-24

| Campo | Valor |
|---|---|
| Documento | `equipo/P2-EstebanEsp07/HU-04-historias.md` (insumo de `docs/BACKLOG.md` §2.2) |
| Versión | **1.0.0** |
| Estado | Propuesto para revisión (PR de HU-04) |
| Autor | P2 · Product Owner (`EstebanEsp07`) |
| Revisor | P3 · Scrum Master y gestor de versiones (`docs/BACKLOG.md` §1.2) |
| Fecha | 2026-11-23 |
| HU | **HU-04** Épicas, historias de negocio y priorización · Sprint 1 · rúbrica: Gestión del proyecto y sprints |
| Épicas | EP-01 a EP-08 (`HU-03-epicas.md`) |
| Contrato de referencia | `docs/ARQUITECTURA_BASE.md` v1.0.0 y `docs/BACKLOG.md` v1.0.0 |

> Este documento reproduce la tabla de `docs/BACKLOG.md` §2.2 (IDs, historias, prioridad, orden, módulo y HU de diseño) y reescribe cada criterio de aceptación en formato **Dado / Cuando / Entonces**. Donde el BACKLOG no traía alguna de las tres partes, se añadió una precondición o disparador neutro que se deduce de la historia o del contrato; el resultado esperado no cambia. Prioridad MoSCoW: **M** = Must (MVP) · **S** = Should · **C** = Could.

---

## 1. Product backlog ordenado

| Orden | HN | Épica | Módulo | Rol | Prior. | Diseñada en |
|---|---|---|---|---|---|---|
| 1 | HN-01 | EP-01 | SVC-01 | ROL-07 | M | HU-07, HU-13 |
| 2 | HN-02 | EP-01 | SVC-01 | ROL-07 | M | HU-07, HU-13 |
| 3 | HN-24 | EP-08 | EXT-04, todos los SVC | ROL-07 | M | HU-28 |
| 4 | HN-03 | EP-02 | SVC-02 | ROL-01 | M | HU-14, HU-16 |
| 5 | HN-04 | EP-02 | SVC-02 | ROL-01 | M | HU-14, HU-24 |
| 6 | HN-08 | EP-03 | SVC-03 | ROL-03 | M | HU-15, HU-17 |
| 7 | HN-09 | EP-03 | SVC-03 | ROL-03 | M | HU-17, HU-22 |
| 8 | HN-10 | EP-03 | SVC-03 | ROL-03 | M | HU-15, HU-17 |
| 9 | HN-11 | EP-03 | SVC-03 | ROL-03 | M | HU-17, HU-25 |
| 10 | HN-12 | EP-03 | SVC-03, SVC-02 | ROL-01 | M | HU-16, HU-25 |
| 11 | HN-20 | EP-06 | TRX-01, TRX-02, TRX-04 | ROL-05 | M | HU-22, HU-32 |
| 12 | HN-21 | EP-06 | TRX-01, TRX-02, TRX-04 | ROL-05 | M | HU-22, HU-32 |
| 13 | HN-13 | EP-04 | SVC-04, SVC-02 | ROL-05 | M | HU-14, HU-18 |
| 14 | HN-14 | EP-04 | SVC-04 | ROL-05 | M | HU-18, HU-26 |
| 15 | HN-15 | EP-04 | SVC-04 | ROL-01 | M | HU-18, HU-26 |
| 16 | HN-17 | EP-05 | SVC-05 | ROL-04 | M | HU-19, HU-26 |
| 17 | HN-18 | EP-05 | SVC-05 | ROL-04 | M | HU-19, HU-26 |
| 18 | HN-19 | EP-05 | SVC-05 | ROL-05 | M | HU-18, HU-24 |
| 19 | HN-07 | EP-02 | SVC-02 | ROL-02 | M | HU-14, HU-25 |
| 20 | HN-16 | EP-04 | SVC-04 | ROL-05 | S | HU-18, HU-26 |
| 21 | HN-05 | EP-02 | SVC-02 | ROL-01 | S | HU-14, HU-24 |
| 22 | HN-06 | EP-02 | SVC-02 | ROL-02 | S | HU-14, HU-28 |
| 23 | HN-23 | EP-07 | SVC-06 | ROL-03 | C | HU-15, HU-31 |
| 24 | HN-22 | EP-07 | SVC-06, TRX-05 | ROL-06 | C | HU-12, HU-30 |

### Resumen por épica

| Épica | Must | Should | Could | Total |
|---|---|---|---|---|
| EP-01 | 2 | 0 | 0 | 2 |
| EP-02 | 3 | 2 | 0 | 5 |
| EP-03 | 5 | 0 | 0 | 5 |
| EP-04 | 3 | 1 | 0 | 4 |
| EP-05 | 3 | 0 | 0 | 3 |
| EP-06 | 2 | 0 | 0 | 2 |
| EP-07 | 0 | 0 | 2 | 2 |
| EP-08 | 1 | 0 | 0 | 1 |
| **Total** | **19** | **3** | **2** | **24** |

**MVP = las 19 HN con prioridad M.**

---

## 2. Historias de negocio

### EP-01 · Catálogo y ubicaciones (SVC-01)

#### HN-01 · Registrar productos
`EP-01` · `SVC-01` · `ROL-07` · Prioridad **M** · Orden 1 · Diseñada en HU-07, HU-13

**Como** Administrador **quiero** registrar productos con SKU, categoría, unidad y si controlan lote y vencimiento, **para** que cada recepción exija los datos correctos.

- **Dado** un SKU nuevo marcado «requiere vencimiento»
- **Cuando** se guarda
- **Entonces** queda activo, se emite EVT-01 ProductoRegistrado y toda recepción de ese SKU exigirá lote y fecha de vencimiento.

#### HN-02 · Definir bodegas, zonas y ubicaciones
`EP-01` · `SVC-01` · `ROL-07` · Prioridad **M** · Orden 2 · Diseñada en HU-07, HU-13

**Como** Administrador **quiero** definir bodegas, zonas y ubicaciones con capacidad y estado, **para** ubicar las existencias y medir la ocupación.

- **Dada** una ubicación bloqueada
- **Cuando** se intenta recibir en ella
- **Entonces** Inventario rechaza el registro.

### EP-02 · Inventario y lotes (SVC-02)

#### HN-03 · Registrar una recepción
`EP-02` · `SVC-02` · `ROL-01` · Prioridad **M** · Orden 4 · Diseñada en HU-14, HU-16

**Como** Operador de bodega **quiero** registrar una recepción con producto, lote, cantidad, ubicación y vencimiento, **para** que el saldo quede actualizado.

*Escenario 1: vencimiento obligatorio*
- **Dado** un SKU con vencimiento obligatorio
- **Cuando** falta la fecha de vencimiento
- **Entonces** la recepción se rechaza.

*Escenario 2: datos válidos*
- **Dado** una recepción con todos los datos válidos
- **Cuando** se registra
- **Entonces** el movimiento y el saldo se guardan juntos (TX-01) y se emiten EVT-04 RecepcionRegistrada y EVT-05 SaldoActualizado.

#### HN-04 · Consultar existencias
`EP-02` · `SVC-02` · `ROL-01` · Prioridad **M** · Orden 5 · Diseñada en HU-14, HU-24

**Como** Operador de bodega **quiero** consultar existencias por SKU, ubicación y lote con su vencimiento, **para** saber qué está realmente disponible.

- **Dado** un SKU con existencias en uno o más lotes
- **Cuando** se consulta ese SKU
- **Entonces** se muestran por lote el saldo físico, reservado y disponible, y la respuesta cumple RNF-REN-01.

#### HN-05 · Transferir existencias
`EP-02` · `SVC-02` · `ROL-01` · Prioridad **S** · Orden 21 · Diseñada en HU-14, HU-24

**Como** Operador de bodega **quiero** transferir existencias entre ubicaciones, **para** reorganizar la bodega.

- **Dado** un SKU con existencias en una ubicación de origen
- **Cuando** se transfiere una cantidad a otra ubicación
- **Entonces** el saldo total del SKU no cambia, queda un movimiento con origen y destino y se emite EVT-06 TransferenciaRegistrada.

#### HN-06 · Registrar ajustes con motivo
`EP-02` · `SVC-02` · `ROL-02` · Prioridad **S** · Orden 22 · Diseñada en HU-14, HU-28

**Como** Supervisor de bodega **quiero** registrar ajustes con motivo, **para** corregir diferencias de conteo con auditoría.

*Escenario 1: sin autorización*
- **Dado** un usuario sin ROL-02
- **Cuando** intenta registrar un ajuste
- **Entonces** el ajuste se rechaza.

*Escenario 2: con autorización*
- **Dado** un usuario con ROL-02
- **Cuando** registra un ajuste
- **Entonces** el motivo es obligatorio y el ajuste queda auditado (OPS-01, EVT-07 AjusteRegistrado).

#### HN-07 · Ver lotes próximos a vencer
`EP-02` · `SVC-02` · `ROL-02` · Prioridad **M** · Orden 19 · Diseñada en HU-14, HU-25

**Como** Supervisor de bodega **quiero** ver los lotes próximos a vencer, **para** priorizar su salida.

- **Dado** un lote que entra en el umbral de días configurado
- **Cuando** el proceso periódico de vencimientos de Inventario lo detecta
- **Entonces** se emite EVT-12 LoteProximoAVencer y el lote aparece en VL-02 y en el KPI «Lotes por vencer» de VL-01.

### EP-03 · Reposición y compras (SVC-03)

#### HN-08 · Configurar la política de reposición
`EP-03` · `SVC-03` · `ROL-03` · Prioridad **M** · Orden 6 · Diseñada en HU-15, HU-17

**Como** Comprador **quiero** configurar mínimo y punto de reorden por SKU, **para** que el sistema detecte el riesgo de agotamiento.

- **Dado** un SKU con una política de reposición vigente
- **Cuando** se cambia la política
- **Entonces** el cambio queda auditado (OPS-04) y se aplica en la siguiente evaluación.

#### HN-09 · Alertar cuando un SKU cae bajo su mínimo
`EP-03` · `SVC-03` · `ROL-03` · Prioridad **M** · Orden 7 · Diseñada en HU-17, HU-22

**Como** Comprador **quiero** una alerta cuando un SKU cae bajo su mínimo, **para** actuar antes del quiebre.

*Escenario 1: riesgo detectado*
- **Dado** un SKU con mínimo de 100
- **Cuando** el disponible baja a 90
- **Entonces** se emite EVT-14 RiesgoAgotamientoDetectado y la alerta aparece en VL-02 en ≤ 5 s.

*Escenario 2: riesgo resuelto*
- **Dada** una alerta activa para ese SKU
- **Cuando** el disponible vuelve a ≥ 100
- **Entonces** EVT-15 RiesgoAgotamientoResuelto retira la alerta.

#### HN-10 · Ver la sugerencia de reposición
`EP-03` · `SVC-03` · `ROL-03` · Prioridad **M** · Orden 8 · Diseñada en HU-15, HU-17

**Como** Comprador **quiero** ver una sugerencia de reposición con cantidad, proveedor y justificación, **para** decidir con datos.

- **Dada** una sugerencia vigente
- **Cuando** el comprador la consulta
- **Entonces** se muestran el mínimo, el disponible, la demanda considerada y el lead time, y solo existe una sugerencia vigente por SKU.

#### HN-11 · Aprobar o rechazar una sugerencia
`EP-03` · `SVC-03` · `ROL-03` · Prioridad **M** · Orden 9 · Diseñada en HU-17, HU-25

**Como** Comprador **quiero** aprobar o rechazar una sugerencia, **para** generar la orden de compra.

*Escenario 1: aprobación*
- **Dada** una sugerencia vigente en estado *Generada*
- **Cuando** el comprador la aprueba
- **Entonces** se crea la OC en estado *Emitida* y se emiten EVT-17 SugerenciaReposicionAprobada y EVT-19 OrdenCompraEmitida.

*Escenario 2: sugerencia reemplazada*
- **Dada** una sugerencia que ya fue reemplazada por otra
- **Cuando** el comprador intenta aprobarla
- **Entonces** la acción se rechaza con un mensaje claro.

#### HN-12 · Registrar la recepción contra una OC
`EP-03` · `SVC-03` · `ROL-01` · Prioridad **M** · Orden 10 · Diseñada en HU-16, HU-25

**Como** Operador de bodega **quiero** registrar la recepción contra una OC, **para** cerrar el ciclo de compra.

*Escenario 1: recepción completa*
- **Dada** una OC de 200 u.
- **Cuando** se reciben 200
- **Entonces** la OC pasa a *Cerrada* (EVT-20 OrdenCompraCerrada).

*Escenario 2: recepción parcial*
- **Dada** una OC de 200 u.
- **Cuando** se reciben 120
- **Entonces** la OC queda *RecibidaParcial*.

### EP-04 · Pedidos y fulfillment (SVC-04)

#### HN-13 · Reservar stock con FEFO
`EP-04` · `SVC-04`, `SVC-02` · `ROL-05` · Prioridad **M** · Orden 13 · Diseñada en HU-14, HU-18

**Como** Supervisor de operaciones **quiero** que al registrar un pedido se reserve stock con FEFO, **para** despachar primero lo que vence antes.

- **Dados** L1 (vence el 10-dic) y L2 (vence el 20-dic) disponibles
- **Cuando** se reserva stock para un pedido
- **Entonces** se asigna primero L1, y los lotes *Vencido* o *EnCuarentena* nunca se asignan.

#### HN-14 · Dejar en espera los pedidos sin stock
`EP-04` · `SVC-04` · `ROL-05` · Prioridad **M** · Orden 14 · Diseñada en HU-18, HU-26

**Como** Supervisor de operaciones **quiero** que los pedidos sin stock suficiente queden en espera, **para** gestionarlos sin perderlos.

- **Dado** un pedido registrado cuya cantidad supera el disponible
- **Cuando** se solicita la reserva y el disponible no alcanza
- **Entonces** el pedido pasa a *EnEspera* (EVT-23 PedidoEnEspera), Inventario emite EVT-09 ReservaRechazada y no se reserva nada parcialmente (MVP).

#### HN-15 · Preparar el pedido
`EP-04` · `SVC-04` · `ROL-01` · Prioridad **M** · Orden 15 · Diseñada en HU-18, HU-26

**Como** Operador de bodega **quiero** que se genere la tarea de preparación y confirmar su término, **para** pasar el pedido a despacho.

*Escenario 1: tarea de preparación*
- **Dado** un pedido con todo su stock reservado
- **Cuando** el pedido queda *Reservado*
- **Entonces** se emite EVT-24 PreparacionSolicitada.

*Escenario 2: preparación confirmada*
- **Dada** una tarea de preparación en curso
- **Cuando** el operador confirma su término
- **Entonces** el pedido pasa a *Preparado* (EVT-25 PreparacionCompletada) y se solicita el despacho (EVT-26 DespachoSolicitado).

#### HN-16 · Cancelar un pedido antes del despacho
`EP-04` · `SVC-04` · `ROL-05` · Prioridad **S** · Orden 20 · Diseñada en HU-18, HU-26

**Como** Supervisor de operaciones **quiero** cancelar un pedido antes del despacho, **para** liberar su reserva.

- **Dado** un pedido en estado *Reservado*, anterior a *EnDespacho*
- **Cuando** se cancela
- **Entonces** se emite EVT-28 PedidoCancelado y, en ≤ 60 s, Inventario libera la reserva (EVT-10 ReservaLiberada) y el disponible se recupera.

### EP-05 · Flota y operación (SVC-05)

#### HN-17 · Asignar tareas a un AMR
`EP-05` · `SVC-05` · `ROL-04` · Prioridad **M** · Orden 16 · Diseñada en HU-19, HU-26

**Como** Operador de patio y flota **quiero** que las tareas de preparación se asignen a un AMR disponible con su ruta, **para** automatizar el picking.

*Escenario 1: AMR disponible*
- **Dados** una tarea de preparación (EVT-24) y un AMR disponible
- **Cuando** se asigna la misión
- **Entonces** el AMR recibe la misión con su ruta y la asignación se informa con EVT-29 MisionAmrAsignada.

*Escenario 2: AMR no elegible*
- **Dado** un AMR sin señal por más de 10 s o *FueraDeServicio*
- **Cuando** se asignan misiones
- **Entonces** ese AMR no recibe misiones.

#### HN-18 · Registrar la llegada de camiones y asignar muelle
`EP-05` · `SVC-05` · `ROL-04` · Prioridad **M** · Orden 17 · Diseñada en HU-19, HU-26

**Como** Operador de patio y flota **quiero** registrar la llegada de camiones y asignarles muelle, **para** ordenar el patio.

*Escenario 1: llegada*
- **Dado** un camión que llega al acceso o a la zona de espera
- **Cuando** se registra su llegada
- **Entonces** aparece en VL-05 (EVT-33 VehiculoArribado).

*Escenario 2: muelle no asignable*
- **Dado** un muelle *Ocupado* o *FueraDeServicio*
- **Cuando** se intenta asignarlo a un camión
- **Entonces** la asignación no se realiza.

#### HN-19 · Cerrar el pedido al salir el camión
`EP-05` · `SVC-05`, `SVC-04`, `SVC-02` · `ROL-05` · Prioridad **M** · Orden 18 · Diseñada en HU-18, HU-24

**Como** Supervisor de operaciones **quiero** que al salir el camión cargado el pedido quede despachado y el stock descontado, **para** cerrar el ciclo.

- **Dado** un pedido en *EnDespacho* con su camión cargado
- **Cuando** se emite EVT-35 VehiculoDespachado
- **Entonces** el pedido pasa a *Despachado* (EVT-27 PedidoDespachado) e Inventario registra la salida consumiendo la reserva (EVT-11 SalidaRegistrada, EVT-05).

### EP-06 · Dashboard operativo (TRX-01, TRX-02, TRX-04)

#### HN-20 · Ver KPIs y alertas con la antigüedad del dato
`EP-06` · `TRX-01`, `TRX-02`, `TRX-04` · `ROL-05` · Prioridad **M** · Orden 11 · Diseñada en HU-22, HU-32

**Como** Supervisor de operaciones **quiero** ver KPIs y alertas en vivo con la antigüedad del dato, **para** decidir con información confiable.

*Escenario 1: antigüedad visible*
- **Dada** cualquier vista del dashboard
- **Cuando** el supervisor la consulta
- **Entonces** muestra «Actualizado hace X s».

*Escenario 2: retraso*
- **Dada** una vista cuyo retraso supera su objetivo de frescura
- **Cuando** el retraso es mayor al objetivo y ≤ 60 s, **entonces** pasa a *Con retraso*; **cuando** supera 60 s o se pierde la conexión, **entonces** pasa a *Desactualizada* (`ARQUITECTURA_BASE` §8.2).

#### HN-21 · Alternar entre planta interna y externa
`EP-06` · `TRX-01`, `TRX-02`, `TRX-04` · `ROL-05` · Prioridad **M** · Orden 12 · Diseñada en HU-22, HU-32

**Como** Supervisor de operaciones **quiero** alternar entre planta interna y planta externa, **para** supervisar racks y AMR, o muelles y camiones.

- **Dado** el dashboard abierto en un modo (Planta interna o Planta externa)
- **Cuando** el supervisor cambia de modo
- **Entonces** solo cambia el mapa central (VL-04 o VL-05) y se mantienen KPIs, alertas, flujo de pedidos y actividad (VL-01, VL-02, VL-03, VL-06).

### EP-07 · Analítica (SVC-06, TRX-05)

#### HN-22 · Consultar indicadores históricos
`EP-07` · `SVC-06`, `TRX-05` · `ROL-06` · Prioridad **C** · Orden 24 · Diseñada en HU-12, HU-30

**Como** Analista **quiero** consultar rotación, quiebres, vencimientos y puntualidad históricos, **para** mejorar las políticas.

- **Dado** el corte diario de indicadores
- **Cuando** el analista consulta VL-07
- **Entonces** VL-07 muestra los indicadores calculados desde el Lakehouse, que nunca se usan como saldo operativo.

#### HN-23 · Sugerencias con pronóstico de demanda
`EP-07` · `SVC-06`, `SVC-03` · `ROL-03` · Prioridad **C** · Orden 23 · Diseñada en HU-15, HU-31

**Como** Comprador **quiero** que las sugerencias consideren un pronóstico de demanda, **para** reducir quiebres y sobrestock.

*Escenario 1: con pronóstico*
- **Dado** un SKU para el que se calcula una sugerencia
- **Cuando** hay un pronóstico vigente (EVT-37 PronosticoDemandaActualizado)
- **Entonces** se usa `ReposicionPorPronostico`.

*Escenario 2: sin pronóstico*
- **Dado** un SKU para el que se calcula una sugerencia
- **Cuando** no hay pronóstico vigente
- **Entonces** se usa `ReposicionMinMax`, sin cambiar el flujo.

### EP-08 · Seguridad y acceso (EXT-04, todos)

#### HN-24 · Asignar roles
`EP-08` · `EXT-04`, todos los SVC · `ROL-07` · Prioridad **M** · Orden 3 · Diseñada en HU-28

**Como** Administrador **quiero** asignar roles a los usuarios, **para** que cada uno ejecute solo las operaciones permitidas.

- **Dado** un usuario con ROL-06
- **Cuando** intenta aprobar una sugerencia
- **Entonces** la capacidad dueña (SVC-03) lo rechaza y queda registrado.

---

## 3. Verificación de la HU-04

| CA (`docs/BACKLOG.md` §4.1) | Estado | Evidencia |
|---|---|---|
| CA1. EP-01 a EP-08 y al menos 24 HN en formato Como/Quiero/Para con criterios Dado/Cuando/Entonces | Cumplido | `HU-03-epicas.md` y §2 (24 HN) |
| CA2. Cada HN tiene módulo, prioridad MoSCoW, orden y la HU que la diseña | Cumplido | §1 y encabezado de cada HN |
| CA3. P4, P5 y P6 aportaron y revisaron las HN de sus épicas | Pendiente de evidencia | Registrar en el PR: P4 → EP-01 y EP-02 (HN-01 a HN-07); P5 → EP-03 (HN-08 a HN-12); P6 → EP-04 y EP-05 (HN-13 a HN-19) |
| CA4. El orden respeta dependencias | Cumplido | Catálogo y roles (órdenes 1 a 3) → inventario (desde el 4) → reposición (desde el 6) |

Comprobaciones de consistencia: 24 HN únicas (HN-01 a HN-24), 24 órdenes únicos (1 a 24), 19 HN de prioridad M, 3 S y 2 C.

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-23 | P2 | Versión inicial: 24 HN con criterios Dado/Cuando/Entonces | HU-04 |
