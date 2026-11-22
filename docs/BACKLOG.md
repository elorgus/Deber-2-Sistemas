# Smart Inventory · Backlog, equipo y plan de sprints

| Campo | Valor |
|---|---|
| Documento | `docs/BACKLOG.md` |
| Versión | **1.0.0** |
| Estado | Propuesto para aprobación del equipo (Sprint 1 · HU-04 y HU-05) |
| Dueños | P2 · Product Owner (§2) y P3 · Scrum Master (§1, §3 a §9) |
| Fecha | 2026-11-20 |
| Contrato de referencia | [`docs/ARQUITECTURA_BASE.md`](ARQUITECTURA_BASE.md) v1.0.0 |

> Todos los identificadores técnicos (`SVC`, `TRX`, `EXT`, `UD`, `EVT`, `INT`, `VL`, `RNF`, `ROL`, `OPS`, `ADR`) vienen del contrato. Este documento solo agrega los de gestión: **P** (integrante), **EP** (épica), **HN** (historia de negocio del producto) y **HU** (historia de trabajo del equipo dentro de un sprint).

---

## 0. Cómo usar este documento

- **§2 Product backlog:** qué necesita el negocio (HN). Lo prioriza P2.
- **§4 Sprint backlog:** qué artefactos produce el equipo en cada sprint (HU), con responsable, revisor y criterios de aceptación. Cada HU diseña una o varias HN.
- **§6 Calidad del trabajo:** Definition of Ready, Definition of Done y checklist de revisión cruzada.
- **§8 Estructura del repositorio:** dónde va cada cosa.
- **§9 Protocolo de IA:** qué darle a tu IA antes de trabajar.

Relación entre IDs: `EP` agrupa `HN` → cada `HN` se diseña en una o más `HU` → cada `HU` produce artefactos que usan `SVC`, `EVT`, `INT`, `VL`, `RNF` y `ADR` del contrato.

---

## 1. Equipo y responsabilidades

### 1.1 Integrantes

| ID | Nombre | Rol en el equipo | Rol del enunciado | Es dueño de | Carga (pts) |
|---|---|---|---|---|---|
| **P1** | *(completar)* | Líder de arquitectura | Líder de arquitectura | Contrato maestro, C4 N2, ADR de límites, disponibilidad y escalabilidad, documento de arquitectura consolidado, revisión de consistencia | 24 |
| **P2** | *(completar)* | Product Owner | Product Owner | Visión y MVP, épicas e historias de negocio, prioridades, propuesta del dashboard, roles y permisos, presentación final | 22 |
| **P3** | *(completar)* | Scrum Master y gestor de versiones | Scrum Master / facilitador | Repositorio, tablero, ceremonias, registros de sprint, política de versionamiento, release, verificación final | 17 + facilitación |
| **P4** | *(completar)* | Diseñador de dominio: Catálogo e Inventario | Equipo de diseño | SVC-01 y SVC-02, glosario y modelo de dominio, propiedad de datos, C4 N3 Inventario, estados de lote/reserva/sugerencia/OC, matriz de trazabilidad | 19 |
| **P5** | *(completar)* | Diseñador de procesos y calidad: Reposición | Equipo de diseño | SVC-03, flujos A y B (el flujo vertical), riesgos, requisitos de calidad, informe final | 20 |
| **P6** | *(completar)* | Diseñador de operación: Pedidos y Flota | Equipo de diseño | SVC-04 y SVC-05, interesados, flujo C, C4 N3 Flota, matriz de interacciones, estados de pedido y flota, recorrido integral | 21 |
| **P7** | *(completar)* | Diseñador de software: patrones y SOLID | Equipo de diseño | C4 N1, C4 N3 Reposición, informe de patrones, evidencia SOLID, CQRS (TRX-04, TRX-05, SVC-06) | 18 |

Los roles pueden rotarse entre sprints (lo permite el enunciado); si ocurre, se registra en el acta de la planning y se actualiza esta tabla por PR.

### 1.2 Revisión cruzada (Definition of Done)

Cada integrante tiene un **revisor por defecto** distinto de sí mismo. Las parejas cruzan dominios para detectar inconsistencias entre módulos que interactúan.

| Integrante | Revisa a | Es revisado por | Por qué esta pareja |
|---|---|---|---|
| P1 | P7 | P7 | Arquitectura ↔ patrones: los patrones deben calzar con los límites |
| P2 | P3 | P3 | Producto ↔ proceso |
| P3 | P2 | P2 | Proceso ↔ producto |
| P4 | P5 | P6 | Inventario es revisado por quien consume la reserva (Pedidos) |
| P5 | P6 | P4 | Reposición es revisada por el dueño de los saldos (Inventario) |
| P6 | P4 | P5 | Pedidos y Flota son revisados por quien define calidad y procesos |
| P7 | P1 | P1 | Patrones ↔ arquitectura |

**Excepciones justificadas** (revisor de dominio en lugar del revisor por defecto):

- **HU-03** la revisa P2 (el PO valida a los interesados).
- **HU-13** la revisa P1 (la propiedad de datos es el núcleo del contrato).
- **HU-15** la revisa P5 (diseñador de SVC-03).

**Regla adicional:** todo PR que modifique `docs/ARQUITECTURA_BASE.md` necesita además la aprobación de **P1** y del **diseñador del módulo afectado** (§1.3).

### 1.3 Diseñador responsable por módulo

| Módulo o componente | Diseñador responsable |
|---|---|
| SVC-01 Catálogo y Ubicaciones · SVC-02 Inventario | P4 |
| SVC-03 Reposición y Compras | P5 |
| SVC-04 Pedidos y Fulfillment · SVC-05 Flota y Operación | P6 |
| SVC-06 Analítica · TRX-04 Proyección de lectura · TRX-05 Data Lakehouse | P7 |
| TRX-01 Dashboard | P2 |
| TRX-02 Capa de Aplicación · TRX-03 Broker de eventos | P1 |

---

## 2. Product backlog

### 2.1 Épicas

| ID | Épica | Módulos | Objetivo |
|---|---|---|---|
| EP-01 | Catálogo y ubicaciones | SVC-01 | Tener un maestro confiable de productos y de la estructura física |
| EP-02 | Inventario y lotes | SVC-02 | Saber qué hay, dónde, de qué lote y cuándo vence |
| EP-03 | Reposición y compras | SVC-03 | Reponer antes del quiebre con decisiones trazables |
| EP-04 | Pedidos y fulfillment | SVC-04 | Reservar con FEFO, preparar y despachar sin perder pedidos |
| EP-05 | Flota y operación | SVC-05 | Coordinar AMR, camiones, muelles y patio |
| EP-06 | Dashboard operativo | TRX-01, TRX-02, TRX-04 | Supervisar en vivo la planta interna y la externa |
| EP-07 | Analítica | SVC-06, TRX-05 | Aprender del histórico y anticipar la demanda |
| EP-08 | Seguridad y acceso | EXT-04, todos los SVC | Que cada usuario haga solo lo que le corresponde |

### 2.2 Historias de negocio (HN)

Prioridad MoSCoW: **M** = Must (MVP) · **S** = Should · **C** = Could. **Orden** = posición en el product backlog. **Diseñada en** = HU que producen sus artefactos.

#### EP-01 · Catálogo y ubicaciones (SVC-01)

| ID | Historia | Criterio de aceptación | Prior. | Orden | Diseñada en |
|---|---|---|---|---|---|
| HN-01 | Como **Administrador** quiero registrar productos con SKU, categoría, unidad y si controlan lote y vencimiento, para que cada recepción exija los datos correctos. | **Dado** un SKU nuevo marcado "requiere vencimiento", **cuando** se guarda, **entonces** queda activo, se emite EVT-01 y toda recepción de ese SKU exigirá lote y fecha de vencimiento. | M | 1 | HU-07, HU-13 |
| HN-02 | Como **Administrador** quiero definir bodegas, zonas y ubicaciones con capacidad y estado, para ubicar las existencias y medir la ocupación. | **Dada** una ubicación bloqueada, **cuando** se intenta recibir en ella, **entonces** Inventario rechaza el registro. | M | 2 | HU-07, HU-13 |

#### EP-02 · Inventario y lotes (SVC-02)

| ID | Historia | Criterio de aceptación | Prior. | Orden | Diseñada en |
|---|---|---|---|---|---|
| HN-03 | Como **Operador de bodega** quiero registrar una recepción con producto, lote, cantidad, ubicación y vencimiento, para que el saldo quede actualizado. | **Dado** un SKU con vencimiento obligatorio, **cuando** falta la fecha, **entonces** se rechaza; **cuando** los datos son válidos, **entonces** movimiento y saldo se guardan juntos (TX-01) y se emiten EVT-04 y EVT-05. | M | 4 | HU-14, HU-16 |
| HN-04 | Como **Operador de bodega** quiero consultar existencias por SKU, ubicación y lote con su vencimiento, para saber qué está realmente disponible. | **Dada** la consulta de un SKU, **entonces** se muestran por lote el saldo físico, reservado y disponible, y la respuesta cumple RNF-REN-01. | M | 5 | HU-14, HU-24 |
| HN-05 | Como **Operador de bodega** quiero transferir existencias entre ubicaciones, para reorganizar la bodega. | **Cuando** se transfiere, **entonces** el saldo total del SKU no cambia, queda un movimiento con origen y destino y se emite EVT-06. | S | 21 | HU-14, HU-24 |
| HN-06 | Como **Supervisor de bodega** quiero registrar ajustes con motivo, para corregir diferencias de conteo con auditoría. | **Dado** un usuario sin ROL-02, **cuando** intenta ajustar, **entonces** se rechaza; con ROL-02 el motivo es obligatorio y el ajuste queda auditado (OPS-01, EVT-07). | S | 22 | HU-14, HU-28 |
| HN-07 | Como **Supervisor de bodega** quiero ver los lotes próximos a vencer, para priorizar su salida. | **Dado** un lote que entra en el umbral de días configurado, **entonces** se emite EVT-12 y aparece en VL-02 y en el KPI "Lotes por vencer" de VL-01. | M | 19 | HU-14, HU-25 |

