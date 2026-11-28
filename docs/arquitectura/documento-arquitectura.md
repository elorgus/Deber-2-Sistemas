# Smart Inventory · Documento de arquitectura

| Campo | Valor |
|---|---|
| Documento | `docs/arquitectura/documento-arquitectura.md` |
| Versión | **1.0.0** |
| Estado | Propuesto para revisión (Sprint 4 · HU-37) |
| Dueño | P1 · Líder de arquitectura |
| Revisión | P7 |
| Fecha | 2026-11-27 |
| Rúbrica | Límites de servicio y arquitectura C4 |
| Contrato | [`ARQUITECTURA_BASE.md`](../ARQUITECTURA_BASE.md) |
| Backlog | [`BACKLOG.md`](../BACKLOG.md) |

> Este documento es la **portada navegable** de la arquitectura. No repite el contenido de los demás archivos: cada sección explica en pocas líneas qué contiene y lo enlaza. Todo se cita por **ID y versión**; si algo cambia en su archivo oficial, solo se actualiza la tabla de esta portada.

---

## Índice

1. [Introducción](#1-introducción)
2. [Capacidades y límites](#2-capacidades-y-límites)
3. [Propiedad de datos](#3-propiedad-de-datos)
4. [Vistas C4](#4-vistas-c4)
5. [Flujos de negocio e interacciones](#5-flujos-de-negocio-e-interacciones)
6. [Requisitos de calidad](#6-requisitos-de-calidad)
7. [Decisiones de arquitectura](#7-decisiones-de-arquitectura)
8. [Conclusiones](#8-conclusiones)

---

## 1. Introducción

Smart Inventory es el diseño de un sistema de inventario inteligente para un centro de distribución con muelles, flota de AMR y reposición automatizada. El diseño parte de un **monolito modular evolutivo** con dos unidades separadas desde el MVP (ADR-001) y se describe solo a nivel arquitectónico: qué componentes existen, qué datos poseen y cómo se comunican.

### 1.1 Cómo se usa este documento

| Si necesitas… | Ve a… |
|---|---|
| Saber qué módulos existen y dónde se despliegan | [Sección 2](#2-capacidades-y-límites) |
| Saber quién es dueño de un dato | [Sección 3](#3-propiedad-de-datos) |
| Ver los diagramas C4 | [Sección 4](#4-vistas-c4) |
| Seguir un proceso de negocio de punta a punta | [Sección 5](#5-flujos-de-negocio-e-interacciones) |
| Verificar disponibilidad, seguridad o escalabilidad | [Sección 6](#6-requisitos-de-calidad) |
| Entender por qué se decidió algo | [Sección 7](#7-decisiones-de-arquitectura) |

### 1.2 Fuente de verdad y reglas de citación

- **Fuente única de nombres:** [`ARQUITECTURA_BASE.md`](../ARQUITECTURA_BASE.md). Todo identificador (`SVC`, `TRX`, `EXT`, `UD`, `PRI`, `EVT`, `INT`, `TX`, `CE`, `VL`, `RNF`, `FAL`, `ROL`, `OPS`, `ADR`) se usa tal como allí aparece.
- **Cómo se cita en este documento:** ID del artefacto + versión + enlace. Los **ADR no llevan versión** (`BACKLOG.md` §7.2); su estado vigente está en su [índice](../adr/README.md).
- **Rutas de diagramas:** los enlaces a diagramas siguen la convención `docs/diagramas/fuente/` y `docs/diagramas/export/`, la misma que usan los ADR.

### 1.3 Alcance (CA3)

Este documento y todo lo que enlaza se limita a diseño arquitectónico. Quedan fuera los temas de [`ARQUITECTURA_BASE.md` §14](../ARQUITECTURA_BASE.md#14-fuera-de-alcance-de-este-deber-recordatorio): diseño detallado de APIs, formato de mensajes y configuración del broker, pruebas, CI/CD, DevOps, despliegue, contenedores e infraestructura como código.

### 1.4 Contrato y glosario

| Artefacto | ID / HU | Versión | Enlace |
|---|---|---|---|
| Contrato maestro | `ARQUITECTURA_BASE` · HU-06 | **1.0.0** | [ARQUITECTURA_BASE.md](../ARQUITECTURA_BASE.md) |
| Respuestas a las decisiones del enunciado | `ARQUITECTURA_BASE` §1 | igual que el contrato | [§1](../ARQUITECTURA_BASE.md#1-respuestas-a-las-decisiones-exigidas-por-el-enunciado) |
| Glosario y modelo conceptual de dominio | `glosario-y-dominio` · HU-07 | **1.0.0** | [glosario-y-dominio.md](../producto/glosario-y-dominio.md) |
| Backlog, equipo y plan de sprints | `BACKLOG` · HU-05 | **1.0.0** | [BACKLOG.md](../BACKLOG.md) |

---

## 2. Capacidades y límites

El sistema se divide en **seis capacidades de negocio** (SVC-01 a SVC-06), apoyadas por cinco componentes transversales (TRX-01 a TRX-05) y cinco sistemas externos (EXT-01 a EXT-05). Se agrupan en **cinco unidades de despliegue** (UD-1 a UD-5) más la infraestructura gestionada. UD-1 reúne el núcleo transaccional (SVC-01 a SVC-04) como monolito modular; Flota (UD-2) y Analítica (UD-3) se separan desde el MVP. Los límites los gobiernan nueve principios (PRI-01 a PRI-09), entre ellos un dueño por dato (PRI-01) y ninguna escritura cruzada (PRI-02). La ruta para extraer módulos a microservicios está definida por señales medibles.

| Tema | Contenido | Versión | Enlace |
|---|---|---|---|
| Estilo arquitectónico | Monolito modular evolutivo y por qué | contrato 1.0.0 | [§2.1](../ARQUITECTURA_BASE.md#21-estilo-monolito-modular-evolutivo-adr-001) |
| Principios de límites y datos | PRI-01 a PRI-09 | contrato 1.0.0 | [§2.2](../ARQUITECTURA_BASE.md#22-principios-de-límites-y-datos) |
| Unidades de despliegue | UD-1 a UD-5 y su razón de agrupar | contrato 1.0.0 | [§2.3](../ARQUITECTURA_BASE.md#23-unidades-de-despliegue-del-mvp) |
| Ruta de evolución | Orden y señales para extraer módulos | contrato 1.0.0 | [§2.4](../ARQUITECTURA_BASE.md#24-ruta-de-evolución-hacia-microservicios) |
| Catálogo de capacidades | SVC-01 a SVC-06: responsabilidad, datos, eventos, reglas | contrato 1.0.0 | [§3](../ARQUITECTURA_BASE.md#3-catálogo-de-capacidades-svc) |
| Componentes transversales y externos | TRX-01 a TRX-05 · EXT-01 a EXT-05 | contrato 1.0.0 | [§4](../ARQUITECTURA_BASE.md#4-componentes-transversales-y-sistemas-externos) |
| Nombres canónicos de puertos y adaptadores | Elementos citados por C4 N3, SOLID y patrones | contrato 1.0.0 | [§10](../ARQUITECTURA_BASE.md#10-nombres-canónicos-de-puertos-estrategias-y-adaptadores) |

Decisiones que sustentan estos límites: [ADR-001](../adr/ADR-001-monolito-modular-evolutivo.md) (monolito modular) y [ADR-007](../adr/ADR-007-flota-como-unidad-separada.md) (Flota separada).

---

## 3. Propiedad de datos

Cada dato tiene **un único dueño** (PRI-01). Ningún módulo lee ni escribe el esquema de otro (PRI-02); entre módulos solo se comparte el identificador (PRI-03). Cuando un módulo necesita datos de referencia de otro, usa una operación síncrona del dueño o una **réplica local de solo lectura** alimentada por eventos (PRI-04). Los datos derivados, como las vistas de TRX-04 y el Lakehouse TRX-05, nunca son fuente de verdad. Cada hecho que cambia un dato se comunica con un evento oficial (EVT-01 a EVT-38) emitido solo por su dueño.

| Artefacto | ID / HU | Versión | Enlace |
|---|---|---|---|
| Matriz detallada de propiedad de datos | `propiedad-de-datos` · HU-13 | **1.0.0** | [propiedad-de-datos.md](./propiedad-de-datos.md) |
| Propiedad de datos (fuente única) | `ARQUITECTURA_BASE` §5 | contrato 1.0.0 | [§5](../ARQUITECTURA_BASE.md#5-propiedad-de-datos-fuente-única-de-verdad) |
| Catálogo de eventos de negocio | `ARQUITECTURA_BASE` §6 (EVT-01 a EVT-38) | contrato 1.0.0 | [§6](../ARQUITECTURA_BASE.md#6-eventos-de-negocio-oficiales) |
| Modelo conceptual de dominio | `CLS-dominio-conceptual` · HU-07 | **1.0.0** | [glosario-y-dominio.md](../producto/glosario-y-dominio.md) |
| Estados canónicos de las entidades | `ARQUITECTURA_BASE` §9 | contrato 1.0.0 | [§9](../ARQUITECTURA_BASE.md#9-estados-canónicos-diagramas-de-estado-y-patrón-state) |

Decisiones relacionadas: [ADR-002](../adr/ADR-002-propiedad-datos-por-esquema.md) (aislamiento por esquema), [ADR-003](../adr/ADR-003-reserva-y-fefo-en-inventario.md) (reserva y FEFO en Inventario) y [ADR-009](../adr/ADR-009-replicas-locales-de-referencia.md) (réplicas locales).

---

## 4. Vistas C4

La arquitectura se muestra en tres niveles. El **nivel 1** sitúa el sistema entre sus usuarios y sistemas externos. El **nivel 2** muestra las unidades de despliegue, el broker, la proyección y el Lakehouse, con el modo de cada interacción (S o A) y su INT; allí se comprueba que ningún contenedor accede al esquema de otro (PRI-02). El **nivel 3** abre los módulos de mayor complejidad: Inventario, Reposición y Compras, y Flota y Operación.

| Nivel | ID del diagrama | HU | Versión | Fuente | Exportación |
|---|---|---|---|---|---|
| N1 · Contexto | `C4N1-contexto` | HU-11 | **1.0.0** | [.puml](../diagramas/fuente/C4N1-contexto.puml) | [.svg](../diagramas/export/C4N1-contexto.svg) |
| N2 · Contenedores | `C4N2-contenedores` | HU-12 | **1.0.0** | [.puml](../diagramas/fuente/C4N2-contenedores.puml) | [.svg](../diagramas/export/C4N2-contenedores.svg) |
| N3 · Inventario (SVC-02) | `C4N3-SVC02-inventario` | HU-14 | **1.0.0** | [.puml](../diagramas/fuente/C4N3-SVC02-inventario.puml) | [.svg](../diagramas/export/C4N3-SVC02-inventario.svg) |
| N3 · Reposición y Compras (SVC-03) | `C4N3-SVC03-reposicion` | HU-15 | **1.0.0** | [.puml](../diagramas/fuente/C4N3-SVC03-reposicion.puml) | [.svg](../diagramas/export/C4N3-SVC03-reposicion.svg) |
| N3 · Flota y Operación (SVC-05) | `C4N3-SVC05-flota` | HU-19 | **1.0.0** | [.puml](../diagramas/fuente/C4N3-SVC05-flota.puml) | [.svg](../diagramas/export/C4N3-SVC05-flota.svg) |

Convención de nombres, leyenda obligatoria (`ID · vX.Y.Z · AAAA-MM-DD · autor · HU`) y herramientas: [`BACKLOG.md` §7.3](../BACKLOG.md).

---

## 5. Flujos de negocio e interacciones

Tres flujos recorren el negocio: **A** (recepción y actualización de existencias), **B** (riesgo de agotamiento y reposición) y **C** (reserva FEFO, preparación y despacho). Un recorrido integral los une en el flujo vertical de la presentación final. Cada flujo se documenta con un diagrama de actividad (el proceso), un diagrama de secuencia (quién habla con quién y en qué modo) y un documento que los explica. Las entidades con ciclo de vida tienen diagramas de estado. La lista completa de quién se comunica con quién, para qué y con qué efecto está en la matriz de interacciones, que se incluye **por referencia**.

### 5.1 Flujos

| Flujo | HU | Documento | Versión | Actividad | Secuencia |
|---|---|---|---|---|---|
| A · Recepción | HU-16 | [flujo-a-recepcion.md](./flujos/flujo-a-recepcion.md) | **1.0.0** | [`ACT-FA-recepcion`](../diagramas/export/ACT-FA-recepcion.svg) | [`SEC-FA-recepcion`](../diagramas/export/SEC-FA-recepcion.svg) |
| B · Reposición | HU-17 | [flujo-b-reposicion.md](./flujos/flujo-b-reposicion.md) | **1.0.0** | [`ACT-FB-reposicion`](../diagramas/export/ACT-FB-reposicion.svg) | [`SEC-FB-reposicion`](../diagramas/export/SEC-FB-reposicion.svg) |
| C · Reserva y despacho | HU-18 | [flujo-c-reserva-despacho.md](./flujos/flujo-c-reserva-despacho.md) | **1.0.0** | [`ACT-FC-reserva-despacho`](../diagramas/export/ACT-FC-reserva-despacho.svg) | [`SEC-FC-reserva-despacho`](../diagramas/export/SEC-FC-reserva-despacho.svg) |
| Recorrido integral | HU-38 | [recorrido-integral.md](./flujos/recorrido-integral.md) | **1.0.0** | — | [`SEC-VERTICAL-recorrido-integral`](../diagramas/export/SEC-VERTICAL-recorrido-integral.svg) |

### 5.2 Matriz de interacciones (por referencia)

| Artefacto | ID / HU | Versión | Enlace |
|---|---|---|---|
| Matriz de interacciones completa | `matriz-interacciones` · HU-24 | **1.0.0** | [matriz-interacciones.md](./matriz-interacciones.md) |
| Interacciones base (INT-01 a INT-34) y criterios S/A | `ARQUITECTURA_BASE` §7 | contrato 1.0.0 | [§7.2](../ARQUITECTURA_BASE.md#72-interacciones-base-semilla-de-la-matriz-de-interacciones-hu-24) · [§7.1](../ARQUITECTURA_BASE.md#7-reglas-de-interacción-síncrono-vs-asíncrono) |
| Transacciones locales y consistencia eventual (TX, CE) | `ARQUITECTURA_BASE` §7.3 | contrato 1.0.0 | [§7.3](../ARQUITECTURA_BASE.md#73-transacciones-locales-y-consistencia-eventual) |

### 5.3 Diagramas de estado

| Entidades | HU | Diagramas | Versión |
|---|---|---|---|
| Lote, Reserva, Sugerencia, Orden de compra | HU-25 | [`EST-lote`](../diagramas/export/EST-lote.svg) · [`EST-reserva`](../diagramas/export/EST-reserva.svg) · [`EST-sugerencia`](../diagramas/export/EST-sugerencia.svg) · [`EST-orden-compra`](../diagramas/export/EST-orden-compra.svg) | **1.0.0** |
| Pedido, Misión AMR, AMR, Visita de vehículo, Muelle | HU-26 | [`EST-pedido`](../diagramas/export/EST-pedido.svg) · [`EST-mision-amr`](../diagramas/export/EST-mision-amr.svg) · [`EST-amr`](../diagramas/export/EST-amr.svg) · [`EST-visita-vehiculo`](../diagramas/export/EST-visita-vehiculo.svg) · [`EST-muelle`](../diagramas/export/EST-muelle.svg) | **1.0.0** |

### 5.4 Dashboard y vistas de lectura

El dashboard (TRX-01) muestra las vistas VL-01 a VL-07 que mantiene la proyección (TRX-04), con su frescura. No es fuente de verdad: cada acción se valida en el módulo dueño (PRI-07).

| Artefacto | ID / HU | Versión | Enlace |
|---|---|---|---|
| Vistas de lectura y frescura | `ARQUITECTURA_BASE` §8 | contrato 1.0.0 | [§8](../ARQUITECTURA_BASE.md#8-proyección-de-lectura-y-dashboard) |
| Propuesta de dashboard (planta interna y externa) | `propuesta-dashboard` · HU-22, HU-32 | **1.0.0** | [propuesta-dashboard.md](../dashboard/propuesta-dashboard.md) |

---

## 6. Requisitos de calidad

La calidad se expresa con requisitos verificables (RNF), supuestos de carga (SUP-01 a SUP-03) y escenarios de fallo (FAL-01 a FAL-09). Cubren rendimiento, disponibilidad, consistencia, seguridad, escalabilidad y mantenibilidad. La seguridad define siete roles (ROL-01 a ROL-07) y ocho operaciones sensibles (OPS-01 a OPS-08), siempre verificadas en el módulo dueño. La disponibilidad describe cómo se degrada y se recupera el sistema ante cada fallo, y cómo escala cada unidad.

| Artefacto | ID / HU | Versión | Enlace |
|---|---|---|---|
| Requisitos de calidad verificables | `requisitos-calidad` · HU-27 | **1.0.0** | [requisitos-calidad.md](./requisitos-calidad.md) |
| Seguridad: roles, permisos y operaciones sensibles | `seguridad-roles` · HU-28 | **1.0.0** | [seguridad-roles.md](./seguridad-roles.md) |
| Disponibilidad y escalabilidad (FAL-01 a FAL-09, RNF-ESC-01) | `disponibilidad-escalabilidad` · HU-29 | **1.0.0** | [disponibilidad-escalabilidad.md](./disponibilidad-escalabilidad.md) |
| Base de calidad semilla (RNF, SUP, FAL, ROL, OPS) | `ARQUITECTURA_BASE` §11 | contrato 1.0.0 | [§11](../ARQUITECTURA_BASE.md#11-base-de-calidad) |

---

## 7. Decisiones de arquitectura

Las decisiones se registran como ADR con contexto, al menos dos alternativas, decisión y consecuencias. Un ADR aceptado no se edita: se reemplaza con uno nuevo. El **estado vigente de cada ADR** está en su índice, que es la única fuente.

**Índice oficial:** [docs/adr/README.md](../adr/README.md)

### 7.1 Registro de ADR

| ADR | Decisión | HU | Enlace |
|---|---|---|---|
| ADR-001 | Monolito modular evolutivo con unidades de despliegue | HU-20 | [ADR-001](../adr/ADR-001-monolito-modular-evolutivo.md) |
| ADR-002 | Propiedad de datos por módulo con aislamiento por esquema | HU-20 | [ADR-002](../adr/ADR-002-propiedad-datos-por-esquema.md) |
| ADR-003 | La reserva de stock y la selección FEFO pertenecen a Inventario | HU-14 | [ADR-003](../adr/ADR-003-reserva-y-fefo-en-inventario.md) |
| ADR-004 | Comandos síncronos y eventos de negocio con Outbox e idempotencia | HU-20 | [ADR-004](../adr/ADR-004-sincrono-y-eventos-con-outbox.md) |
| ADR-005 | Proyección de lectura (CQRS) con consistencia eventual informada | HU-30 | [ADR-005](../adr/ADR-005-proyeccion-de-lectura-cqrs.md) |
| ADR-006 | La detección de riesgo de agotamiento pertenece a Reposición | HU-17 | [ADR-006](../adr/ADR-006-deteccion-riesgo-en-reposicion.md) |
| ADR-007 | Flota y Operación como unidad de despliegue separada desde el MVP | HU-19 | [ADR-007](../adr/ADR-007-flota-como-unidad-separada.md) |
| ADR-008 | Política de versionamiento de artefactos de diseño | HU-21 | [ADR-008](../adr/ADR-008-versionamiento-de-artefactos.md) |
| ADR-009 | Réplicas locales de datos de referencia en lugar de consultas síncronas | HU-29 | [ADR-009](../adr/ADR-009-replicas-locales-de-referencia.md) |

### 7.2 Patrones y principios de diseño

| Artefacto | ID / HU | Versión | Enlace |
|---|---|---|---|
| Informe de patrones (con referentes y patrones candidatos) | `patrones` · HU-09, HU-30 | **1.0.0** | [patrones.md](../diseno/patrones.md) |
| Evidencia de principios SOLID | `solid` · HU-31 | **1.0.0** | [solid.md](../diseno/solid.md) |
| Diagramas de clases SOLID | `CLS-SOLID-inventario`, `CLS-SOLID-reposicion`, `CLS-SOLID-flota` · HU-31 | **1.0.0** | [`inventario`](../diagramas/export/CLS-SOLID-inventario.svg) · [`reposicion`](../diagramas/export/CLS-SOLID-reposicion.svg) · [`flota`](../diagramas/export/CLS-SOLID-flota.svg) |

---

## 8. Conclusiones

La arquitectura responde a las cinco decisiones que exige el enunciado (contrato §1): qué se despliega por separado, quién es la fuente de cada dato, qué es síncrono y qué asíncrono, qué necesita el dashboard y cómo informa su frescura, y cómo se comportan la seguridad y la disponibilidad ante fallos. Lo hace con un núcleo transaccional simple y consistente donde importa (TX-01 a TX-06), con la flota y la analítica aisladas de ese núcleo, y con eventos con Outbox donde el retraso es aceptable (CE-01 a CE-07). Los límites quedan protegidos por esquemas y credenciales propios, y el diseño ya contempla cómo extraer módulos cuando las señales lo justifiquen.

La auditoría de consistencia (HU-34) confirmó que el diseño es robusto y no presenta escrituras cruzadas. Se detectaron 8 hallazgos menores (como la falta de algunos INT o la unificación de rutas), los cuales fueron resueltos en la versión 1.1.0 del contrato. El paquete de diseño está listo para su entrega final.

### 8.1 Trazabilidad y consistencia

| Artefacto | ID / HU | Versión | Enlace |
|---|---|---|---|
| Matriz de trazabilidad (historias → artefactos) | `matriz-trazabilidad` · HU-35 | **1.0.0** | [matriz-trazabilidad.md](../trazabilidad/matriz-trazabilidad.md) |
| Revisión integral de consistencia (checklist §6.3 y hallazgos) | `revision-consistencia` · HU-34 | **1.0.0** | [revision-consistencia.md](../trazabilidad/revision-consistencia.md) |
| Verificación final contra rúbrica y DoD | `verificacion-entrega` · HU-41 | **1.0.0** | [verificacion-entrega.md](../trazabilidad/verificacion-entrega.md) |

### 8.2 Versionamiento y release

| Artefacto | ID / HU | Versión | Enlace |
|---|---|---|---|
| Política de versionamiento | `politica-versionamiento` · HU-21, HU-36 | **1.0.0** | [politica-versionamiento.md](../versionamiento/politica-versionamiento.md) |
| Historial del paquete de diseño | `CHANGELOG` · HU-01, HU-36 | **1.0.0** | [CHANGELOG.md](../../CHANGELOG.md) |

El paquete de diseño se publica con la etiqueta `diseno-v1.0.0` y el contrato con `contrato-vX.Y.Z` (`BACKLOG.md` §7.1). Antes de publicar, se completan los valores `[completar]` de este documento a partir del `CHANGELOG.md`.

### 8.3 Mantenimiento de esta portada

1. Un artefacto nuevo o con versión nueva se refleja aquí **solo en su fila**, sin copiar su contenido.
2. Si cambia el contrato (`ARQUITECTURA_BASE.md`), se actualiza la versión citada en las secciones 1.4 a 6 y 8.
3. Los ADR se enlazan siempre al archivo y al índice, sin copiar su estado.

---

## Historial de cambios

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-27 | P1 | Versión inicial de la portada navegable de la arquitectura | HU-37 |
