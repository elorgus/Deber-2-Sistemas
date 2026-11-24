# HU-31 · Evidencia SOLID inicial · Sprint 3

**Versión** 0.1.0 · **Fecha** 2026-11-23 · **Autor** P7 (`luisdavidborjap-web`) · **Revisor** P1 (`elorgus`)

Pseudocódigo de diseño basado en los nombres canónicos de la [§10 del contrato v1.0.0](../../docs/ARQUITECTURA_BASE.md#10-nombres-canónicos-de-puertos-estrategias-y-adaptadores). Las flechas indican dependencias o realizaciones descritas en cada línea.

### S · Responsabilidad única (SRP)

```text
SVC-02 Inventario · RegistroMovimientos
    registra los movimientos que modifican el saldo

SVC-03 Reposición y Compras · EvaluadorRiesgoAgotamiento
    evalúa EVT-05 SaldoActualizado contra la política del SKU
```

**Cambio futuro que facilita.** Ajustar el criterio de riesgo de agotamiento solo requiere cambiar la evaluación en Reposición y Compras. El registro de movimientos conserva su responsabilidad en Inventario.

### O · Abierto/cerrado (OCP)

```text
SVC-03 → usa EstrategiaReposicion para calcular la sugerencia
    ReposicionMinMax         → realiza EstrategiaReposicion
    ReposicionPuntoReorden   → realiza EstrategiaReposicion
    ReposicionPorPronostico  → realiza EstrategiaReposicion
```

**Cambio futuro que facilita.** Incorporar otra política de cálculo mediante `EstrategiaReposicion` permite conservar el flujo de generación y aprobación de sugerencias.

### L · Sustitución de Liskov (LSP)

```text
SVC-05 Flota y Operación → depende de PuertoTransportista
    AdaptadorTransportistaPropio  → realiza PuertoTransportista
    AdaptadorTransportistaExterno → realiza PuertoTransportista

Ambos adaptadores conservan el significado de solicitud,
confirmación y falta de confirmación de un vehículo.
```

**Cambio futuro que facilita.** Sustituir el adaptador propio por el externo conserva las expectativas de Flota si ambos ofrecen el mismo comportamiento observable. Por ejemplo, un vehículo pendiente de confirmación mantiene ese significado en ambos casos.

### I · Segregación de interfaces (ISP)

```text
SVC-02 ofrece tres puertos de entrada:
    ConsultaExistencias  ← consulta de existencias y lotes
    GestionReservas      ← reserva o liberación
    RegistroMovimientos  ← recepción, transferencia, ajuste o salida
```

**Cambio futuro que facilita.** Ampliar la consulta de existencias deja a quienes registran movimientos o gestionan reservas dependiendo únicamente de sus respectivos puertos.

### D · Inversión de dependencias (DIP)

```text
SVC-04 Pedidos y Fulfillment → depende de PuertoReservasInventario
    AdaptadorReservasInventario → realiza PuertoReservasInventario
    AdaptadorReservasInventario → conecta con GestionReservas de SVC-02

INT-19 Solicitar reserva FEFO (S)
    MVP: llamada interna a UD-1 Núcleo transaccional
    Extracción de SVC-02: llamada remota mediante el adaptador
```

**Cambio futuro que facilita.** Extraer Inventario del núcleo transaccional requiere adaptar la comunicación. La coordinación de Pedidos y Fulfillment conserva su dependencia de `PuertoReservasInventario`.
