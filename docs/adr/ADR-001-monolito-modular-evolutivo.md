[ADR-001-monolito-modular-evolutivo.md](https://github.com/user-attachments/files/35381851/ADR-001-monolito-modular-evolutivo.md)
# ADR-001: Monolito modular evolutivo con unidades de despliegue

| Campo | Valor |
|---|---|
| Fecha | 2026-11-24 |
| Estado | Propuesto (pasa a *Aceptado* tras la revisión de P7 y el acta del equipo) |
| Responsable | P1 · Líder de arquitectura |
| Historia | [HU-20](../BACKLOG.md) |
| Revisor | P7 |

## Contexto

Smart Inventory cubre seis capacidades (SVC-01 a SVC-06) para un centro de
distribución con 4 muelles, 20 AMR y ~5.000 SKU (SUP-01). La carga normal es de
60 usuarios concurrentes, 1.500 movimientos/hora y 300 pedidos/hora (SUP-02), con
picos de 3× (SUP-03).

Hay que decidir cuántas unidades se despliegan por separado en el MVP. Intervienen
tres hechos del contrato:

- **Las transacciones locales pertenecen a un solo módulo.** TX-01 (recepción),
  TX-02 (reserva) y TX-03 (salida) son todas de SVC-02 y se confirman dentro de su
  esquema. TX-04 es de SVC-03, TX-05 de SVC-04 y TX-06 de SVC-05. Por eso siguen
  siendo locales **sin importar si el módulo vive en un monolito o en su propia
  unidad**: agrupar módulos no es lo que las hace posibles.
- **Lo que sí cruza módulos son las interacciones.** La más exigente es la reserva
  síncrona INT-19 (SVC-04 → SVC-02), que debe resolverse en ≤ 1 s p95 con carga
  normal (RNF-REN-02) y que protege el invariante de que el disponible nunca es
  negativo (RNF-CON-01). El resto de las interacciones entre módulos son
  asíncronas por eventos (§7.2).
- **Flota y Analítica tienen otro perfil.** La telemetría continua con hardware
  (EXT-03) y las cargas analíticas pesadas tienen otros ritmos de cambio y de
  fallo, y no deben competir con el núcleo transaccional (RNF-DIS-03, FAL-03,
  FAL-04).

## Alternativas consideradas

### Alternativa A: Microservicios puros desde el inicio (un despliegue por SVC)

- A favor: escalado y despliegue independientes desde el primer día; fronteras
  físicas que dificultan la erosión de límites.
- En contra: seis unidades multiplican fallos parciales, latencia de red y
  monitoreo sin beneficio para la carga del MVP (SUP-01 a SUP-03). TX-01 a TX-06
  **no se vuelven distribuidas** (cada una sigue dentro de su módulo), pero las
  interacciones entre módulos sí se vuelven llamadas remotas: INT-19 pasaría a
  depender de la red, con tiempo límite y Circuit Breaker en cada pedido, y la
  propia meta de RNF-REN-02 quedaría expuesta a esa latencia desde el MVP. Se
  pagaría ese costo sin que exista todavía la señal que lo justifique (§2.4).

### Alternativa B: Monolito único con todos los módulos, incluidos Flota y Analítica

- A favor: la menor complejidad operativa posible.
- En contra: la telemetría continua y los cálculos pesados competirían por
  recursos con las operaciones transaccionales, y una falla de Flota o de
  Analítica podría arrastrar al inventario. Contradice RNF-DIS-03 y FAL-03.

### Alternativa C: Monolito modular evolutivo con unidades de despliegue (elegida)

- UD-1 agrupa SVC-01 a SVC-04 como monolito modular. SVC-05 (UD-2) y SVC-06 con
  TRX-05 (UD-3) se separan desde el MVP. TRX-01 y TRX-02 forman UD-4, y TRX-04
  forma UD-5.
- Los módulos de UD-1 se comunican como si ya estuvieran separados, mediante
  puertos y eventos (PRI-09), y cada uno tiene esquema propio ([ADR-002](./ADR-002-propiedad-datos-por-esquema.md)).

## Decisión

Se adopta la **Alternativa C**: un monolito modular para el núcleo transaccional
(UD-1) y dos unidades separadas desde el MVP, Flota y Operación (UD-2) y
Analítica (UD-3). Los módulos se extraen a microservicios solo cuando aparezcan
las señales de [ARQUITECTURA_BASE §2.4](../ARQUITECTURA_BASE.md):

1. SVC-02 Inventario, si la reserva incumple RNF-REN-02 de forma sostenida o
   se requiere disponibilidad independiente.
2. SVC-04 Pedidos y Fulfillment, con picos ≥ 3× la carga normal que degraden
   al núcleo, o con nuevos canales de venta.
3. SVC-03 Reposición y Compras, si crecen las integraciones ERP o cambian
   seguido las políticas.

SVC-01 Catálogo y Ubicaciones se mantiene agrupado.

## Consecuencias positivas

- **INT-19 es una llamada interna al proceso** mientras SVC-02 y SVC-04 estén en
  UD-1: no cruza la red y facilita cumplir RNF-REN-02. Al extraer SVC-02, solo
  cambia el adaptador `AdaptadorReservasInventario`; el puerto `GestionReservas`
  y la TX-02 (que ya es local a SVC-02) no cambian.
- El costo operativo es proporcional al MVP: menos fallos parciales, menos latencia de red y menos monitoreo.
- Flota y Analítica no comparten recursos ni fallos con el núcleo ([ADR-007](./ADR-007-flota-como-unidad-separada.md), FAL-03, FAL-04).
- Como los módulos ya usan puertos y eventos (PRI-09), extraer un módulo cambia su despliegue, no sus interacciones.

## Consecuencias negativas

- Los límites entre módulos de UD-1 pueden erosionarse con el tiempo, porque la frontera es lógica y no física.
- UD-1 escala como un bloque: un pico de pedidos escala también Catálogo y Reposición hasta que se extraiga el módulo.
- Un defecto en un módulo de UD-1 puede afectar a los otros tres en el mismo despliegue (RNF-DIS-04 aplica a toda UD-1).
- Exige disciplina para respetar puertos, esquemas y eventos aunque el atajo sea más fácil.

**Mitigación de la erosión de límites:** PRI-02, credenciales por esquema
([ADR-002](./ADR-002-propiedad-datos-por-esquema.md)), la checklist de revisión
cruzada (`BACKLOG.md` §6.3) y RNF-MAN-02.

## Historias y diagramas relacionados

- Historias: [HU-20](../BACKLOG.md) (este ADR, §4.2), [HU-12](../BACKLOG.md), [HU-19](../BACKLOG.md).
- Diagramas: [`C4N2-contenedores` (fuente)](../../diagramas/c4/src/C4N2-contenedores.puml) · [`C4N2-contenedores` (SVG)](../../diagramas/c4/export/C4N2-contenedores.svg): UD-1 a UD-5 y nota de evolución.
- ADR relacionados: [ADR-002](./ADR-002-propiedad-datos-por-esquema.md), [ADR-004](./ADR-004-sincrono-y-eventos-con-outbox.md), [ADR-007](./ADR-007-flota-como-unidad-separada.md).
- Contrato: [ARQUITECTURA_BASE](../ARQUITECTURA_BASE.md) §2.1, §2.3, §2.4, §7.3; PRI-09; RNF-REN-02, RNF-DIS-04, RNF-MAN-02.