#### EP-03 · Reposición y compras (SVC-03)

| ID | Historia | Criterio de aceptación | Prior. | Orden | Diseñada en |
|---|---|---|---|---|---|
| HN-08 | Como **Comprador** quiero configurar mínimo y punto de reorden por SKU, para que el sistema detecte el riesgo de agotamiento. | **Cuando** se cambia la política, **entonces** el cambio queda auditado (OPS-04) y se aplica en la siguiente evaluación. | M | 6 | HU-15, HU-17 |
| HN-09 | Como **Comprador** quiero una alerta cuando un SKU cae bajo su mínimo, para actuar antes del quiebre. | **Dado** un mínimo de 100, **cuando** el disponible baja a 90, **entonces** se emite EVT-14 y la alerta aparece en VL-02 en ≤ 5 s; **cuando** vuelve a ≥ 100, **entonces** EVT-15 la retira. | M | 7 | HU-17, HU-22 |
| HN-10 | Como **Comprador** quiero ver una sugerencia de reposición con cantidad, proveedor y justificación, para decidir con datos. | **Dada** una sugerencia vigente, **entonces** muestra mínimo, disponible, demanda considerada y lead time; solo existe una sugerencia vigente por SKU. | M | 8 | HU-15, HU-17 |
| HN-11 | Como **Comprador** quiero aprobar o rechazar una sugerencia, para generar la orden de compra. | **Cuando** aprueba, **entonces** se crea la OC en estado *Emitida* y se emiten EVT-17 y EVT-19; **si** la sugerencia ya fue reemplazada, **entonces** la acción se rechaza con un mensaje claro. | M | 9 | HU-17, HU-25 |
| HN-12 | Como **Operador de bodega** quiero registrar la recepción contra una OC, para cerrar el ciclo de compra. | **Dada** una OC de 200 u., **cuando** se reciben 200, **entonces** la OC pasa a *Cerrada* (EVT-20); **si** se reciben 120, queda *RecibidaParcial*. | M | 10 | HU-16, HU-25 |

#### EP-04 · Pedidos y fulfillment (SVC-04)

| ID | Historia | Criterio de aceptación | Prior. | Orden | Diseñada en |
|---|---|---|---|---|---|
| HN-13 | Como **Supervisor de operaciones** quiero que al registrar un pedido se reserve stock con FEFO, para despachar primero lo que vence antes. | **Dados** L1 (vence el 10-dic) y L2 (vence el 20-dic) disponibles, **cuando** se reserva, **entonces** se asigna primero L1; los lotes *Vencido* o *EnCuarentena* nunca se asignan. | M | 13 | HU-14, HU-18 |
| HN-14 | Como **Supervisor de operaciones** quiero que los pedidos sin stock suficiente queden en espera, para gestionarlos sin perderlos. | **Cuando** el disponible no alcanza, **entonces** el pedido pasa a *EnEspera* (EVT-23), Inventario emite EVT-09 y no se reserva nada parcialmente (MVP). | M | 14 | HU-18, HU-26 |
| HN-15 | Como **Operador de bodega** quiero que se genere la tarea de preparación y confirmar su término, para pasar el pedido a despacho. | **Cuando** el pedido queda *Reservado*, **entonces** se emite EVT-24; **cuando** el operador confirma, **entonces** pasa a *Preparado* (EVT-25) y se solicita despacho (EVT-26). | M | 15 | HU-18, HU-26 |
| HN-16 | Como **Supervisor de operaciones** quiero cancelar un pedido antes del despacho, para liberar su reserva. | **Cuando** se cancela un pedido *Reservado*, **entonces** se emite EVT-28 y, en ≤ 60 s, Inventario libera la reserva (EVT-10) y el disponible se recupera. | S | 20 | HU-18, HU-26 |

#### EP-05 · Flota y operación (SVC-05)

| ID | Historia | Criterio de aceptación | Prior. | Orden | Diseñada en |
|---|---|---|---|---|---|
| HN-17 | Como **Operador de patio y flota** quiero que las tareas de preparación se asignen a un AMR disponible con su ruta, para automatizar el picking. | **Dado** un AMR sin señal por más de 10 s o *FueraDeServicio*, **entonces** no recibe misiones; la asignación se informa con EVT-29. | M | 16 | HU-19, HU-26 |
| HN-18 | Como **Operador de patio y flota** quiero registrar la llegada de camiones y asignarles muelle, para ordenar el patio. | **Cuando** llega un camión, **entonces** aparece en VL-05 (EVT-33); no se asigna un muelle *Ocupado* o *FueraDeServicio*. | M | 17 | HU-19, HU-26 |
| HN-19 | Como **Supervisor de operaciones** quiero que al salir el camión cargado el pedido quede despachado y el stock descontado, para cerrar el ciclo. | **Cuando** se emite EVT-35, **entonces** el pedido pasa a *Despachado* (EVT-27) e Inventario registra la salida consumiendo la reserva (EVT-11, EVT-05). | M | 18 | HU-18, HU-24 |

#### EP-06 · Dashboard operativo (TRX-01, TRX-02, TRX-04)

| ID | Historia | Criterio de aceptación | Prior. | Orden | Diseñada en |
|---|---|---|---|---|---|
| HN-20 | Como **Supervisor de operaciones** quiero ver KPIs y alertas en vivo con la antigüedad del dato, para decidir con información confiable. | **Dada** cualquier vista, **entonces** muestra "Actualizado hace X s"; **si** el retraso supera su objetivo, **entonces** pasa a *Con retraso* o *Desactualizada* (ARQUITECTURA_BASE §8.2). | M | 11 | HU-22, HU-32 |
| HN-21 | Como **Supervisor de operaciones** quiero alternar entre planta interna y planta externa, para supervisar racks y AMR, o muelles y camiones. | **Cuando** cambia de modo, **entonces** solo cambia el mapa central (VL-04 o VL-05) y se mantienen KPIs, alertas, flujo de pedidos y actividad. | M | 12 | HU-22, HU-32 |

#### EP-07 · Analítica (SVC-06, TRX-05)

| ID | Historia | Criterio de aceptación | Prior. | Orden | Diseñada en |
|---|---|---|---|---|---|
| HN-22 | Como **Analista** quiero consultar rotación, quiebres, vencimientos y puntualidad históricos, para mejorar las políticas. | **Dado** el corte diario, **entonces** VL-07 muestra los indicadores calculados desde el Lakehouse, que nunca se usan como saldo operativo. | C | 24 | HU-12, HU-30 |
| HN-23 | Como **Comprador** quiero que las sugerencias consideren un pronóstico de demanda, para reducir quiebres y sobrestock. | **Cuando** hay pronóstico vigente (EVT-37), **entonces** se usa `ReposicionPorPronostico`; **si no**, `ReposicionMinMax`, sin cambiar el flujo. | C | 23 | HU-15, HU-31 |

#### EP-08 · Seguridad y acceso (EXT-04, todos)

| ID | Historia | Criterio de aceptación | Prior. | Orden | Diseñada en |
|---|---|---|---|---|---|
| HN-24 | Como **Administrador** quiero asignar roles a los usuarios, para que cada uno ejecute solo las operaciones permitidas. | **Dado** un usuario con ROL-06, **cuando** intenta aprobar una sugerencia, **entonces** la capacidad dueña (SVC-03) lo rechaza y queda registrado. | M | 3 | HU-28 |

**MVP = todas las HN con prioridad M** (19 historias). Recorren los flujos A, B y C en su camino principal; la analítica del MVP se limita a la ingesta al Lakehouse.

---

## 3. Plan de sprints

### 3.1 Resumen

Cuatro sprints de dos semanas. Fechas a completar en HU-05.

| Sprint | Fechas | Objetivo | Incremento revisable | HU | Pts | Tag al cierre |
|---|---|---|---|---|---|---|
| **S1** | *(dd/mm – dd/mm)* | Acordar qué se diseña, para quién y cómo se reparte | Product backlog priorizado, tablero, riesgos y contrato v1.0.0 aprobado | HU-01 a HU-10 | 28 | `diseno-v0.1.0` |
| **S2** | *(dd/mm – dd/mm)* | Hacer visible la arquitectura base | C4 N1 a N3, propiedad de datos, flujos A, B y C, ADR-001 a ADR-004 y ADR-006 a ADR-008, dashboard v1 | HU-11 a HU-23 | 45 | `diseno-v0.2.0` |
| **S3** | *(dd/mm – dd/mm)* | Cerrar interacciones, calidad y diseño de software | Matriz de interacciones, diagramas de estado, RNF, seguridad, fallos y escalabilidad, patrones, SOLID, ADR-005 y ADR-009, dashboard v2 | HU-24 a HU-33 | 36 | `diseno-v0.3.0` |
| **S4** | *(dd/mm – dd/mm)* | Entregar un paquete coherente, trazable y presentable | Revisión de consistencia, trazabilidad, release, documento consolidado, recorrido integral, informe y presentación | HU-34 a HU-42 | 32 | `diseno-v1.0.0` |

### 3.2 Ceremonias

