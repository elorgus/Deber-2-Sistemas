# Smart Inventory · Seguridad: roles, permisos y operaciones sensibles

| Campo | Valor |
|---|---|
| Documento | `equipo/P2-EstebanEsp07/HU-28-seguridad-roles.md` (borrador de `docs/arquitectura/seguridad-roles.md`) |
| Versión | **1.0.0** |
| Estado | Propuesto para revisión (PR de HU-28) |
| Autor | P2 · Product Owner (`EstebanEsp07`) |
| Revisor | P3 · Scrum Master y gestor de versiones (`docs/BACKLOG.md` §1.2) |
| Fecha | 2026-11-24 |
| HU | **HU-28** Seguridad: roles, permisos y operaciones sensibles · Sprint 3 · rúbrica: Requisitos de calidad |
| Épica | **EP-08** Seguridad y acceso (EXT-04, todos los SVC) |
| Historias | HN-24 (Must); relacionadas: HN-06, HN-11, HN-16 |
| Requisitos | RNF-SEG-01, RNF-SEG-02 (semilla, `ARQUITECTURA_BASE` §11.2) |
| Contrato de referencia | `docs/ARQUITECTURA_BASE.md` v1.0.0 (§3, §4, §7.2, §9, §11.3, §11.4) |

> Todos los identificadores son los del contrato. No se crearon identificadores nuevos. Lo que el contrato no define aparece como **PROPUESTA** y no se usa como oficial. Este documento no incluye APIs, mensajes, configuraciones de herramientas, CI/CD ni pruebas.

---

## 1. Principios

Tomados de `ARQUITECTURA_BASE` §11.4 y desarrollados en las secciones indicadas.

| # | Principio | Qué significa | Sección |
|---|---|---|---|
| 1 | Identidad centralizada en EXT-04 | Nadie administra credenciales dentro de Smart Inventory; EXT-04 autentica y entrega los roles (INT-02) | §6 |
| 2 | Autorización en la capacidad dueña | Ocultar un botón no basta: el módulo dueño del dato verifica el rol en cada operación | §6 |
| 3 | Mínimo privilegio | Cada rol solo hace lo que su trabajo exige | §3 |
| 4 | Separación de funciones | Quien aprueba una OC no registra su recepción | §8 |
| 5 | Identidad propia por módulo | Cada módulo se identifica ante otro en las llamadas entre módulos | §9 |
| 6 | Secretos fuera del repositorio | 0 secretos en el repositorio (RNF-SEG-02) | §10 |

---

## 2. Roles

| Rol | Perfil | Puede (contrato, §11.4) | HN de referencia |
|---|---|---|---|
| ROL-01 | Operador de bodega | Registrar recepciones y transferencias, confirmar preparación, consultar | HN-03, HN-04, HN-12, HN-15, HN-05 |
| ROL-02 | Supervisor de bodega | Todo lo de ROL-01 + ajustes, cambio de estado de lote, umbrales de vencimiento | HN-07, HN-06 |
| ROL-03 | Comprador | Políticas de reposición, aprobar o rechazar sugerencias, proveedores, OC | HN-08 a HN-11, HN-23 |
| ROL-04 | Operador de patio y flota | Llegadas y salidas de vehículos, muelles, misiones AMR | HN-17, HN-18 |
| ROL-05 | Supervisor de operaciones | Consulta global, registrar y cancelar pedidos, prioridades | HN-13, HN-14, HN-19 a HN-21, HN-16 |
| ROL-06 | Analista | Solo lectura y analítica | HN-22 |
| ROL-07 | Administrador | Catálogo, ubicaciones, usuarios y roles (no opera inventario) | HN-01, HN-02, HN-24 |

---

## 3. Matriz ROL × operación (CA1)

**Leyenda:** ✔ permitido · ✔● permitido con control (auditoría, motivo o segunda aprobación; ver §4) · L solo consulta · S solo la identidad de un módulo, nunca una persona · — no permitido.
**Origen:** *contrato* = definido en §3 o §11.4; *PROPUESTA* = deducido del mínimo privilegio, a validar en HU-27 y HU-34.

