# ADR-005 · Proyección de lectura (CQRS) con consistencia eventual informada

| Campo | Valor |
|---|---|
| Fecha | 2026-11-23 |
| Estado | Propuesto |
| Responsable | P7 · `luisdavidborjap-web` |
| Revisor | P1 · `elorgus` |
| Historia de trabajo | [HU-30](https://github.com/elorgus/Deber-2-Sistemas/issues/74) |
| Base de diseño | [Contrato de arquitectura v1.0.0](../ARQUITECTURA_BASE.md) |

## Contexto

El dashboard reúne existencias, pedidos, alertas, operación de flota e indicadores. Las vistas deben informar al usuario cuándo se actualizaron, como pide HN-20. Operaciones como aprobar una sugerencia dependen del estado vigente del módulo dueño, como pide HN-11. Consultas consolidadas y decisiones de negocio tienen requisitos de frescura diferentes.

## Opciones consideradas

| Opción | Qué resuelve | Costo o límite |
|---|---|---|
| Consultar a los módulos dueños en cada actualización del dashboard | Obtiene el dato de su dueño al momento de cada consulta. | Multiplica llamadas para una pantalla consolidada. Aumenta la carga operativa y expone la pantalla a retrasos de varias capacidades. |
| Mantener una proyección de lectura alimentada por eventos | Prepara las vistas consolidadas de antemano y permite escalar su consulta de forma independiente. | La actualización es eventual. La frescura debe indicarse y se requiere un mecanismo de reconstrucción ante fallos. |

## Decisión propuesta

Adoptar **TRX-04 Proyección de lectura** como modelo de consulta del dashboard. `ActualizadorVista` prepara las vistas VL-01 a VL-07 a partir de hechos de negocio. **TRX-02 Capa de Aplicación** carga el snapshot y se suscribe a actualizaciones mediante **INT-03 (S + A)**. Los módulos dueños conservan la escritura y validan las acciones contra sus datos vigentes conforme a PRI-05 y PRI-07. Por ejemplo, **INT-12 (S)** dirige la aprobación de una sugerencia a **SVC-03 Reposición y Compras**.

Cada vista muestra la leyenda «Actualizado hace X s», calculada desde el último evento aplicado. Los objetivos dependen de la vista, según el §8.1 del contrato. Cuando la proyección está caída o corrupta, el dashboard avisa. El detalle de existencias puede consultarse a Inventario mediante **INT-15 (S)** y la proyección se reconstruye reprocesando los eventos, conforme a FAL-06.

| Estado de frescura | Condición del contrato |
|---|---|
| En vivo | Retraso dentro del objetivo de la vista. |
| Con retraso | Retraso mayor que el objetivo de la vista y hasta 60 s. |
| Desactualizada | Retraso mayor que 60 s o conexión perdida. |

## Consecuencias

**Beneficios.** Las vistas operativas se consultan sin componer todos sus datos en cada solicitud. La antigüedad queda visible. Cada módulo conserva la responsabilidad de aprobar, reservar o modificar sus datos.

**Costos y límites.** Una vista puede retrasarse con respecto al dueño. TRX-04 requiere consumo idempotente, orden por entidad y capacidad de reconstrucción. La consulta exacta de existencias se mantiene en Inventario conforme a INT-15. El objetivo de actualización de las vistas se evalúa con los requisitos del contrato, entre ellos RNF-REN-03 y RNF-CON-02.

## Trazabilidad y revisión

| Referencia | Qué aporta |
|---|---|
| [HU-30 · Informe de patrones](../diseno/patrones.md) | Justificación de CQRS, sus beneficios y límites. |
| [HU-12 · C4N2-contenedores](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/C4N2-contenedores.svg) | Ubica TRX-04 en UD-5 y muestra INT-03 (S + A) para el snapshot inicial y las actualizaciones en vivo. |
| [HU-16 · SEC-FA-recepcion](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/SEC-FA-recepcion.svg) | Muestra INT-08 (A), que actualiza VL-01, VL-04 y VL-06 tras una recepción. |
| [HU-17 · SEC-FB-reposicion](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/diagramas/export/SEC-FB-reposicion.svg) | Muestra cómo INT-12 (S) lleva la acción a SVC-03, que valida la vigencia de la sugerencia en su propio estado. |
| [HU-20 · ADR-004](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/adr/ADR-004-sincrono-y-eventos-con-outbox.md) | Documenta la publicación asíncrona por Outbox, la idempotencia y la separación entre vistas y decisiones de negocio. |

P1 revisa esta propuesta en HU-30. [HU-27](https://github.com/elorgus/Deber-2-Sistemas/issues/71) y [HU-32](https://github.com/elorgus/Deber-2-Sistemas/issues/76) permiten contrastar posteriormente las metas de frescura y su representación en el dashboard.
