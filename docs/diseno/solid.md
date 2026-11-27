# HU-31 · Evidencia de principios SOLID

| Campo | Valor |
|---|---|
| Versión | 0.1.0 |
| Fecha | 2026-11-27 |
| Estado | Propuesto para revisión |
| Responsable | P7 · `luisdavidborjap-web` |
| Revisor | P1 · `elorgus` |
| Referencias | [HU-31](https://github.com/elorgus/Deber-2-Sistemas/issues/75) y [nombres canónicos de la sección 10 del contrato](../ARQUITECTURA_BASE.md#10-nombres-canónicos-de-puertos-estrategias-y-adaptadores) |

## Evidencia de arquitectura

- **Inventario, HU-14.** Se toma como referencia el [C4 de Inventario revisado en el PR #56](https://github.com/elorgus/Deber-2-Sistemas/blob/2563a615ac463cd135fc3d47af139311a6b65ce4/docs/diagramas/export/C4N3-SVC02-inventario.svg). La referencia apunta a la revisión consultada mientras se resuelve la integración del PR.
- **Reposición y Compras, HU-15.** El [C4 de SVC-03](../diagramas/export/C4N3-SVC03-reposicion.svg) identifica al evaluador, la estrategia y sus variantes.
- **Flota y Operación, HU-19.** El [C4 de SVC-05](../../diagramas/c4/export/C4N3-SVC05-flota.svg) muestra el puerto de transporte, sus dos adaptadores y la interacción con EXT-02.
- **Flujo C, HU-18.** La [secuencia de reserva y despacho](../../diagramas/secuencia/export/SEC-FC-reserva-despacho.svg) muestra INT-19 y la reacción cuando Inventario no responde.

## S · Responsabilidad única (SRP)

`RegistroMovimientos` pertenece a **SVC-02 Inventario** y registra movimientos que afectan las existencias. `EvaluadorRiesgoAgotamiento` pertenece a **SVC-03 Reposición y Compras** y evalúa el disponible frente a la política del SKU. Inventario publica `EVT-05 SaldoActualizado`. Reposición lo recibe vía `TRX-03` y evalúa el riesgo mediante `INT-10 (A)`. Cada módulo conserva la responsabilidad sobre sus propios datos.

```text
SVC-02 · RegistroMovimientos
    Registra el movimiento que modifica el saldo

SVC-03 · EvaluadorRiesgoAgotamiento
    Evalúa EVT-05 SaldoActualizado frente a la política del SKU
```

**Cambio futuro que facilita.** Ajustar cuándo se activa o resuelve la alerta cambia la evaluación en Reposición y deja intacto el registro de movimientos en Inventario.

**Evidencia de diseño.** [CLS-SOLID-reposicion](../../diagramas/clases/export/CLS-SOLID-reposicion.svg).

## O · Abierto/cerrado (OCP)

En **SVC-03**, el flujo de sugerencias usa `EstrategiaReposicion`. `ReposicionMinMax`, `ReposicionPuntoReorden` y `ReposicionPorPronostico` realizan el mismo contrato y encapsulan las variantes del cálculo. Cuando falta el pronóstico necesario, se aplica `ReposicionMinMax`. La propuesta de cantidad se mantiene separada de la aprobación de la sugerencia.

```text
SVC-03 usa EstrategiaReposicion para calcular la sugerencia
    ReposicionMinMax realiza EstrategiaReposicion
    ReposicionPuntoReorden realiza EstrategiaReposicion
    ReposicionPorPronostico realiza EstrategiaReposicion
```

**Cambio futuro que facilita.** Incorporar otra política de cálculo requiere añadir una estrategia compatible y seleccionar cuándo se usa. El flujo de generación y aprobación de sugerencias se conserva, conforme a `RNF-MAN-01`.

**Evidencia de diseño.** [CLS-SOLID-reposicion](../../diagramas/clases/export/CLS-SOLID-reposicion.svg).

## L · Sustitución de Liskov (LSP)

En **SVC-05 Flota y Operación**, `AdaptadorTransportistaPropio` y `AdaptadorTransportistaExterno` realizan `PuertoTransportista`. Ambos conservan el comportamiento observable que espera el flujo de despacho cuando solicita o confirma un vehículo mediante `INT-27 (A)` con **EXT-02 Sistemas de transportistas**.

```text
SVC-05 usa PuertoTransportista
    AdaptadorTransportistaPropio realiza PuertoTransportista
    AdaptadorTransportistaExterno realiza PuertoTransportista

Con cualquiera de los adaptadores
    Solo una confirmación efectiva confirma el vehículo
    Si el transportista no responde, el pedido sigue EnDespacho
    El flujo conserva el reintento y la asignación manual por ROL-04
```

**Cambio futuro que facilita.** Sustituir un adaptador por el otro conserva el flujo de despacho siempre que ambos respeten el significado de la confirmación y el tratamiento de la falta de respuesta definido en `FAL-09`.

**Evidencia de diseño.** [CLS-SOLID-flota](../../diagramas/clases/export/CLS-SOLID-flota.svg).

## I · Segregación de interfaces (ISP)

**SVC-02** expone puertos de entrada separados. La Capa de Aplicación consulta el detalle por `ConsultaExistencias` y registra recepciones por `RegistroMovimientos`. **SVC-04 Pedidos y Fulfillment** solicita la reserva por `GestionReservas`.

```text
ConsultaExistencias  → consultar existencias y lotes    · INT-15 (S)
RegistroMovimientos  → registrar la recepción         · INT-06 (S)
GestionReservas      → solicitar la reserva FEFO      · INT-19 (S)
```

**Cambio futuro que facilita.** Ampliar el detalle de la consulta por lote afecta el puerto de consulta y a quienes lo usan. Quienes solicitan reservas o registran recepciones mantienen sus propios contratos.

**Evidencia de diseño.** [CLS-SOLID-inventario](../../diagramas/clases/export/CLS-SOLID-inventario.svg).

## D · Inversión de dependencias (DIP)

**SVC-04** depende de su puerto de salida `PuertoReservasInventario`. `AdaptadorReservasInventario` realiza ese puerto y conecta con el puerto de entrada `GestionReservas` de **SVC-02** mediante `INT-19 (S)`. Inventario verifica el disponible y confirma la reserva en su esquema con `TX-02`.

```text
SVC-04 depende de PuertoReservasInventario
    AdaptadorReservasInventario realiza PuertoReservasInventario
    AdaptadorReservasInventario solicita la reserva a GestionReservas

MVP                     → llamada interna de UD-1
Tras extraer Inventario → llamada remota mediante el adaptador
```

**Cambio futuro que facilita.** Extraer Inventario de UD-1 requiere cambiar cómo el adaptador se comunica con `GestionReservas`. Pedidos conserva su dependencia del puerto y la regla de reserva permanece en Inventario, conforme a `PRI-02` y `PRI-09`.

**Evidencia de diseño.** [CLS-SOLID-inventario](../../diagramas/clases/export/CLS-SOLID-inventario.svg).

## Fuentes de los diagramas

| Diagrama | Fuente | Exportación |
|---|---|---|
| CLS-SOLID-inventario | [PlantUML](../../diagramas/clases/src/CLS-SOLID-inventario.puml) | [SVG](../../diagramas/clases/export/CLS-SOLID-inventario.svg) |
| CLS-SOLID-reposicion | [PlantUML](../../diagramas/clases/src/CLS-SOLID-reposicion.puml) | [SVG](../../diagramas/clases/export/CLS-SOLID-reposicion.svg) |
| CLS-SOLID-flota | [PlantUML](../../diagramas/clases/src/CLS-SOLID-flota.puml) | [SVG](../../diagramas/clases/export/CLS-SOLID-flota.svg) |