| # | Operación | SVC | ROL-01 | ROL-02 | ROL-03 | ROL-04 | ROL-05 | ROL-06 | ROL-07 | Origen |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Registrar o actualizar producto (alta o baja: OPS-06) | SVC-01 | — | — | — | — | — | — | ✔● | contrato |
| 2 | Definir bodega, zona o ubicación (OPS-06) | SVC-01 | — | — | — | — | — | — | ✔● | contrato |
| 3 | Bloquear ubicación | SVC-01 | — | — | — | — | — | — | ✔● | contrato; control: PROPUESTA |
| 4 | Consultar catálogo | SVC-01 | L | L | L | L | L | L | ✔ | PROPUESTA |
| 5 | Registrar recepción | SVC-02 | ✔ | ✔ | — | — | — | — | — | contrato |
| 6 | Registrar transferencia | SVC-02 | ✔ | ✔ | — | — | — | — | — | contrato |
| 7 | Registrar ajuste (OPS-01) | SVC-02 | — | ✔● | — | — | — | — | — | contrato |
| 8 | Cambiar estado de lote: cuarentena o liberación (OPS-02) | SVC-02 | — | ✔● | — | — | — | — | — | contrato (ver §12, obs. 1) |
| 9 | Configurar umbrales de vencimiento | SVC-02 | — | ✔● | — | — | — | — | — | contrato; control: PROPUESTA |
| 10 | Consultar existencias y lotes (INT-15) | SVC-02 | L | L | L | — | L | L | — | contrato (01, 02, 05, 06); 03: PROPUESTA |
| 11 | Solicitar reserva FEFO (INT-19) | SVC-02 | — | — | — | — | — | — | — | S: solo SVC-04 |
| 12 | Liberar reserva y registrar salida (por EVT-28 y EVT-27) | SVC-02 | — | — | — | — | — | — | — | S: reacción a eventos de SVC-04 |
| 13 | Configurar política de reposición (OPS-04) | SVC-03 | — | — | ✔● | — | — | — | — | contrato |
| 14 | Consultar sugerencias | SVC-03 | — | — | L | — | L | L | — | contrato (03, 05, 06) |
| 15 | Aprobar sugerencia y emitir OC (OPS-03) | SVC-03 | — | — | ✔● | — | — | — | — | contrato |
| 16 | Rechazar sugerencia | SVC-03 | — | — | ✔● | — | — | — | — | contrato; control: PROPUESTA |
| 17 | Gestionar proveedores | SVC-03 | — | — | ✔● | — | — | — | — | contrato; control: PROPUESTA |
| 18 | Consultar OC | SVC-03 | L | L | L | — | L | L | — | contrato (03, 05, 06); 01 y 02: PROPUESTA (recepción contra OC, HN-12) |
| 19 | Registrar pedido (INT-17; la importación INT-18 la hace el adaptador de EXT-01) | SVC-04 | — | — | — | — | ✔ | — | — | contrato |
| 20 | Cancelar pedido (OPS-05) | SVC-04 | — | — | — | — | ✔● | — | — | contrato |
| 21 | Consultar pedido | SVC-04 | L | L | — | L | ✔ | L | — | contrato (05, 06); resto: PROPUESTA |
| 22 | Confirmar preparación (INT-25) | SVC-04 | ✔ | ✔ | — | — | — | — | — | contrato |
| 23 | Registrar llegada o salida manual de vehículo (INT-33) | SVC-05 | — | — | — | ✔ | — | — | — | contrato |
| 24 | Asignar muelle (INT-33) | SVC-05 | — | — | — | ✔ | — | — | — | contrato |
| 25 | Reasignar o abortar misión AMR (OPS-08) | SVC-05 | — | — | — | ✔● | — | — | — | contrato |
| 26 | Consultar flota y patio | SVC-05 | — | — | — | ✔ | L | L | — | contrato |
| 27 | Consultar indicadores históricos (INT-34) | SVC-06 | — | — | — | — | L | L | — | contrato |
| 28 | Consultar pronóstico | SVC-06 | — | — | L | — | L | L | — | contrato (05, 06); 03: HN-23 |
| 29 | Gestionar usuarios y roles (OPS-07, en EXT-04) | EXT-04 | — | — | — | — | — | — | ✔● | contrato |
| 30 | Ver dashboard VL-01 a VL-06 | TRX-01 | L | L | L | L | L | L | — | PROPUESTA |
| 31 | Ver indicadores históricos VL-07 | TRX-01 | — | — | — | — | L | L | — | PROPUESTA |

**Notas**

- ROL-02 hereda todo lo de ROL-01 (§11.4).
- ROL-07 no opera inventario: no tiene ninguna operación de SVC-02 a SVC-06.
- «Prioridades» de ROL-05 (§11.4) no corresponden a ninguna operación de §3; ver §12, observación 5.
- En el dashboard, cada botón exige el rol de la fila que le corresponde: aprobar o rechazar (13 a 16), ver detalle de lote (10), cancelar pedido (20), reasignar misión (25), registrar llegada y asignar muelle (23 y 24).