| Ceremonia | Cuándo | Duración | Evidencia |
|---|---|---|---|
| Planning | Día 1 del sprint | 60 min | `docs/gestion/sprints/sprint-N.md` (sección Planning) + tablero |
| Sincronización | Lunes, miércoles y viernes, asíncrona en el canal del equipo | 10 min | Resumen semanal en `sprint-N.md` |
| Refinamiento | Mitad del sprint | 30 min | Cambios a este documento por PR |
| Review | Último día del sprint | 45 min | `sprint-N.md` (sección Review) + tag del sprint |
| Retrospectiva | Después de la review | 30 min | `sprint-N.md` (sección Retro) con acciones y responsable |

Las tareas de cada HU se desglosan como checklist dentro de su tarjeta del tablero.

---

## 4. Sprint backlog (historias de trabajo HU)

Formato de cada HU: historia · metadatos (`Resp.` responsable, `Rev.` revisor, criterio de la rúbrica, puntos, entregable) · criterios de aceptación (CA).

### 4.1 Sprint 1 · Visión, backlog y límites candidatos

| HU | Título | Resp. | Rev. | Pts |
|---|---|---|---|---|
| HU-01 | Repositorio, estructura y tablero | P3 | P2 | 3 |
| HU-02 | Visión del producto y MVP | P2 | P3 | 3 |
| HU-03 | Interesados y perfiles de usuario | P6 | P2 | 2 |
| HU-04 | Épicas, historias de negocio y priorización | P2 | P3 | 5 |
| HU-05 | Plan de sprints y tablero cargado | P3 | P2 | 2 |
| HU-06 | Contrato maestro de arquitectura v1.0 | P1 | P7 | 5 |
| HU-07 | Glosario y modelo conceptual de dominio | P4 | P6 | 3 |
| HU-08 | Registro inicial de riesgos | P5 | P4 | 2 |
| HU-09 | Referentes y patrones candidatos | P7 | P1 | 2 |
| HU-10 | Registro del Sprint 1 | P3 | P2 | 1 |

#### HU-01 · Repositorio, estructura y tablero
**Como** equipo **necesitamos** un repositorio ordenado y un tablero visible **para** que cada cambio sea trazable desde el primer día.
`Resp.: P3` · `Rev.: P2` · `Rúbrica: Gestión` · `Pts: 3` · `Entregable: README.md, CHANGELOG.md, plantillas/, tablero`
- **CA1.** El repositorio tiene el árbol de §8.1 y un README con propósito, integrantes P1–P7 (nombre y rol), cómo navegar y versión vigente.
- **CA2.** Existen las plantillas de ADR, registro de sprint y revisión de PR en `plantillas/`.
- **CA3.** El tablero tiene las columnas Backlog · Listo · En curso · En revisión · Terminado y una tarjeta por cada HU-01 a HU-42 con responsable, revisor y sprint.
- **CA4.** El README establece que nada entra a `docs/` o `diagramas/` sin PR aprobado por el revisor asignado (§7.1).

#### HU-02 · Visión del producto y MVP
**Como** Product Owner **necesito** una visión y un MVP acordados **para** priorizar con criterio.
`Resp.: P2` · `Rev.: P3` · `Rúbrica: Gestión` · `Pts: 3` · `Entregable: docs/producto/vision-y-mvp.md`
- **CA1.** Visión en una página: problema, usuarios, propuesta de valor y métricas de éxito.
- **CA2.** MVP delimitado (qué entra y qué no), coherente con ARQUITECTURA_BASE §2.3 y con las HN de prioridad M (§2.2).
- **CA3.** Flujo vertical explícito: recepción → actualización de stock → alerta → reposición (ARQUITECTURA_BASE §6.3).
- **CA4.** Supuestos y restricciones listados, incluidos SUP-01 a SUP-03.

#### HU-03 · Interesados y perfiles de usuario
**Como** equipo **necesitamos** conocer a los interesados **para** escribir historias y roles correctos.
`Resp.: P6` · `Rev.: P2` · `Rúbrica: Gestión` · `Pts: 2` · `Entregable: docs/producto/interesados.md`
- **CA1.** Al menos 8 interesados con interés, influencia y necesidad principal.
- **CA2.** Matriz poder/interés.
- **CA3.** Cada perfil de usuario corresponde a un rol ROL-01 a ROL-07.
- **CA4.** Cada interesado está vinculado a al menos una épica (EP-01 a EP-08).

#### HU-04 · Épicas, historias de negocio y priorización
**Como** Product Owner **necesito** un product backlog priorizado con criterios de aceptación **para** guiar el diseño.
`Resp.: P2` · `Rev.: P3` · `Rúbrica: Gestión` · `Pts: 5` · `Entregable: docs/BACKLOG.md §2`
- **CA1.** Épicas EP-01 a EP-08 y al menos 24 HN en formato Como/Quiero/Para con criterios Dado/Cuando/Entonces.
- **CA2.** Cada HN tiene módulo (SVC/TRX), prioridad MoSCoW, orden y la HU que la diseña.
- **CA3.** P4, P5 y P6 aportaron y revisaron las HN de sus épicas (evidencia en el PR).
- **CA4.** El orden respeta dependencias (catálogo y roles antes que inventario; inventario antes que reposición).

#### HU-05 · Plan de sprints y tablero cargado
**Como** Scrum Master **necesito** el plan de los 4 sprints visible **para** que todos sepan qué hacer y cuándo.
`Resp.: P3` · `Rev.: P2` · `Rúbrica: Gestión` · `Pts: 2` · `Entregable: docs/BACKLOG.md §3, tablero`
- **CA1.** Fechas de inicio y fin de los 4 sprints y calendario de ceremonias (§3.2).
- **CA2.** Capacidad declarada por integrante.
- **CA3.** Las HU están asignadas por sprint en el tablero tal como en §4.
- **CA4.** El objetivo de cada sprint está escrito y visible en el tablero.

#### HU-06 · Contrato maestro de arquitectura v1.0
**Como** equipo **necesitamos** un contrato único de límites, datos, eventos e interacciones **para** trabajar en paralelo sin contradicciones.
`Resp.: P1` · `Rev.: P7` · `Rúbrica: Límites y C4` · `Pts: 5` · `Entregable: docs/ARQUITECTURA_BASE.md`
- **CA1.** Contiene SVC, TRX, EXT, UD, propiedad de datos, EVT, INT, VL, estados, puertos y base de calidad.
- **CA2.** Los 7 integrantes lo leyeron y aprobaron (acta en `docs/gestion/actas/`).
- **CA3.** Está etiquetado como `contrato-v1.0.0`.
- **CA4.** Todo cambio posterior sigue ARQUITECTURA_BASE §0.3.

#### HU-07 · Glosario y modelo conceptual de dominio
**Como** equipo **necesitamos** el mismo vocabulario y un modelo de dominio por módulo **para** que todos los diagramas hablen de las mismas cosas.
`Resp.: P4` · `Rev.: P6` · `Rúbrica: Límites y C4` · `Pts: 3` · `Entregable: docs/producto/glosario-y-dominio.md, CLS-dominio-conceptual`
- **CA1.** Glosario ampliado desde ARQUITECTURA_BASE §13 (al menos 30 términos), sin contradicciones con el contrato.
- **CA2.** Diagrama de clases conceptual con las entidades de cada SVC agrupadas por módulo, sin atributos técnicos.
- **CA3.** Las relaciones entre entidades de distintos módulos son solo por identificador (PRI-03).

#### HU-08 · Registro inicial de riesgos
**Como** equipo **necesitamos** anticipar riesgos de proyecto y de arquitectura **para** mitigarlos a tiempo.
`Resp.: P5` · `Rev.: P4` · `Rúbrica: Gestión` · `Pts: 2` · `Entregable: docs/gestion/riesgos.md`
- **CA1.** Al menos 10 riesgos (proyecto y arquitectura) con probabilidad, impacto, mitigación y responsable.
- **CA2.** Los riesgos de arquitectura están vinculados a SVC, FAL o RNF.
- **CA3.** Tiene una columna "última revisión" que se actualiza en cada sprint review.

#### HU-09 · Referentes y patrones candidatos
**Como** equipo **necesitamos** saber qué tomar de los referentes **para** justificar los patrones con fundamento.
`Resp.: P7` · `Rev.: P1` · `Rúbrica: Patrones` · `Pts: 2` · `Entregable: docs/diseno/patrones.md (sección inicial)`
- **CA1.** Resumen de los 5 referentes del enunciado: qué se toma de cada uno y para qué módulo.
- **CA2.** Lista de patrones candidatos con el problema que resuelven y el SVC donde aplican.
- **CA3.** Selección preliminar de al menos 6 patrones, alineada con ARQUITECTURA_BASE §10.

#### HU-10 · Registro del Sprint 1
**Como** Scrum Master **necesito** evidencia de las ceremonias **para** demostrar la gestión del sprint.
`Resp.: P3` · `Rev.: P2` · `Rúbrica: Gestión` · `Pts: 1` · `Entregable: docs/gestion/sprints/sprint-1.md`
- **CA1.** Planning (objetivo, HU, capacidad), seguimiento, review (qué se mostró, retroalimentación, decisiones) y retrospectiva (mantener, mejorar, acciones con responsable).
- **CA2.** Tablero y CHANGELOG actualizados; tag `diseno-v0.1.0` creado.

### 4.2 Sprint 2 · C4, propiedad de datos, flujos y primeros ADR

