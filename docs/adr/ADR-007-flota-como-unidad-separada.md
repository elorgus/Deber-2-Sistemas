# ADR-007 · Flota y Operación como unidad de despliegue separada desde el MVP

| Campo | Valor |
|---|---|
| Estado | Propuesto |
| Fecha | 2026-11-24 |
| Responsable · Revisor | P6 · P5 |
| HU relacionada | HU-19 |
| Elementos del contrato afectados | `SVC-05`, `UD-2`, `EVT-24`, `EVT-26`, `EVT-28` a `EVT-36`, `INT-21`, `INT-24`, `INT-26`, `INT-29`, `INT-31`, `INT-33`, `RNF-ESC-01`, `RNF-DIS-04`, `FAL-03`, `CE-02`, `CE-05`, `PRI-09` |
| Diagramas relacionados | `C4N3-SVC05-flota`, `ACT-FC-reserva-despacho`, `SEC-FC-reserva-despacho` |

## 1. Contexto

Smart Inventory arranca como un monolito modular para el núcleo transaccional (`ADR-001`), con dos unidades separadas desde el MVP: Flota y Analítica (§2.1). El núcleo, `UD-1`, agrupa `SVC-01` a `SVC-04` porque comparten ritmo de cambio y necesitan transacciones locales simples (§2.3).

`SVC-05` Flota y Operación tiene características distintas al núcleo (§2.3 y §3.5):

- Opera la flota interna (AMR, misiones, rutas, telemetría) y la operación externa (vehículos, muelles, patio, acceso).
- Recibe telemetría continua y se integra con hardware (`EXT-03`), además de `EXT-02` y `EXT-05`, siempre mediante adaptadores.
- Tiene fallos propios: un AMR sin señal por más de 10 s no recibe misiones (`FAL-03`).
- No es responsable de los estados del pedido (`SVC-04`) ni de las existencias (`SVC-02`), y no escribe en sus datos (`PRI-02`).
- Su carga crece con el número de AMR y de vehículos activos (`RNF-ESC-01`), no con los usuarios ni los pedidos por hora.

Hay que decidir si `SVC-05` se despliega junto al núcleo o por separado en el MVP, de modo que su caída no afecte inventario (§2.3).

## 2. Opciones consideradas (mínimo 2)

### Opción A · `SVC-05` dentro de `UD-1` (monolito modular)
- Descripción: `SVC-05` se agrupa con `SVC-01` a `SVC-04` en un solo despliegue.
- Ventajas: un solo despliegue y un solo punto de monitoreo; transacciones locales simples; menor costo operativo, que es el argumento central de `ADR-001` (§2.1).
- Desventajas: la telemetría continua, la integración con hardware y sus fallos compartirían despliegue con inventario y pedidos, lo contrario de lo que pide §2.3. `SVC-05` escalaría con `UD-1` (usuarios concurrentes, pedidos por hora) en lugar de hacerlo según AMR y vehículos activos (`RNF-ESC-01`). Un problema de hardware o de telemetría pondría en riesgo la disponibilidad de `UD-1` (`RNF-DIS-04`).

### Opción B · `SVC-05` como `UD-2` separada desde el MVP (elegida)
- Descripción: `SVC-05` se despliega como unidad propia y se comunica con el núcleo solo mediante puertos y eventos (`PRI-09`).
- Ventajas: aislamiento de fallos, escala independiente y cumplimiento de §2.3.
- Desventajas: una unidad más que operar y monitorear, y consistencia eventual en toda interacción con el núcleo.

## 3. Decisión

Se elige la **Opción B**: `SVC-05` Flota y Operación se despliega como **`UD-2`, separada desde el MVP**. La razón principal es que sus fallos de hardware y telemetría no deben afectar inventario (§2.3), y que escala con los AMR y vehículos activos y no con el núcleo (`RNF-ESC-01`). Se comunica con `SVC-04` y `SVC-02` solo por eventos a través de `TRX-03`, y mantiene su propio esquema sin escribir en datos de otros módulos (`PRI-01`, `PRI-02`).

## 4. Consecuencias

**Positivas**
- Aislamiento de fallos: un AMR sin señal no afecta a `SVC-02` ni a `SVC-04` (`FAL-03`), y si `SVC-05` no responde la tarea queda en cola y es posible el picking manual (`INT-21`).
- Escala independiente: `UD-2` escala con el número de AMR y vehículos activos (`RNF-ESC-01`) sin escalar `UD-1`.
- La telemetría continua y el hardware no compiten con el núcleo transaccional por recursos (§2.1).
- Las interacciones no se rediseñan: los límites ya están trazados con puertos y eventos (`PRI-09`).

**Negativas o riesgos**
- Más operación: dos unidades de despliegue que operar y monitorear en lugar de una, justo el costo que `ADR-001` busca contener. Se acepta solo para Flota y Analítica.
- Consistencia eventual: toda interacción con el núcleo es asíncrona. La tarea espera en cola (`INT-21`), el despacho espera (`INT-26`), el cierre del pedido puede diferirse (`INT-29`) y el stock sigue reservado mientras se cancelan misiones (`INT-31`, `CE-05`).
- Sin transacción compartida: pedido y flota se coordinan por coreografía con compensación (por ejemplo, `EVT-28` hacia `INT-31`), no por transacciones distribuidas.
- Réplicas locales atrasadas: las de ubicaciones (`EVT-03`) y órdenes de compra (`EVT-19`) pueden ir detrás de su fuente (`CE-02`, ≤ 5 s para ubicaciones), con rutas calculadas sobre un layout levemente desactualizado.
- Dependencia del broker: si `TRX-03` no está disponible, los eventos esperan en el outbox y se publican en orden al volver (`FAL-01`); mientras tanto, la coordinación con el núcleo se retrasa.

## 5. Notas

Un ADR *Aceptado* no se edita: si la decisión cambia, se crea un ADR nuevo que lo reemplaza (`docs/VERSIONAMIENTO.md` §6).
Fuera de alcance: endpoints, payloads, CI/CD y pruebas.

ADR relacionados: `ADR-001` (monolito modular con unidades de despliegue), `ADR-004` (comandos síncronos y eventos con Outbox e idempotencia) y `ADR-009` (réplicas locales de datos de referencia). `SVC-05` no aparece en la ruta de evolución de §2.4 porque ya se despliega separada desde el MVP.