---

## 4. Operaciones sensibles OPS-01 a OPS-08

| ID | Operación | Rol | Dueño que la autoriza | Control del contrato | Qué se audita | Evento o interacción relacionada |
|---|---|---|---|---|---|---|
| OPS-01 | Registrar ajuste de inventario | ROL-02 | SVC-02 | Motivo obligatorio y auditoría | Quién, cuándo, SKU · ubicación · lote, cantidad antes y después, motivo | EVT-07 AjusteRegistrado |
| OPS-02 | Cambiar estado de lote | ROL-02 | SVC-02 | Auditoría | Quién, cuándo, lote, estado anterior y nuevo, motivo | Se refleja en EVT-05 (§9) |
| OPS-03 | Aprobar sugerencia y emitir OC | ROL-03 | SVC-03 | Auditoría; sobre el umbral, segunda aprobación (§5) | Quién aprueba (y quién da la segunda aprobación), cuándo, sugerencia, monto, OC resultante | INT-12 · EVT-17, EVT-19 · TX-04 |
| OPS-04 | Modificar política de reposición | ROL-03 | SVC-03 | Auditoría | Quién, cuándo, SKU, valores anteriores y nuevos | — |
| OPS-05 | Cancelar pedido | ROL-05 | SVC-04 | Motivo obligatorio | Quién, cuándo, pedido, estado previo, motivo | EVT-28 PedidoCancelado |
| OPS-06 | Alta o baja de SKU o ubicación | ROL-07 | SVC-01 | Auditoría | Quién, cuándo, SKU o ubicación, tipo de cambio | EVT-01, EVT-02, EVT-03 |
| OPS-07 | Gestionar usuarios y roles | ROL-07 | EXT-04 | Auditoría | Quién, cuándo, usuario afectado, rol anterior y nuevo | INT-02 (lectura de roles) |
| OPS-08 | Reasignar o abortar misión AMR | ROL-04 | SVC-05 | Auditoría | Quién, cuándo, misión, AMR origen y destino, motivo | INT-33 · EVT-29 (si se reasigna) |

Registro mínimo de auditoría exigido por RNF-SEG-01: **quién, cuándo, qué y motivo**. Donde el contrato no exige motivo (OPS-02, OPS-03, OPS-04, OPS-06, OPS-07, OPS-08), el campo existe pero puede quedar vacío; se propone exigirlo en OPS-02 y OPS-08 (HU-27).

---

## 5. Umbral de monto de OPS-03 (CA2)

OPS-03 exige segunda aprobación sobre un umbral de monto «definido en HU-28» (§11.4).

### 5.1 Definición

| Elemento | Definición |
|---|---|
| **Umbral** | **USD 5.000** por OC (monto total = suma de cantidad × costo unitario de las líneas de la sugerencia) |
| Quién da la segunda aprobación | Otro usuario con **ROL-03**, distinto del que aprobó primero |
| Acumulado anti-fraccionamiento | Si las aprobaciones al mismo proveedor en 24 h suman más de USD 5.000, la que supera el umbral requiere segunda aprobación |
| Si el monto es igual o menor al umbral | Una sola aprobación de ROL-03; TX-04 sin cambios |
| Si la sugerencia es reemplazada antes de la segunda aprobación | La aprobación pendiente se anula y la nueva sugerencia empieza de cero |
| Auditoría | Se registran ambas aprobaciones (§4) |

### 5.2 Justificación

- **Falta de datos de costo.** SUP-01 a SUP-03 solo dan volúmenes (5.000 SKU, 1.500 movimientos/hora, 300 pedidos/hora); no hay precios. El valor es un **supuesto del Product Owner**, a confirmar con la dirección de operaciones (`HU-02-vision.md` §2.1, interesado 8) y a recalibrar con datos reales.
- **Riesgo.** Con ~5.000 SKU activos, un solo comprador puede emitir muchas OC al día; el umbral limita cuánto dinero puede comprometer una persona sin un segundo par de ojos.
- **Fluidez.** La reposición es una decisión de horas o días (CE-03); un umbral muy bajo convertiría la segunda aprobación en un cuello de botella para reposiciones rutinarias.
- **Moneda.** USD, por ser la moneda del contexto del proyecto (USFQ); es un supuesto.
- **Criterio de calibración.** El umbral debe dejar la segunda aprobación en una minoría de las OC (meta de referencia: ≤ 10 %); se revisa en HU-34 con los datos del Sprint 4.

