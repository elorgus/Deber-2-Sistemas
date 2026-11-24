# ADR-004: Comandos síncronos y eventos de negocio con Outbox e idempotencia

| Campo | Valor |
|---|---|
| Fecha | 2026-11-24 |
| Estado | Propuesto (pasa a *Aceptado* tras la revisión de P7 y el acta del equipo) |
| Responsable | P1 · Líder de arquitectura |
| Historia | HU-20 |
| Revisor | P7 |

## Contexto

Los módulos, el dashboard y los sistemas externos deben comunicarse sin acoplar sus
fallos. Hay interacciones donde el usuario espera una respuesta inmediata o se
protege un invariante (el disponible nunca es negativo, RNF-CON-01), y otras donde
solo se informa de un hecho ya ocurrido a varios interesados: proyección,
Lakehouse, réplicas, reposición y alertas.

Si todo fuera por llamada directa, la caída de un destino (broker, ERP,
Analítica, Proyección) se propagaría hacia atrás. Si todo fuera por eventos, el
usuario no tendría confirmación inmediata ni se protegerían los invariantes. Además,
publicar un evento y guardar el cambio de negocio por separado puede dejar a
ambos inconsistentes si falla uno de los dos pasos.

## Alternativas consideradas

### Alternativa A: Llamadas directas síncronas entre componentes para todo

- A favor: modelo simple; el resultado es inmediato en cada paso.
- En contra: el acoplamiento temporal propaga los fallos. Una caída de Reposición
  o del ERP podría impedir registrar una recepción. Los consumidores múltiples
  (TRX-04, SVC-06, SVC-03, réplicas) harían crecer la cadena de llamadas, y
  Analítica pasaría a depender de la operación en línea. Contradice RNF-DIS-01
  y los escenarios FAL-01, FAL-04 y FAL-05.

### Alternativa B: Eventos publicados directamente al broker, sin Outbox

- A favor: menos piezas dentro de cada módulo.
- En contra: el cambio de negocio y la publicación son dos escrituras
  independientes. Si falla una, se pierde el evento o se publica uno falso. Con el
  broker caído, el módulo no podría completar su operación (FAL-01).

### Alternativa C: Comandos síncronos solo donde se justifican + eventos de negocio con Outbox e idempotencia (elegida)

- Se usa modo S únicamente para los criterios S1 a S3 de §7.1 (usuario esperando,
  invariante que debe confirmarse, dato exacto del dueño). Todo hecho de negocio se
  propaga como evento por TRX-03, guardado en el outbox en la misma transacción
  que el cambio (TX-01 a TX-06).
- Los consumidores son idempotentes, porque el broker entrega al menos una vez.

## Decisión

Se adopta la **Alternativa C**.

1. **Síncrono (S)** para los comandos del usuario hacia el módulo dueño, las
   consultas de detalle exacto (INT-15) y la reserva (INT-19). Toda llamada S entre
   módulos lleva tiempo límite y Circuit Breaker en el adaptador del consumidor.
   Solo se reintentan operaciones idempotentes (la reserva usa el identificador del
   pedido como clave). Se permiten como máximo dos saltos síncronos por operación.
2. **Asíncrono (A)** para comunicar hechos de negocio (EVT-xx), siguiendo los
   criterios A1 a A4. Solo el dueño del dato emite el evento (§6.1).
3. **Outbox (PRI-06):** el cambio de negocio y su evento se guardan en la misma
   transacción local del dueño. La publicación a TRX-03 ocurre después.
4. **Idempotencia:** todo consumidor (`ConsumidorIdempotente`) registra los eventos
   ya procesados y tolera duplicados. El orden se garantiza por entidad. Tras
   reintentos acotados, un evento fallido pasa a la DLQ y genera una alerta (FAL-08).
5. Los eventos informan hechos; las decisiones se toman en el dueño del dato
   (PRI-05). Los resultados en vista de TRX-04 nunca se usan para decidir (PRI-07).

**Por qué eventos para la comunicación de hechos y alertas:** el destinatario puede
reaccionar segundos después sin dañar el negocio (A2). Un retraso se resuelve del
lado conservador (A4: liberar una reserva tarde nunca provoca sobreventa; CE-04 y
CE-05). Los destinatarios externos o inestables se desacoplan con reintentos (A3:
ERP, transportistas). El emisor sigue operando aunque el broker o un consumidor
estén caídos.

## Consecuencias positivas

- Con el broker caído, los módulos siguen operando y los eventos esperan en el outbox. Al volver, se publican en orden por entidad sin pérdida ni efectos duplicados (FAL-01, RNF-DIS-01).
- Cambio de negocio y evento nunca quedan inconsistentes, porque van en la misma transacción.
- Agregar un consumidor (TRX-04, SVC-06, una réplica nueva) no modifica al emisor.
- La invariante crítica (disponible ≥ 0) se protege con confirmación síncrona en el dueño (TX-02).
- Es coherente con la extracción futura: los mismos eventos y puertos sirven en monolito y en microservicios (PRI-09).

## Consecuencias negativas

- Consistencia eventual en vistas, réplicas, detección de riesgo y cierres de OC (CE-01 a CE-07): hay que mostrar la frescura al usuario (§8.2).
- Más complejidad: outbox, `ConsumidorIdempotente`, DLQ y alertas operativas.
- El flujo C es una coreografía con compensación (EVT-28 → EVT-10), más difícil de seguir que una llamada directa.
- Los duplicados y los fallos persistentes requieren monitoreo y reproceso (FAL-08).
- Las llamadas síncronas siguen necesitando tiempo límite y Circuit Breaker para no heredar los fallos del destino.

## Historias y diagramas relacionados

- Historias: HU-20 (este ADR), HU-12, HU-24, HU-30.
- Diagramas: `C4N2-contenedores` (relaciones S y A con su INT), secuencias de los flujos A, B y C.
- ADR relacionados: ADR-001, ADR-002, ADR-005, ADR-009.
- Contrato: ARQUITECTURA_BASE §6.1, §7.1, §7.3 (TX-01 a TX-06, CE-01 a CE-07), PRI-05 a PRI-07; FAL-01, FAL-08.