| HU | Título | Resp. | Rev. | Pts |
|---|---|---|---|---|
| HU-11 | C4 Nivel 1: contexto | P7 | P1 | 3 |
| HU-12 | C4 Nivel 2: contenedores | P1 | P7 | 5 |
| HU-13 | Matriz detallada de propiedad de datos | P4 | P1 | 3 |
| HU-14 | C4 Nivel 3: Inventario y ADR-003 | P4 | P6 | 5 |
| HU-15 | C4 Nivel 3: Reposición y Compras | P7 | P5 | 3 |
| HU-16 | Flujo A: recepción y actualización de existencias | P5 | P4 | 3 |
| HU-17 | Flujo B: riesgo de agotamiento y reposición, ADR-006 | P5 | P4 | 5 |
| HU-18 | Flujo C: reserva FEFO, preparación y despacho | P6 | P5 | 5 |
| HU-19 | C4 Nivel 3: Flota y Operación y ADR-007 | P6 | P5 | 3 |
| HU-20 | ADR-001, ADR-002 y ADR-004 | P1 | P7 | 3 |
| HU-21 | Política de versionamiento y ADR-008 | P3 | P2 | 3 |
| HU-22 | Dashboard v1: vistas y datos | P2 | P3 | 3 |
| HU-23 | Registro del Sprint 2 | P3 | P2 | 1 |

#### HU-11 · C4 Nivel 1: contexto
**Como** equipo **necesitamos** el diagrama de contexto **para** mostrar quién usa el sistema y con qué sistemas externos se relaciona.
`Resp.: P7` · `Rev.: P1` · `Rúbrica: Límites y C4` · `Pts: 3` · `Entregable: diagramas/c4/src/C4N1-contexto + export`
- **CA1.** Smart Inventory como un solo sistema; actores ROL-01 a ROL-07; sistemas EXT-01 a EXT-05.
- **CA2.** Cada relación indica su propósito; no aparecen tecnologías.
- **CA3.** Fuente y exportación en el repositorio, con la leyenda de §7.3.

#### HU-12 · C4 Nivel 2: contenedores
**Como** líder de arquitectura **necesito** el diagrama de contenedores **para** mostrar las unidades de despliegue, sus datos y cómo se comunican.
`Resp.: P1` · `Rev.: P7` · `Rúbrica: Límites y C4` · `Pts: 5` · `Entregable: diagramas/c4/src/C4N2-contenedores + export`
- **CA1.** Muestra UD-1 a UD-5, TRX-03, TRX-05, EXT-01 a EXT-05 y un esquema de datos lógico por SVC.
- **CA2.** Cada relación lleva su modo (S o A) y su ID INT.
- **CA3.** Ningún contenedor accede al esquema de otro (RNF-MAN-02).
- **CA4.** Incluye una nota con la agrupación del MVP y la ruta de evolución (ARQUITECTURA_BASE §2.4).

#### HU-13 · Matriz detallada de propiedad de datos
**Como** equipo **necesitamos** saber quién es dueño de cada dato y cómo lo obtienen los demás **para** que nadie modifique datos ajenos.
`Resp.: P4` · `Rev.: P1` · `Rúbrica: Límites y C4` · `Pts: 3` · `Entregable: docs/arquitectura/propiedad-de-datos.md`
- **CA1.** Cada entidad de ARQUITECTURA_BASE §5 con dueño único, lectores y mecanismo (consulta S o réplica por EVT).
- **CA2.** Incluye todas las réplicas locales de ARQUITECTURA_BASE §3 con su evento de origen.
- **CA3.** Ningún dato tiene dos dueños; es coherente con el C4 N2 (HU-12).

#### HU-14 · C4 Nivel 3: Inventario y ADR-003
**Como** diseñador de Inventario **necesito** su diagrama de componentes **para** mostrar puertos, FEFO y Outbox, y justificar dónde vive la reserva.
`Resp.: P4` · `Rev.: P6` · `Rúbrica: Límites y C4, SOLID` · `Pts: 5` · `Entregable: diagramas/c4/src/C4N3-SVC02-inventario + export, docs/adr/ADR-003`
- **CA1.** Componentes con los nombres de ARQUITECTURA_BASE §10: `ConsultaExistencias`, `GestionReservas`, `RegistroMovimientos`, `PoliticaSeleccionLotes`, `RepositorioSaldos`, `PublicadorEventos`, `ReplicaCatalogo`, `ConsumidorIdempotente`.
- **CA2.** Muestra el outbox y el proceso de detección de vencimientos (EVT-12, EVT-13).
- **CA3.** ADR-003 redactado con al menos 2 alternativas (por ejemplo, reserva en SVC-04) y sus consecuencias.

#### HU-15 · C4 Nivel 3: Reposición y Compras
**Como** diseñador de software **necesito** los componentes de Reposición **para** usarlos como base de la evidencia de Strategy, State y SRP.
`Resp.: P7` · `Rev.: P5` · `Rúbrica: Límites y C4, SOLID` · `Pts: 3` · `Entregable: diagramas/c4/src/C4N3-SVC03-reposicion + export`
- **CA1.** `EvaluadorRiesgoAgotamiento` separado de `EstrategiaReposicion` y de la gestión de sugerencias.
- **CA2.** `PuertoErp` con `AdaptadorErp`; `EstadoSugerencia` y `EstadoOrdenCompra` visibles.
- **CA3.** Muestra la copia local del pronóstico (EVT-37) y la réplica de catálogo.

#### HU-16 · Flujo A: recepción y actualización de existencias
**Como** diseñador de procesos **necesito** modelar la recepción **para** mostrar responsabilidades, transacción local y notificación por Outbox.
`Resp.: P5` · `Rev.: P4` · `Rúbrica: Flujos e interacciones` · `Pts: 3` · `Entregable: ACT-FA-recepcion, SEC-FA-recepcion, docs/arquitectura/flujos/flujo-a-recepcion.md`
- **CA1.** Diagrama de actividad con validaciones: SKU activo, lote y vencimiento cuando corresponda, ubicación activa, OC opcional.
- **CA2.** Diagrama de secuencia con INT-06 a INT-09, EVT-04 y EVT-05, y TX-01 marcada.
- **CA3.** Caminos alternativos: datos inválidos y broker caído (FAL-01).
- **CA4.** Texto que explica la responsabilidad de cada componente en cada paso.

#### HU-17 · Flujo B: riesgo de agotamiento y reposición, ADR-006
**Como** diseñador de procesos **necesito** modelar la reposición **para** mostrar quién detecta, quién decide y quién notifica.
`Resp.: P5` · `Rev.: P4` · `Rúbrica: Flujos e interacciones` · `Pts: 5` · `Entregable: ACT-FB-reposicion, SEC-FB-reposicion, docs/arquitectura/flujos/flujo-b-reposicion.md, docs/adr/ADR-006`
- **CA1.** Actividad y secuencia con INT-10 a INT-14 y la cadena EVT-05 → EVT-14 → EVT-16 → EVT-17 → EVT-19.
- **CA2.** La decisión humana (aprobar o rechazar) y la validación contra el estado vigente están explícitas.
- **CA3.** Incluye la resolución del riesgo (EVT-15) y el caso de sugerencia reemplazada.
- **CA4.** ADR-006 redactado con alternativas (detección en Inventario o en la proyección).

#### HU-18 · Flujo C: reserva FEFO, preparación y despacho
**Como** diseñador de operación **necesito** modelar el ciclo del pedido **para** mostrar la coordinación entre Pedidos, Inventario y Flota.
`Resp.: P6` · `Rev.: P5` · `Rúbrica: Flujos e interacciones` · `Pts: 5` · `Entregable: ACT-FC-reserva-despacho, SEC-FC-reserva-despacho, docs/arquitectura/flujos/flujo-c-reserva-despacho.md`
- **CA1.** Actividad y secuencia con INT-17 a INT-32 y los eventos EVT-21 a EVT-36 según ARQUITECTURA_BASE §6.2.
- **CA2.** Ejemplo FEFO con dos lotes de distinta fecha de vencimiento.
- **CA3.** Caminos alternativos: sin stock (*EnEspera*), Inventario no disponible (*PendienteReserva*) y cancelación con liberación de reserva.
- **CA4.** Justifica por qué la reserva es síncrona (INT-19) y el resto asíncrono.

#### HU-19 · C4 Nivel 3: Flota y Operación y ADR-007
**Como** diseñador de operación **necesito** los componentes de Flota **para** aislar telemetría e integraciones externas.
`Resp.: P6` · `Rev.: P5` · `Rúbrica: Límites y C4` · `Pts: 3` · `Entregable: diagramas/c4/src/C4N3-SVC05-flota + export, docs/adr/ADR-007`
- **CA1.** Componentes: `EstrategiaAsignacionAmr`, `PuertoControladorAmr`, `PuertoTransportista` con sus adaptadores, `AsignadorMuelles` y el procesamiento de telemetría (señal EVT-32 separada de los eventos de negocio).
- **CA2.** Las integraciones con EXT-02, EXT-03 y EXT-05 aparecen solo mediante adaptadores.
- **CA3.** ADR-007 con alternativas y consecuencias.

#### HU-20 · ADR-001, ADR-002 y ADR-004
**Como** líder de arquitectura **necesito** formalizar las decisiones de base **para** que sus razones queden registradas y versionadas.
`Resp.: P1` · `Rev.: P7` · `Rúbrica: Límites y C4` · `Pts: 3` · `Entregable: docs/adr/ADR-001, ADR-002, ADR-004`
- **CA1.** Cada ADR usa la plantilla completa: contexto, al menos 2 opciones, decisión, consecuencias positivas y negativas, fecha y estado.
- **CA2.** Cada ADR enlaza las HU y diagramas que afecta.
- **CA3.** Estado *Aceptado* tras la revisión; índice `docs/adr/README.md` actualizado.