| Alternativa evaluada | Ventaja | Problema |
|---|---|---|
| USD 1.000 | Máximo control | La segunda aprobación se vuelve rutina y se aprueba sin revisar |
| **USD 5.000** | Equilibrio entre control y fluidez | Requiere recalibrar con costos reales |
| USD 25.000 | Casi sin fricción | Una persona puede comprometer montos altos sin control |

---

## 6. Autorización en la capacidad dueña (CA3)

**Regla:** la pantalla nunca autoriza. TRX-02 autentica y propaga la identidad y los roles del usuario; **el módulo dueño verifica el rol antes de ejecutar** (§11.4).

1. El usuario inicia sesión y TRX-02 obtiene su identidad y roles desde EXT-04 (INT-02).
2. El usuario ejecuta una acción en TRX-01; TRX-02 la recibe (INT-01) y la enruta al dueño de la operación.
3. TRX-02 envía al dueño la identidad del usuario, sus roles y su propia identidad como módulo.
4. El dueño comprueba que el rol figura en la matriz de §3 para esa operación. Si no, la rechaza y registra el intento.
5. Si el rol es válido, aplica los controles de §4 (motivo, segunda aprobación) y ejecuta la operación en su transacción local (TX-01 a TX-06).
6. Registra la auditoría y publica el evento correspondiente por Outbox.

| Aspecto | Comportamiento |
|---|---|
| Botones ocultos en la UI | Es comodidad de uso, no seguridad |
| Dashboard *Desactualizada* | Las acciones siguen disponibles porque siempre se validan en el dueño (§8.2, PRI-07) |
| Rechazo | El usuario ve un mensaje claro de falta de permiso; el dueño registra el intento (ejemplo: HN-24, ROL-06 intenta aprobar una sugerencia y SVC-03 lo rechaza) |
| Sin rol válido en la sesión | Ninguna capacidad ejecuta la operación |

---

## 7. Auditoría

| Aspecto | Definición |
|---|---|
| Qué se registra | Quién, cuándo, qué y motivo (RNF-SEG-01), según §4 |
| Dónde | En el módulo dueño de la operación; la de OPS-07 queda en EXT-04. No hay un registro central nuevo |
| Qué es | Registro de solo adición: no se edita ni se borra; una corrección es un nuevo registro |
| Intentos rechazados | También se registran (quién, cuándo, qué intentó) |
| Quién la consulta | **PROPUESTA:** ROL-07, para revisión administrativa; a validar en HU-27 y HU-34 |

---

## 8. Separación de funciones

| Regla | Qué evita | Cómo se garantiza |
|---|---|---|
| **Aprobar una OC ≠ registrar su recepción** (contrato) | Que una persona compre y reciba | ROL-03 no tiene operaciones de recepción (3, filas 5 a 6) y ROL-01 y ROL-02 no aprueban; **ROL-03 es incompatible con ROL-01 y ROL-02 en el mismo usuario** |
| Segunda aprobación de OC por otra persona | Que una persona apruebe dos veces su propia compra | Segunda aprobación de un ROL-03 distinto (§5) |
| Nadie se asigna roles a sí mismo | Escalamiento de privilegios | OPS-07 rechaza que ROL-07 modifique sus propios roles (**PROPUESTA**) |
| El Administrador no opera inventario (contrato) | Que quien define el catálogo también mueva stock | ROL-07 es incompatible con ROL-01 y ROL-02 en el mismo usuario |

---

## 9. Identidad entre módulos

Cada módulo tiene identidad propia y verificable ante los demás, además de la identidad del usuario cuando la hay. El dueño verifica **ambas**: qué módulo llama y qué usuario actúa.

| Llamada | Módulo que llama | Qué verifica el dueño |
|---|---|---|
| INT-19 Solicitar reserva FEFO | SVC-04 → SVC-02 | Que la llamada viene de SVC-04; ninguna persona ni otro módulo puede reservar |
| INT-06, INT-12, INT-15, INT-17, INT-25, INT-33, INT-34 (comandos y consultas) | TRX-02 → dueño | Que viene de TRX-02 **y** que el usuario propagado tiene el rol de la matriz (§3) |
| Publicación de eventos | Cada módulo → TRX-03 | Que cada módulo publica solo los eventos de los que es productor (§6.2) |
| Reacción a EVT-27 y EVT-28 | SVC-02 consume | Que el evento proviene de SVC-04, su único productor |
| Adaptadores de sistemas externos (INT-14, INT-18, INT-22, INT-23, INT-27, INT-28) | EXT-01 a EXT-05 | Identidad propia por sistema externo; ninguno recibe permisos de usuario |
| Proyección de lectura | TRX-04 | Solo lectura y consumo de eventos; no recibe comandos (PRI-07) |

