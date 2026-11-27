# ADR-009: Réplicas locales de datos de referencia en lugar de consultas síncronas

| Campo | Valor |
|---|---|
| Fecha | 2026-11-27 |
| Estado | Propuesto (pasa a *Aceptado* tras la aprobación de P7 en el PR y el acta del equipo) |
| Responsable | P1 · Líder de arquitectura |
| Historia | [HU-29](../BACKLOG.md) |
| Revisor | P7 |

## Contexto

SVC-02 Inventario valida cada recepción contra los datos de SVC-01 Catálogo y Ubicaciones: que el SKU exista y no esté descontinuado, que la ubicación esté activa y no bloqueada, y qué reglas de lote y vencimiento aplican (§3.1). También necesita datos de catálogo para la reserva FEFO, por ejemplo la vida útil mínima de despacho (§3.2). Otros módulos tienen necesidades parecidas: SVC-03 y SVC-04 usan el catálogo, y SVC-05 en UD-2 usa las ubicaciones y el layout para calcular rutas (§3).

El dueño de esos datos es SVC-01 (PRI-01) y ningún módulo puede leer su esquema (PRI-02). La pregunta es **cómo accede cada módulo a esos datos de referencia**. Intervienen cuatro hechos del contrato:

- **Rendimiento.** La reserva FEFO debe resolverse en ≤ 1 s p95 con carga normal y ≤ 2 s p95 en pico (RNF-REN-02). La recepción (INT-06) es una operación que un usuario espera (S1).
- **Disponibilidad.** Con el broker caído, SVC-02 debe seguir aceptando recepciones y reservas (RNF-DIS-01). Si SVC-05 (UD-2) depende de otra unidad para sus datos, el aislamiento de ADR-007 se debilita.
- **Límite de saltos síncronos.** Máximo dos por operación: TRX-02 → módulo dueño → como mucho un módulo más (§7.1). El pedido registrado en SVC-04 ya usa dos saltos (TRX-02 → SVC-04 → SVC-02).
- **Mismo contrato en monolito y microservicios** (PRI-09): una decisión válida solo mientras SVC-01 y SVC-02 compartan UD-1 no sirve.

## Alternativas consideradas

### Alternativa A: Consulta síncrona a SVC-01 cada vez que otro módulo necesita validar

SVC-02, SVC-03, SVC-04 y SVC-05 llaman a una operación de SVC-01 en cada validación. En §7.2 **no existe un INT para esto**: sería una interacción nueva **[PROPUESTA]**.

- **A favor:** el dato siempre es el vigente; no hay datos duplicados ni retraso entre módulos (criterio S3, "dato exacto de la fuente de verdad").
- **Rendimiento (en contra):** agrega un salto por cada recepción y por cada reserva. Para un pedido registrado, la cadena sería TRX-02 → SVC-04 → SVC-02 → SVC-01, es decir, **tres saltos síncronos**, y rompe el límite de §7.1. Cada salto consume parte del presupuesto de 1 s de RNF-REN-02.
- **Disponibilidad (en contra):** SVC-01 se convierte en una dependencia en línea de todos los módulos. Si falla, no se puede recibir ni reservar, aunque Inventario esté sano. En SVC-05 (UD-2) el efecto sería peor, porque calcular rutas dependería de una llamada remota a UD-1 incluso en el MVP.
- **Acoplamiento (en contra):** SVC-02 queda acoplado en tiempo de ejecución a SVC-01 en vez de a sus hechos. La extracción de módulos (§2.4) convierte llamadas internas en remotas y obliga a agregar tiempo límite y Circuit Breaker a cada una.
- **Necesidad real (en contra):** validar una recepción no exige exactitud al milisegundo. El contrato ya acepta que un SKU nuevo tarde unos segundos en ser recibible (CE-02).

### Alternativa B: Réplica local de solo lectura alimentada por eventos (elegida)

Cada módulo que necesita datos de referencia mantiene una **copia de solo lectura** que actualiza al consumir los eventos del dueño (PRI-04). El dueño sigue siendo SVC-01.

- **A favor:** validación local, sin salto y sin depender de SVC-01 en línea. Reutiliza interacciones que ya existen (INT-04, INT-16) y los mecanismos ya decididos en [ADR-004](./ADR-004-sincrono-y-eventos-con-outbox.md): eventos con Outbox, consumidores idempotentes y orden por entidad.
- **En contra:** consistencia eventual (CE-02: ≤ 5 s), datos duplicados en cada módulo y lógica de sincronización por mantener.

### Alternativa C: Leer directamente el esquema de SVC-01

Los módulos consultan las tablas de Catálogo, con joins o claves foráneas.

- **A favor:** el dato es siempre el vigente; no hay latencia de red ni sincronización.
- **En contra:** viola PRI-02 y PRI-03, y RNF-MAN-02 exige 0 dependencias de datos cruzadas. Impide extraer módulos. Ya se descartó como parte de [ADR-002](./ADR-002-propiedad-datos-por-esquema.md).

## Decisión

Se adopta la **Alternativa B**: **réplicas locales de solo lectura de los datos de referencia, alimentadas por eventos** (PRI-04).

1. **Qué se replica** (según §3, columna "Réplicas locales"):