#### HU-21 · Política de versionamiento y ADR-008
**Como** Scrum Master **necesito** una política de versionamiento escrita **para** que cada artefacto tenga historial claro y comparable.
`Resp.: P3` · `Rev.: P2` · `Rúbrica: Versionamiento` · `Pts: 3` · `Entregable: docs/versionamiento/politica-versionamiento.md, docs/adr/ADR-008`
- **CA1.** Desarrolla §7.2 y §7.3: cuándo MAJOR, MINOR o PATCH; nombres y leyenda de diagramas; ciclo de vida de ADR; tags por sprint; formato del CHANGELOG.
- **CA2.** Incluye un ejemplo real de cambio versionado (un diagrama del Sprint 2).
- **CA3.** ADR-008 aceptado.

#### HU-22 · Dashboard v1: vistas y datos
**Como** Product Owner **necesito** definir qué muestra el dashboard y de dónde sale **para** que la propuesta visual sea coherente con la arquitectura.
`Resp.: P2` · `Rev.: P3` · `Rúbrica: Trazabilidad y presentación` · `Pts: 3` · `Entregable: docs/dashboard/propuesta-dashboard.md`
- **CA1.** VL-01 a VL-07 con contenido, eventos de origen y frescura objetivo, coherentes con ARQUITECTURA_BASE §8.
- **CA2.** Cada acción del dashboard indica a qué SVC va y por qué INT.
- **CA3.** Boceto de planta interna y externa basado en las Figuras 3 y 4 del enunciado, guardado en `diagramas/dashboard/`.

#### HU-23 · Registro del Sprint 2
**Como** Scrum Master **necesito** evidencia de las ceremonias **para** demostrar la gestión del sprint.
`Resp.: P3` · `Rev.: P2` · `Rúbrica: Gestión` · `Pts: 1` · `Entregable: docs/gestion/sprints/sprint-2.md`
- **CA1.** Planning, seguimiento, review y retrospectiva registrados; acciones de la retro del Sprint 1 revisadas.
- **CA2.** Tablero y CHANGELOG actualizados; tag `diseno-v0.2.0` creado.

### 4.3 Sprint 3 · Interacciones, calidad, patrones y SOLID

| HU | Título | Resp. | Rev. | Pts |
|---|---|---|---|---|
| HU-24 | Matriz de interacciones completa | P6 | P5 | 5 |
| HU-25 | Diagramas de estado: Lote, Reserva, Sugerencia y Orden de compra | P4 | P6 | 3 |
| HU-26 | Diagramas de estado: Pedido, Misión AMR, AMR, Visita de vehículo y Muelle | P6 | P5 | 3 |
| HU-27 | Requisitos de calidad verificables | P5 | P4 | 5 |
| HU-28 | Seguridad: roles, permisos y operaciones sensibles | P2 | P3 | 3 |
| HU-29 | Disponibilidad, escalabilidad y ADR-009 | P1 | P7 | 3 |
| HU-30 | Informe de patrones y ADR-005 | P7 | P1 | 5 |
| HU-31 | Evidencia de principios SOLID | P7 | P1 | 5 |
| HU-32 | Dashboard v2: vistas interna y externa | P2 | P3 | 3 |
| HU-33 | Registro del Sprint 3 | P3 | P2 | 1 |

#### HU-24 · Matriz de interacciones completa
**Como** equipo **necesitamos** una matriz de interacciones vinculada a los flujos **para** demostrar dependencias coherentes con el negocio.
`Resp.: P6` · `Rev.: P5` · `Rúbrica: Flujos e interacciones` · `Pts: 5` · `Entregable: docs/arquitectura/matriz-interacciones.md`
- **CA1.** Incluye todas las INT de ARQUITECTURA_BASE §7.2 (y las nuevas aprobadas) con origen, destino, propósito, modo S/A, efecto en el flujo de negocio, efecto si falla, flujo y EVT relacionado.
- **CA2.** No contiene endpoints, métodos, códigos ni contenido de mensajes.
- **CA3.** Es coherente con el C4 N2 y con las secuencias A, B y C: cada flecha de una secuencia tiene su INT.

#### HU-25 · Diagramas de estado: Lote, Reserva, Sugerencia y Orden de compra
**Como** diseñador de dominio **necesito** los ciclos de vida de estas entidades **para** controlar transiciones válidas con el patrón State.
`Resp.: P4` · `Rev.: P6` · `Rúbrica: Flujos e interacciones` · `Pts: 3` · `Entregable: EST-lote, EST-reserva, EST-sugerencia, EST-orden-compra`
- **CA1.** Estados y transiciones exactamente como en ARQUITECTURA_BASE §9.
- **CA2.** Cada transición indica el comando o evento que la provoca.
- **CA3.** Se listan transiciones inválidas (por ejemplo, aprobar una sugerencia *Reemplazada*).

#### HU-26 · Diagramas de estado: Pedido, Misión AMR, AMR, Visita de vehículo y Muelle
**Como** diseñador de operación **necesito** los ciclos de vida del pedido y de la flota **para** controlar transiciones válidas con el patrón State.
`Resp.: P6` · `Rev.: P5` · `Rúbrica: Flujos e interacciones` · `Pts: 3` · `Entregable: EST-pedido, EST-mision-amr, EST-amr, EST-visita-vehiculo, EST-muelle`
- **CA1.** Estados y transiciones exactamente como en ARQUITECTURA_BASE §9.
- **CA2.** Cada transición indica el comando o evento que la provoca.
- **CA3.** Se listan transiciones inválidas (por ejemplo, cancelar un pedido *EnDespacho*).

#### HU-27 · Requisitos de calidad verificables
**Como** equipo **necesitamos** requisitos de calidad medibles **para** que la arquitectura se pueda evaluar con criterios objetivos.
`Resp.: P5` · `Rev.: P4` · `Rúbrica: Requisitos de calidad` · `Pts: 5` · `Entregable: docs/arquitectura/requisitos-calidad.md`
- **CA1.** Al menos 2 RNF por atributo (seguridad, disponibilidad, rendimiento, escalabilidad, consistencia, mantenibilidad) con formato fuente · estímulo · entorno · respuesta · medida.
- **CA2.** Umbrales justificados con SUP-01 a SUP-03 (validados o ajustados).
- **CA3.** Cada RNF está vinculado a un SVC, INT o VL y al patrón o ADR que lo hace posible.
- **CA4.** Los RNF semilla de ARQUITECTURA_BASE §11.2 se conservan o su cambio queda registrado.

#### HU-28 · Seguridad: roles, permisos y operaciones sensibles
**Como** Product Owner **necesito** definir quién puede hacer qué **para** proteger las operaciones sensibles del negocio.
`Resp.: P2` · `Rev.: P3` · `Rúbrica: Requisitos de calidad` · `Pts: 3` · `Entregable: docs/arquitectura/seguridad-roles.md`
- **CA1.** Matriz ROL × operación de cada SVC, incluidas OPS-01 a OPS-08.
- **CA2.** Umbral de monto de OPS-03 definido y justificado.
- **CA3.** Explica autorización en la capacidad dueña, auditoría, separación de funciones, identidad entre módulos y manejo de secretos fuera del repositorio, sin configuraciones de herramientas.

#### HU-29 · Disponibilidad, escalabilidad y ADR-009
**Como** líder de arquitectura **necesito** describir cómo se degrada y cómo escala el sistema **para** cumplir los requisitos de disponibilidad.
`Resp.: P1` · `Rev.: P7` · `Rúbrica: Requisitos de calidad` · `Pts: 3` · `Entregable: docs/arquitectura/disponibilidad-escalabilidad.md, docs/adr/ADR-009`
- **CA1.** FAL-01 a FAL-09 desarrollados: detección, degradación, recuperación y qué ve el usuario.
- **CA2.** Tabla de escalamiento: qué UD o consumidor escala, con qué señal y hasta dónde (RNF-ESC-01).
- **CA3.** ADR-009 aceptado.

#### HU-30 · Informe de patrones y ADR-005
**Como** diseñador de software **necesito** justificar los patrones elegidos **para** demostrar que resuelven problemas reales del sistema.
`Resp.: P7` · `Rev.: P1` · `Rúbrica: Patrones` · `Pts: 5` · `Entregable: docs/diseno/patrones.md, docs/adr/ADR-005`
- **CA1.** Al menos 6 patrones justificados (el enunciado exige 4): problema, ubicación (SVC y componente), beneficios, costos y límites.
- **CA2.** Cada patrón referencia el diagrama donde se ve (C4 N3, secuencia o estados).
- **CA3.** ADR-005 aceptado.

#### HU-31 · Evidencia de principios SOLID
**Como** diseñador de software **necesito** mostrar SOLID en diagramas o pseudocódigo **para** evidenciar qué cambios futuros facilita el diseño.
`Resp.: P7` · `Rev.: P1` · `Rúbrica: SOLID` · `Pts: 5` · `Entregable: docs/diseno/solid.md, CLS-SOLID-*`
- **CA1.** Los 5 principios con diagrama de clases o pseudocódigo, usando los nombres de ARQUITECTURA_BASE §10.
- **CA2.** Cada principio indica el cambio futuro que facilita (por ejemplo, agregar `ReposicionPorPronostico` sin tocar el flujo).
- **CA3.** Solo pseudocódigo de diseño, sin código de implementación ni configuración.

#### HU-32 · Dashboard v2: vistas interna y externa
**Como** Product Owner **necesito** la propuesta visual final del dashboard **para** explicar cómo se supervisa la operación y cómo se informa la consistencia eventual.
`Resp.: P2` · `Rev.: P3` · `Rúbrica: Trazabilidad y presentación` · `Pts: 3` · `Entregable: docs/dashboard/propuesta-dashboard.md, diagramas/dashboard/`
- **CA1.** Explica los modos Planta interna y Planta externa y las vistas que comparten.
- **CA2.** El indicador de frescura con sus 3 estados (ARQUITECTURA_BASE §8.2) aparece en el boceto.
- **CA3.** Capturas anotadas: cada elemento visual indica su VL y cada botón su INT.