---

## 10. Manejo de secretos

- **Qué es un secreto aquí:** credenciales de cada módulo, de los adaptadores a EXT-01 a EXT-05 y del acceso a EXT-04.
- **Regla (RNF-SEG-02):** 0 secretos en el repositorio. Ni en documentos, ni en diagramas, ni en el historial de ramas.
- **Verificación:** la checklist de cada PR (`docs/BACKLOG.md` §6.3) incluye «sin secretos ni datos personales reales».
- **Dónde viven:** en un almacén de secretos administrado por Desarrollo de Software Moderno; su herramienta y configuración quedan fuera de este deber.
- **Si se filtra uno:** se considera comprometido, se reemplaza y se revisa la auditoría. El contenido del repositorio no se «limpia» sin reemplazar el secreto.
- **Datos de ejemplo:** los documentos solo usan datos ilustrativos.

---

## 11. Comportamiento ante fallos

| Fallo | Efecto en seguridad |
|---|---|
| FAL-07 EXT-04 IdP no disponible | No hay nuevos inicios de sesión; las sesiones vigentes continúan hasta expirar. El dueño sigue verificando roles en cada operación |
| FAL-01 / FAL-06 Broker o proyección caídos | Dashboard marcado *Desactualizada*; las acciones siguen disponibles y se validan en el dueño |
| Sesión expirada | La acción se rechaza y el usuario vuelve a autenticarse; no se pierden los datos del formulario (FAL-02) |

---

## 12. Observaciones y propuestas de cambio

No cambian el contrato. Cualquier ajuste se tramita con PR (y ADR si aplica) según `ARQUITECTURA_BASE` §0.3.

1. **OPS-02 y umbrales de vencimiento no figuran en las operaciones de SVC-02** (§3.2 lista recepción, transferencia, ajuste, consulta y reserva). **PROPUESTA:** agregarlas a §3.2 (P4, HU-14).
2. **OPS-05 dice «pedido en preparación»**, pero EVT-28 y HN-16 permiten cancelar desde cualquier estado previo a *EnDespacho* (§9). **PROPUESTA:** que el motivo obligatorio aplique a toda cancelación anterior al despacho.
3. **Sin INT para cancelar pedido, ni para OPS-01, OPS-02, OPS-04 y OPS-06** en §7.2. **PROPUESTA:** que HU-24 (P6) las agregue como interacciones síncronas desde TRX-02 al dueño.
4. **Estado de sugerencia para la segunda aprobación.** §9 no tiene un estado «pendiente de segunda aprobación». **PROPUESTA:** la primera aprobación se guarda como registro de auditoría y la sugerencia sigue *Generada* hasta la segunda; TX-04 se ejecuta solo con la aprobación completa.
5. **«Prioridades» de ROL-05** (§11.4) no corresponde a ninguna operación de §3. **PROPUESTA:** definir si es una operación de SVC-04 o retirarla del rol.
6. **Umbral de OPS-03.** El valor es supuesto del PO (§5.2); registrar su validación en HU-27.

---

## 13. Trazabilidad

| Criterio (`docs/BACKLOG.md` §4.3) | Dónde se cumple |
|---|---|
| CA1. Matriz ROL × operación de cada SVC, incluidas OPS-01 a OPS-08 | §3 y §4 |
| CA2. Umbral de monto de OPS-03 definido y justificado | §5 |
| CA3. Autorización en la capacidad dueña, auditoría, separación de funciones, identidad entre módulos y secretos fuera del repositorio, sin configuraciones de herramientas | §6, §7, §8, §9 y §10 |

Épica EP-08 · HN-24 · RNF-SEG-01 y RNF-SEG-02 · Rúbrica *Requisitos de calidad*. Insumo de HU-27 (requisitos de calidad), HU-24 (matriz de interacciones), HU-37 y HU-35.

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-24 | P2 | Versión inicial: matriz ROL × operación, OPS-01 a OPS-08, umbral de OPS-03 y principios de seguridad | HU-28 |
