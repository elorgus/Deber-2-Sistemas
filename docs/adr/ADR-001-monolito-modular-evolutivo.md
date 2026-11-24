# ADR-001: Monolito modular evolutivo con unidades de despliegue

| Campo | Valor |
|---|---|
| Fecha | 2026-11-24 |
| Estado | Propuesto (pasa a *Aceptado* tras la revisión de P7 y el acta del equipo) |
| Responsable | P1 · Líder de arquitectura |
| Historia | HU-20 |
| Revisor | P7 |

## Contexto

Smart Inventory cubre seis capacidades (SVC-01 a SVC-06) para un centro de
distribución con 4 muelles, 20 AMR y ~5.000 SKU (SUP-01). La carga normal es de
60 usuarios concurrentes, 1.500 movimientos/hora y 300 pedidos/hora (SUP-02), con
picos de 3× (SUP-03).

Hay que decidir cuántas unidades se despliegan por separado en el MVP. Dos
fuerzas compiten:

- Recepción, reserva y salida necesitan transacciones locales todo o nada
  (TX-01 a TX-03) y una reserva rápida (RNF-REN-02).
- Flota (telemetría continua, hardware EXT-03) y Analítica (cargas pesadas)
  tienen otros ritmos de cambio y de fallo, y no deben competir con el núcleo
  transaccional (RNF-DIS-03).

## Alternativas consideradas

### Alternativa A: Microservicios puros desde el inicio (un despliegue por SVC)

- A favor: escalado y despliegue independientes desde el primer día; fronteras
  físicas que dificultan la erosión de límites.
- En contra: seis unidades multiplican fallos parciales, latencia de red y
  monitoreo sin beneficio para la carga del MVP (SUP-01 a SUP-03). Las
  operaciones que hoy son una transacción local (TX-01 a TX-03) se convertirían
  en coordinación distribuida. La reserva síncrona (INT-19) sería una llamada
  remota con su latencia y su riesgo en cada pedido.

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
  puertos y eventos (PRI-09), y cada uno tiene esquema propio (ADR-002).

## Decisión

Se adopta la **Alternativa C**: un monolito modular para el núcleo transaccional
(UD-1) y dos unidades separadas desde el MVP, Flota y Operación (UD-2) y
Analítica (UD-3). Los módulos se extraen a microservicios solo cuando aparezcan
las señales de ARQUITECTURA_BASE §2.4:

1. SVC-02 Inventario, si la reserva incumple RNF-REN-02 de forma sostenida o
   se requiere disponibilidad independiente.
2. SVC-04 Pedidos y Fulfillment, con picos ≥ 3× la carga normal que degraden
   al núcleo, o con nuevos canales de venta.
3. SVC-03 Reposición y Compras, si crecen las integraciones ERP o cambian
   seguido las políticas.

SVC-01 Catálogo y Ubicaciones se mantiene agrupado.

## Consecuencias positivas

- Las transacciones locales TX-01 a TX-05 son simples y confiables dentro de UD-1.
- El costo operativo es proporcional al MVP: menos fallos parciales, menos latencia de red y menos monitoreo.
- Flota y Analítica no comparten recursos ni fallos con el núcleo (ADR-007, FAL-03, FAL-04).
- Como los módulos ya usan puertos y eventos (PRI-09), extraer SVC-02 cambia su despliegue, no sus interacciones: INT-19 pasa de llamada interna a remota con el mismo puerto `GestionReservas`.

## Consecuencias negativas

- Los límites entre módulos de UD-1 pueden erosionarse con el tiempo, porque la frontera es lógica y no física.
- UD-1 escala como un bloque: un pico de pedidos escala también Catálogo y Reposición hasta que se extraiga el módulo.
- Un defecto en un módulo de UD-1 puede afectar a los otros tres en el mismo despliegue.
- Exige disciplina para respetar puertos, esquemas y eventos aunque el atajo sea más fácil.

**Mitigación de la erosión de límites:** PRI-02, credenciales por esquema (ADR-002),
la checklist de revisión cruzada (`BACKLOG.md` §6.3) y RNF-MAN-02.

## Historias y diagramas relacionados

- Historias: HU-20 (este ADR), HU-12, HU-19.
- Diagramas: `C4N2-contenedores` (UD-1 a UD-5 y nota de evolución).
- ADR relacionados: ADR-002, ADR-004, ADR-007.
- Contrato: ARQUITECTURA_BASE §2.1, §2.3, §2.4; PRI-09; RNF-DIS-04, RNF-MAN-02.