#### HU-33 · Registro del Sprint 3
**Como** Scrum Master **necesito** evidencia de las ceremonias **para** demostrar la gestión del sprint.
`Resp.: P3` · `Rev.: P2` · `Rúbrica: Gestión` · `Pts: 1` · `Entregable: docs/gestion/sprints/sprint-3.md`
- **CA1.** Planning, seguimiento, review y retrospectiva registrados; acciones de la retro anterior revisadas.
- **CA2.** Tablero y CHANGELOG actualizados; tag `diseno-v0.3.0` creado.

### 4.4 Sprint 4 · Revisión integral, versionamiento, trazabilidad y presentación

| HU | Título | Resp. | Rev. | Pts |
|---|---|---|---|---|
| HU-34 | Revisión integral de consistencia | P1 | P7 | 3 |
| HU-35 | Matriz de trazabilidad | P4 | P6 | 5 |
| HU-36 | Auditoría de versionamiento y release del paquete de diseño | P3 | P2 | 3 |
| HU-37 | Documento de arquitectura consolidado | P1 | P7 | 5 |
| HU-38 | Recorrido integral del flujo vertical | P6 | P5 | 3 |
| HU-39 | Informe final (Word o PDF) | P5 | P4 | 5 |
| HU-40 | Presentación final | P2 | P3 | 5 |
| HU-41 | Verificación final contra rúbrica y DoD | P3 | P2 | 2 |
| HU-42 | Registro del Sprint 4 y retrospectiva final | P3 | P2 | 1 |

#### HU-34 · Revisión integral de consistencia
**Como** líder de arquitectura **necesito** revisar todos los artefactos juntos **para** garantizar que nombres, límites e interacciones coinciden.
`Resp.: P1` · `Rev.: P7` · `Rúbrica: Trazabilidad y presentación` · `Pts: 3` · `Entregable: docs/trazabilidad/revision-consistencia.md`
- **CA1.** Checklist de §6.3 aplicada a todos los artefactos, con resultado por artefacto.
- **CA2.** Hallazgos registrados, asignados y cerrados.
- **CA3.** Contrato actualizado a la versión 1.x que corresponda, con historial y CHANGELOG.

#### HU-35 · Matriz de trazabilidad
**Como** equipo **necesitamos** trazar cada historia hasta sus artefactos **para** demostrar que la arquitectura responde a los requisitos.
`Resp.: P4` · `Rev.: P6` · `Rúbrica: Trazabilidad y presentación` · `Pts: 5` · `Entregable: docs/trazabilidad/matriz-trazabilidad.md`
- **CA1.** Cadena HN → SVC → EVT/INT → diagramas → ADR → RNF → versión del artefacto.
- **CA2.** El 100 % de las HN de prioridad M está trazado.
- **CA3.** Ningún huérfano: no hay artefacto sin HN ni HN de prioridad M sin artefacto.

#### HU-36 · Auditoría de versionamiento y release del paquete de diseño
**Como** Scrum Master **necesito** auditar las versiones **para** entregar un paquete con historial claro de cambios y decisiones.
`Resp.: P3` · `Rev.: P2` · `Rúbrica: Versionamiento` · `Pts: 3` · `Entregable: CHANGELOG.md, docs/versionamiento/, release diseno-v1.0.0`
- **CA1.** Cada diagrama tiene fuente y exportación sincronizadas y leyenda completa.
- **CA2.** Todos los ADR tienen el estado correcto en el índice.
- **CA3.** CHANGELOG completo de `diseno-v0.1.0` a `diseno-v1.0.0`; release `diseno-v1.0.0` publicada con notas.

#### HU-37 · Documento de arquitectura consolidado
**Como** líder de arquitectura **necesito** un documento único de arquitectura **para** entregarlo como pieza central del informe.
`Resp.: P1` · `Rev.: P7` · `Rúbrica: Límites y C4` · `Pts: 5` · `Entregable: docs/arquitectura/documento-arquitectura.md`
- **CA1.** Integra límites, propiedad de datos, C4 N1 a N3, flujos UML, matriz de interacciones (por referencia), requisitos de calidad y ADR.
- **CA2.** Todo se cita por ID y versión; el contenido que vive en otro archivo se enlaza, no se duplica.
- **CA3.** No contiene nada fuera de alcance (ARQUITECTURA_BASE §14).

#### HU-38 · Recorrido integral del flujo vertical
**Como** equipo **necesitamos** un diagrama de extremo a extremo **para** presentar recepción → actualización de stock → alerta → reposición.
`Resp.: P6` · `Rev.: P5` · `Rúbrica: Flujos e interacciones` · `Pts: 3` · `Entregable: SEC-VERTICAL-recorrido-integral, docs/arquitectura/flujos/recorrido-integral.md`
- **CA1.** Secuencia de extremo a extremo según ARQUITECTURA_BASE §6.3 (pasos 1 a 9).
- **CA2.** Cada paso muestra dueño, modo S/A, INT, EVT y qué ve el dashboard.
- **CA3.** Es la base visual de la presentación (HU-40).

#### HU-39 · Informe final (Word o PDF)
**Como** equipo **necesitamos** el informe en el formato pedido **para** subirlo a la plataforma del curso.
`Resp.: P5` · `Rev.: P4` · `Rúbrica: Trazabilidad y presentación` · `Pts: 5` · `Entregable: entregables/informe/`
- **CA1.** Portada con autores y sección de contribuciones individuales basada en `equipo/Px/CONTRIBUCIONES.md`.
- **CA2.** Incluye todos los entregables de §5.2, con figuras exportadas que muestran ID y versión.
- **CA3.** Enlaza el repositorio y la release `diseno-v1.0.0`.

#### HU-40 · Presentación final
**Como** equipo **necesitamos** una presentación clara **para** explicar responsabilidades e interacciones en el recorrido integral.
`Resp.: P2` · `Rev.: P3` · `Rúbrica: Trazabilidad y presentación` · `Pts: 5` · `Entregable: entregables/presentacion/`
- **CA1.** Recorre recepción → actualización de stock → alerta → reposición, explicando responsabilidades e interacciones (HU-38).
- **CA2.** Identifica autores y contribuciones; los 7 integrantes exponen una parte.
- **CA3.** Ensayo cronometrado registrado en un acta.

#### HU-41 · Verificación final contra rúbrica y DoD
**Como** Scrum Master **necesito** verificar la entrega contra la rúbrica **para** no perder puntos por omisiones.
`Resp.: P3` · `Rev.: P2` · `Rúbrica: Todos los criterios` · `Pts: 2` · `Entregable: docs/trazabilidad/verificacion-entrega.md`
- **CA1.** Los 8 criterios de la rúbrica con evidencia enlazada (§5.1).
- **CA2.** DoD verificada para cada HU.
- **CA3.** Lista de pendientes en cero antes de entregar.

#### HU-42 · Registro del Sprint 4 y retrospectiva final
**Como** Scrum Master **necesito** cerrar el proyecto con evidencia **para** completar el registro de los cuatro sprints.
`Resp.: P3` · `Rev.: P2` · `Rúbrica: Gestión` · `Pts: 1` · `Entregable: docs/gestion/sprints/sprint-4.md`
- **CA1.** Planning, seguimiento, review y retrospectiva final (qué aprendió el equipo en los 4 sprints).
- **CA2.** Tablero cerrado; enlace a la release `diseno-v1.0.0` (HU-36).

---

## 5. Cobertura

### 5.1 Rúbrica

| Criterio (puntos) | HU que lo cubren | Responsables |
|---|---|---|
| Gestión del proyecto y sprints (1,5) | HU-01, HU-02, HU-03, HU-04, HU-05, HU-08, HU-10, HU-23, HU-33, HU-42 | P2, P3, P5, P6 |
| Límites de servicio y arquitectura C4 (2,0) | HU-06, HU-07, HU-11, HU-12, HU-13, HU-14, HU-15, HU-19, HU-20, HU-37 | P1, P4, P6, P7 |
| Flujos e interacciones arquitectónicas (1,5) | HU-16, HU-17, HU-18, HU-24, HU-25, HU-26, HU-38 | P4, P5, P6 |
| Requisitos de calidad (1,0) | HU-27, HU-28, HU-29 | P1, P2, P5 |
| Patrones de software (1,0) | HU-09, HU-30 | P7 |
| Principios SOLID (1,0) | HU-31 (con base en HU-14 y HU-15) | P7, P4 |
| Versionamiento de artefactos de diseño (1,0) | HU-21, HU-36 | P3 |
| Trazabilidad y presentación (1,0) | HU-22, HU-32, HU-34, HU-35, HU-39, HU-40, HU-41 | P1, P2, P3, P4, P5 |

### 5.2 Entregables del enunciado (§11)

| Entregable | Dónde vive | HU |
|---|---|---|
| Repositorio con README, tablero/backlog, historial de cambios, ramas y releases | `README.md`, `CHANGELOG.md`, tablero, tags | HU-01, HU-05, HU-36 |
| Visión, interesados, MVP, épicas, historias y criterios de aceptación | `docs/producto/`, `docs/BACKLOG.md` §2 | HU-02, HU-03, HU-04 |
| Registro de cuatro sprints | `docs/gestion/sprints/` | HU-10, HU-23, HU-33, HU-42 |
| Documento de arquitectura (límites, propiedad de datos, C4, UML, calidad, ADR) | `docs/arquitectura/documento-arquitectura.md` + `docs/adr/` + `diagramas/` | HU-37 (integra HU-11 a HU-20 y HU-25 a HU-29) |
| Matriz de interacciones | `docs/arquitectura/matriz-interacciones.md` | HU-24 |
| Informe de patrones y SOLID | `docs/diseno/` | HU-30, HU-31 |
| Política de versionamiento con historial y trazabilidad | `docs/versionamiento/`, `CHANGELOG.md` | HU-21, HU-36 |
| Propuesta visual del dashboard | `docs/dashboard/`, `diagramas/dashboard/` | HU-22, HU-32 |
| Presentación final (recepción → stock → alerta → reposición) | `entregables/presentacion/` | HU-38, HU-40 |
| Informe PDF o Word | `entregables/informe/` | HU-39 |