| Módulo | Réplica | Evento que la alimenta |
|---|---|---|
| SVC-02 | Productos y ubicaciones; OC emitidas pendientes de recepción | EVT-01 a EVT-03; EVT-19, EVT-20 |
| SVC-03 | Catálogo; disponible agregado por SKU; demanda no atendida; pronóstico de demanda | EVT-01, EVT-02; EVT-05; EVT-09; EVT-37 |
| SVC-04 | Catálogo; muelle asignado | EVT-01, EVT-02; EVT-34 |
| SVC-05 | Ubicaciones y layout; OC emitidas | EVT-03; EVT-19 |

   SVC-01 y SVC-06 no tienen réplicas (§3.1, §3.6).
2. **Cómo se accede:** a través del puerto de salida de solo lectura `ReplicaCatalogo` (§10), de modo que el dominio no depende de cómo se alimente la réplica (DIP, PRI-08).
3. **Cómo se alimenta:** por eventos publicados por el dueño vía Outbox (PRI-06, INT-04 y INT-16), con `ConsumidorIdempotente` y orden por entidad. El retraso máximo objetivo es de **5 s** (CE-02).
4. **Qué nunca se decide con una réplica:** los invariantes críticos. El disponible y las reservas se verifican siempre en SVC-02 dentro de TX-02 (RNF-CON-01, S2). La réplica de "disponible agregado por SKU" en SVC-03 solo sirve para evaluar el riesgo de agotamiento, que tolera retraso (CE-03, PRI-05).
5. **Cuándo sí se consulta al dueño:** si una operación necesita el dato exacto de la fuente de verdad en ese momento (S3), usa una operación síncrona del dueño y no la réplica. Es el criterio ya vigente en INT-15.

## Consecuencias positivas

- **Rendimiento:** validar recepciones y aplicar FEFO no agrega saltos síncronos. Se respeta el límite de dos saltos y se protege RNF-REN-02.
- **Disponibilidad:** SVC-01 o el broker caídos no impiden recibir ni reservar con los datos ya replicados (RNF-DIS-01). SVC-05 calcula rutas sin depender de UD-1, lo que refuerza ADR-007.
- **Menor acoplamiento:** SVC-02 depende de los **hechos** publicados por SVC-01, no de su disponibilidad. Extraer cualquier módulo (§2.4) no cambia cómo consulta sus datos de referencia (PRI-09).
- **Coherencia de diseño:** no agrega una interacción nueva: usa INT-04, que ya existe, y los mecanismos de [ADR-004](./ADR-004-sincrono-y-eventos-con-outbox.md).
- **Propiedad intacta:** la réplica es solo de lectura. El único dueño y el único que escribe sigue siendo SVC-01 (PRI-01, PRI-02).

## Consecuencias negativas

- **Consistencia eventual (CE-02):** un SKU recién creado tarda hasta unos segundos en ser recibible. En sentido inverso, un SKU recién descontinuado o una ubicación recién bloqueada podría aceptar una recepción durante esa ventana. Ese caso no está tratado en CE-02. Se asume como riesgo residual y se corrige con un ajuste, porque los movimientos nunca se editan (§3.2).
- **Datos duplicados:** cuatro módulos mantienen copias de referencia y cada copia necesita su propia sincronización.
- **Con el broker caído (FAL-01), las réplicas se atrasan:** los módulos siguen funcionando con datos viejos hasta que los eventos pendientes se publiquen (INT-04, "réplica atrasada").
- **No hay indicador de retraso de réplica:** §8.2 mide solo la frescura de las vistas, no la de las réplicas **[PROPUESTA: indicador de retraso de réplica, a decidir en HU-27]**.
- **Alta o reconstrucción de una réplica no definida:** el contrato describe snapshots solo para TRX-04 (§8.3). Un módulo nuevo o con réplica corrupta necesita un mecanismo de carga inicial **[PROPUESTA: solicitar snapshot al dueño, como hace TRX-04]**.
- **Riesgo de uso indebido:** un módulo podría usar la réplica para decidir invariantes. Se mitiga con la regla 4 de la decisión y con la checklist de revisión cruzada (`BACKLOG.md` §6.3).

## Historias y diagramas relacionados

- Historias: [HU-29](../BACKLOG.md) (este ADR), [HU-12](../BACKLOG.md), [HU-14](../BACKLOG.md).
- Diagramas: [`C4N2-contenedores` (fuente)](../../diagramas/c4/src/C4N2-contenedores.puml) · [`C4N2-contenedores` (SVG)](../../diagramas/c4/export/C4N2-contenedores.svg): INT-04 y esquema propio por módulo.
- Documento: [Disponibilidad y escalabilidad](../arquitectura/disponibilidad-escalabilidad.md) (FAL-01 y FAL-02).
- ADR relacionados: [ADR-001](./ADR-001-monolito-modular-evolutivo.md), [ADR-002](./ADR-002-propiedad-datos-por-esquema.md), [ADR-004](./ADR-004-sincrono-y-eventos-con-outbox.md), [ADR-007](./ADR-007-flota-como-unidad-separada.md).
- Contrato: [ARQUITECTURA_BASE](../ARQUITECTURA_BASE.md) §2.2 (PRI-04), §3, §7.1, §7.3 (CE-02), §10 (`ReplicaCatalogo`); RNF-REN-02, RNF-DIS-01.