### 5.3 Carga por integrante

| Integrante | S1 | S2 | S3 | S4 | Total |
|---|---|---|---|---|---|
| P1 | 5 | 8 | 3 | 8 | 24 |
| P2 | 8 | 3 | 6 | 5 | 22 |
| P3 | 6 | 4 | 1 | 6 | 17 |
| P4 | 3 | 8 | 3 | 5 | 19 |
| P5 | 2 | 8 | 5 | 5 | 20 |
| P6 | 2 | 8 | 8 | 3 | 21 |
| P7 | 2 | 6 | 10 | 0 | 18 |
| **Total** | **28** | **45** | **36** | **32** | **141** |

P3 tiene menos puntos porque facilita todas las ceremonias. P7 no tiene HU propias en el Sprint 4: su capacidad se dedica a revisar HU-34 y HU-37 y a corregir patrones y SOLID según los hallazgos de la revisión integral.

---

## 6. Calidad del trabajo

### 6.1 Definition of Ready (antes de empezar una HU)

- Tiene responsable y revisor asignados.
- Sus criterios de aceptación son verificables.
- Está vinculada a una HN o a un módulo (SVC/TRX) y a un criterio de la rúbrica.
- Los artefactos de los que depende ya están aprobados (o se acordó trabajar en paralelo).
- Cabe en el sprint según la capacidad declarada.

### 6.2 Definition of Done (para cerrar una HU)

1. Cumple todos sus criterios de aceptación y está vinculada a su módulo (SVC/TRX) y a sus HN.
2. Los diagramas, modelos de proceso o ADR afectados se actualizaron y **los aprobó el revisor asignado** en el PR.
3. Los artefactos modificados están **versionados**: leyenda con versión, entrada en `CHANGELOG.md`, y fuente y exportación en el mismo PR.
4. Pasa la checklist de revisión cruzada (§6.3).
5. La evidencia de revisión, retroalimentación y decisiones quedó en el registro del sprint.
6. La tarjeta está en *Terminado* con enlace al PR.
7. No se exige evidencia de pipeline ni de despliegue (fuera del alcance de la materia).

### 6.3 Checklist de revisión cruzada

Copiar en la descripción de cada PR y marcar:

- [ ] Los nombres de SVC, TRX, EXT, EVT, INT, VL, estados y puertos son **idénticos** a `ARQUITECTURA_BASE.md`.
- [ ] Ningún diagrama muestra a un módulo leyendo o escribiendo datos de otro (PRI-02).
- [ ] Cada interacción indica S o A y coincide con su INT.
- [ ] Los eventos están en pasado y los emite solo su dueño.
- [ ] El dashboard y la proyección no aparecen como origen de decisiones ni como fuente de verdad.
- [ ] Fuente y exportación presentes y sincronizadas; leyenda completa (§7.3).
- [ ] Sin contenido fuera de alcance: endpoints, métodos HTTP, códigos de estado, JSON/OpenAPI, configuración de broker o colas, Docker, CI/CD, despliegue.
- [ ] Vinculado a su HU/HN y a un criterio de la rúbrica; `CHANGELOG.md` actualizado.
- [ ] Sin secretos ni datos personales reales.

---

## 7. Flujo de trabajo y versionamiento

> Este flujo versiona **artefactos de diseño**. El flujo de ramas, CI/CD y releases del **software** corresponde a Desarrollo de Software Moderno.

### 7.1 Ramas, PR, commits y tags

| Elemento | Convención | Ejemplo |
|---|---|---|
| Rama principal | `main`: solo contenido aprobado | — |
| Rama de trabajo | Una por HU: `hu-NN-descripcion-corta` | `hu-14-c4n3-inventario` |
| Commit | `HU-NN: verbo en imperativo + qué cambió` | `HU-14: agrega el puerto GestionReservas al C4 N3` |
| Pull request | Título `HU-NN · Título de la HU`; revisor = revisor de la HU; checklist §6.3 en la descripción | `HU-14 · C4 Nivel 3: Inventario y ADR-003` |
| PR que toca el contrato | Además aprueban P1 y el diseñador del módulo afectado (§1.3) | — |
| Tag del contrato | `contrato-vX.Y.Z` en cada versión aprobada de `ARQUITECTURA_BASE.md` | `contrato-v1.0.0` |
| Tag del paquete | `diseno-vX.Y.Z` al cierre de cada sprint | `diseno-v0.2.0` |

### 7.2 Versionamiento semántico de artefactos

Aplica al contrato, a cada documento y a cada diagrama.

| Tipo de cambio | Sube | Ejemplo |
|---|---|---|
| Cambia un límite o el dueño de un dato; se elimina o renombra un SVC, EVT, INT o VL | **MAJOR** | Mover la reserva de SVC-02 a SVC-04 |
| Se agrega algo compatible: un EVT, INT, VL, RNF o diagrama nuevo | **MINOR** | Agregar un evento nuevo aprobado por ADR |
| Corrección de forma sin cambiar el significado | **PATCH** | Corregir una etiqueta en un diagrama |

- **ADR:** no llevan versión. Ciclo de vida: *Propuesto → Aceptado → Reemplazado por ADR-nnn* o *Deprecado*. Un ADR aceptado no se edita; se crea uno nuevo que lo reemplaza.
- **Paquete de diseño:** `diseno-v0.1.0` (S1), `diseno-v0.2.0` (S2), `diseno-v0.3.0` (S3), `diseno-v1.0.0` (S4).
- **CHANGELOG.md:** una sección por versión del paquete con fecha, HU incluidas, artefactos cambiados y ADR aceptados.

### 7.3 Convención de diagramas

**Nombre:** `TIPO-ALCANCE-descripcion`, igual en la fuente (`src/`) y en la exportación (`export/`).

| Tipo | Prefijo | Ejemplos |
|---|---|---|
| C4 | `C4N1`, `C4N2`, `C4N3` | `C4N1-contexto`, `C4N2-contenedores`, `C4N3-SVC02-inventario`, `C4N3-SVC03-reposicion`, `C4N3-SVC05-flota` |
| Actividad | `ACT` | `ACT-FA-recepcion`, `ACT-FB-reposicion`, `ACT-FC-reserva-despacho` |
| Secuencia | `SEC` | `SEC-FA-recepcion`, `SEC-FB-reposicion`, `SEC-FC-reserva-despacho`, `SEC-VERTICAL-recorrido-integral` |
| Estados | `EST` | `EST-pedido`, `EST-reserva`, `EST-lote`, `EST-sugerencia`, `EST-orden-compra`, `EST-mision-amr`, `EST-amr`, `EST-visita-vehiculo`, `EST-muelle` |
| Clases | `CLS` | `CLS-dominio-conceptual`, `CLS-SOLID-inventario`, `CLS-SOLID-reposicion`, `CLS-SOLID-flota` |
| Dashboard | `DSH` | `DSH-planta-interna`, `DSH-planta-externa` |

**Leyenda obligatoria** en cada diagrama: `ID · vX.Y.Z · AAAA-MM-DD · autor (Px) · HU-NN`.

**Herramienta recomendada:** PlantUML (con C4-PlantUML para los C4), porque es texto y el historial muestra exactamente qué cambió. Para los bocetos del dashboard se acepta draw.io (el archivo `.drawio` también se versiona). Exportar en SVG (preferido) o PNG con el mismo nombre en `export/`, siempre en el mismo PR que la fuente.

---

## 8. Estructura del repositorio

### 8.1 Árbol

```text
smart-inventory/
├── README.md                          # Portada: propósito, integrantes y roles, cómo navegar, versión vigente (HU-01)
├── CHANGELOG.md                       # Historial del paquete de diseño por versión (HU-01, HU-36)
│
├── docs/                              # TODO lo oficial y aprobado por PR
│   ├── ARQUITECTURA_BASE.md           # Contrato maestro: nombres canónicos (P1, HU-06)
│   ├── BACKLOG.md                     # Este documento (P2 §2, P3 resto)
│   │
│   ├── producto/
│   │   ├── vision-y-mvp.md            # HU-02 (P2)
│   │   ├── interesados.md             # HU-03 (P6)
│   │   └── glosario-y-dominio.md      # HU-07 (P4)
│   │
│   ├── gestion/
│   │   ├── riesgos.md                 # HU-08 (P5)
│   │   ├── sprints/
│   │   │   ├── sprint-1.md            # HU-10 (P3)
│   │   │   ├── sprint-2.md            # HU-23 (P3)
│   │   │   ├── sprint-3.md            # HU-33 (P3)
│   │   │   └── sprint-4.md            # HU-42 (P3)
│   │   └── actas/
│   │       └── AAAA-MM-DD-tema.md     # Acuerdos, aprobación del contrato, ensayos
│   │
│   ├── arquitectura/
│   │   ├── documento-arquitectura.md  # HU-37 (P1) consolidado
│   │   ├── propiedad-de-datos.md      # HU-13 (P4)
│   │   ├── matriz-interacciones.md    # HU-24 (P6)
│   │   ├── requisitos-calidad.md      # HU-27 (P5)
│   │   ├── seguridad-roles.md         # HU-28 (P2)
│   │   ├── disponibilidad-escalabilidad.md  # HU-29 (P1)
│   │   └── flujos/
│   │       ├── flujo-a-recepcion.md         # HU-16 (P5)
│   │       ├── flujo-b-reposicion.md        # HU-17 (P5)
│   │       ├── flujo-c-reserva-despacho.md  # HU-18 (P6)
│   │       └── recorrido-integral.md        # HU-38 (P6)
│   │
│   ├── adr/
│   │   ├── README.md                  # Índice con estado de cada ADR
│   │   ├── ADR-001-monolito-modular-evolutivo.md
│   │   ├── ADR-002-propiedad-datos-por-esquema.md
│   │   ├── ADR-003-reserva-y-fefo-en-inventario.md
│   │   ├── ADR-004-sincrono-y-eventos-con-outbox.md
│   │   ├── ADR-005-proyeccion-de-lectura-cqrs.md
│   │   ├── ADR-006-deteccion-riesgo-en-reposicion.md
│   │   ├── ADR-007-flota-como-unidad-separada.md
│   │   ├── ADR-008-versionamiento-de-artefactos.md
│   │   └── ADR-009-replicas-locales-de-referencia.md
│   │
│   ├── diseno/
│   │   ├── patrones.md                # HU-09, HU-30 (P7)
│   │   └── solid.md                   # HU-31 (P7), con pseudocódigo
│   │
│   ├── dashboard/
│   │   └── propuesta-dashboard.md     # HU-22, HU-32 (P2)
│   │
│   ├── trazabilidad/
│   │   ├── matriz-trazabilidad.md     # HU-35 (P4)
│   │   ├── revision-consistencia.md   # HU-34 (P1)
│   │   └── verificacion-entrega.md    # HU-41 (P3)
│   │
│   └── versionamiento/
│       └── politica-versionamiento.md # HU-21, HU-36 (P3)
│
├── diagramas/                         # Fuentes y exportaciones, siempre en pareja
│   ├── README.md                      # Convención §7.3 y cómo exportar
│   ├── c4/
│   │   ├── src/                       # C4N1-contexto.puml, C4N2-contenedores.puml, C4N3-SVC02-inventario.puml ...
│   │   └── export/                    # C4N1-contexto.svg, C4N2-contenedores.svg ...
│   ├── actividad/   { src/ , export/ }   # ACT-FA, ACT-FB, ACT-FC
│   ├── secuencia/   { src/ , export/ }   # SEC-FA, SEC-FB, SEC-FC, SEC-VERTICAL
│   ├── estados/     { src/ , export/ }   # EST-*
│   ├── clases/      { src/ , export/ }   # CLS-dominio-conceptual, CLS-SOLID-*
│   └── dashboard/   { src/ , export/ }   # DSH-planta-interna, DSH-planta-externa
│
├── plantillas/
│   ├── plantilla-adr.md
│   ├── plantilla-sprint.md
│   └── plantilla-revision-pr.md       # Checklist §6.3
│
├── equipo/                            # Espacio personal: NO es entregable
│   ├── P1/
│   │   ├── CONTRIBUCIONES.md          # HU hechas, PR, revisiones realizadas
│   │   ├── registro-ia.md             # Prompts usados y qué se aceptó o corrigió
│   │   └── borradores/
│   ├── P2/   (misma estructura)
│   ├── P3/
│   ├── P4/
│   ├── P5/
│   ├── P6/
│   └── P7/
│
└── entregables/
    ├── informe/
    │   ├── fuente/                    # Archivo editable del informe
    │   └── Informe-Smart-Inventory-v1.0.0.pdf   # HU-39 (P5)
    └── presentacion/
        ├── Presentacion-Smart-Inventory # Archivo de diapositivas, HU-40 (P2)
        └── guion.md                   # Reparto de exposición entre P1 y P7
```

### 8.2 Por qué esta estructura

- **`docs/` vs. `equipo/`:** lo oficial vive en `docs/` y `diagramas/` y solo entra por PR revisado. `equipo/` es el espacio de cada integrante (borradores, registro de IA, contribuciones); nada ahí cuenta como entregable. Así los borradores de las IAs no contaminan el paquete oficial.
- **`src/` y `export/` en pareja:** el enunciado exige incluir los diagramas fuente **y** las exportaciones usadas en el informe. Mismo nombre en ambas carpetas: se ve de inmediato si una exportación está desactualizada.
- **Carpeta por tipo de diagrama:** facilita encontrar todos los C4 o todas las secuencias, y coincide con las secciones del documento de arquitectura.
- **`docs/adr/` con índice:** un archivo por decisión, numerados e inmutables; el índice muestra el estado de cada uno sin abrirlos.
- **Un archivo por flujo:** P5 y P6 trabajan en paralelo sin conflictos de edición.
- **`entregables/` separado:** el informe y la presentación son productos derivados del repositorio, no fuentes de verdad.

### 8.3 Qué mantiene cada integrante

| Integrante | Archivos oficiales que mantiene | Espacio personal |
|---|---|---|
| P1 | `docs/ARQUITECTURA_BASE.md`, `docs/arquitectura/documento-arquitectura.md`, `docs/arquitectura/disponibilidad-escalabilidad.md`, `docs/trazabilidad/revision-consistencia.md`, `diagramas/c4/` (C4N2), ADR-001, 002, 004, 009 | `equipo/P1/` |
| P2 | `docs/BACKLOG.md` §2, `docs/producto/vision-y-mvp.md`, `docs/dashboard/`, `docs/arquitectura/seguridad-roles.md`, `diagramas/dashboard/`, `entregables/presentacion/` | `equipo/P2/` |
| P3 | `README.md`, `CHANGELOG.md`, `docs/BACKLOG.md` §1 y §3 a §9, `docs/gestion/sprints/`, `docs/gestion/actas/`, `docs/versionamiento/`, `docs/trazabilidad/verificacion-entrega.md`, `plantillas/`, `docs/adr/README.md`, ADR-008 | `equipo/P3/` |
| P4 | `docs/producto/glosario-y-dominio.md`, `docs/arquitectura/propiedad-de-datos.md`, `docs/trazabilidad/matriz-trazabilidad.md`, `diagramas/c4/` (C4N3-SVC02), `diagramas/estados/` (lote, reserva, sugerencia, OC), `diagramas/clases/` (dominio conceptual), ADR-003 | `equipo/P4/` |
| P5 | `docs/gestion/riesgos.md`, `docs/arquitectura/requisitos-calidad.md`, `docs/arquitectura/flujos/flujo-a-recepcion.md` y `flujo-b-reposicion.md`, `diagramas/actividad/` y `diagramas/secuencia/` (FA, FB), `entregables/informe/`, ADR-006 | `equipo/P5/` |
| P6 | `docs/producto/interesados.md`, `docs/arquitectura/matriz-interacciones.md`, `docs/arquitectura/flujos/flujo-c-reserva-despacho.md` y `recorrido-integral.md`, `diagramas/actividad/` y `diagramas/secuencia/` (FC, VERTICAL), `diagramas/c4/` (C4N3-SVC05), `diagramas/estados/` (pedido, misión AMR, AMR, visita de vehículo, muelle), ADR-007 | `equipo/P6/` |
| P7 | `docs/diseno/`, `diagramas/c4/` (C4N1, C4N3-SVC03), `diagramas/clases/` (CLS-SOLID-*), ADR-005 | `equipo/P7/` |

---

## 9. Protocolo de trabajo con IA

Cada integrante usa su propia IA, pero **todas parten del mismo contexto**. Antes de pedir cualquier artefacto:

1. Adjunta `docs/ARQUITECTURA_BASE.md`, `docs/BACKLOG.md` y los artefactos aprobados de los que depende tu HU.
2. Usa el prompt base de abajo.
3. Guarda el prompt y un resumen de lo que aceptaste o corregiste en `equipo/Px/registro-ia.md`.
4. Revisa la salida con la checklist §6.3 **antes** de abrir el PR. La IA no reemplaza la revisión cruzada.

**Prompt base (copiar y completar):**

```text
Eres el asistente de diseño de <Px – nombre>, integrante del equipo Smart Inventory
(Diseño de Sistemas, USFQ).

Te adjunto:
1. docs/ARQUITECTURA_BASE.md (contrato maestro, versión <X.Y.Z>)
2. docs/BACKLOG.md
3. Artefactos aprobados de los que depende mi historia: <lista>

Mi historia es <HU-NN · título>. Sus criterios de aceptación son:
<copiar CA1, CA2, ...>

Reglas obligatorias:
- Usa solo los identificadores y nombres del contrato (SVC, TRX, EXT, UD, EVT, INT, VL,
  estados de §9, puertos de §10, RNF, ROL, OPS). No inventes sinónimos ni los traduzcas
  al inglés.
- Si necesitas algo que el contrato no tiene, márcalo como «PROPUESTA DE CAMBIO»,
  explica por qué y no lo uses como si fuera oficial.
- Respeta la propiedad de datos: ningún módulo lee ni escribe datos de otro. Respeta el
  modo síncrono o asíncrono de cada INT.
- No incluyas endpoints, métodos HTTP, códigos de estado, JSON/OpenAPI, configuración de
  broker o colas, Docker, CI/CD ni despliegue.
- Diagramas en PlantUML (C4-PlantUML para C4) con la leyenda:
  ID · versión · fecha · autor · HU.
- Responde en español, en Markdown listo para el repositorio.
```

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-20 | P2, P3 | Versión inicial del backlog, plan de sprints y estructura del repositorio | HU-04, HU-05 |
