# ADR-007 · Flota y Operación (`SVC-05`) como unidad separada

| Campo | Valor |
|---|---|
| Documento | `docs/adr/ADR-007-flota-como-unidad-separada.md` |
| Estado | Aceptado en diseño |
| Fecha | 2026-11-24 |
| Autor | P6 · Diseñador de operación (Pedidos y Flota) |
| Revisor | P5 |
| Historia | HU-19 · C4 Nivel 3: Flota y Operación y ADR-007 |

## Contexto
La operación logística reúne dos dinámicas distintas: la gestión de pedidos y preparación (`SVC-04` Pedidos y Fulfillment) y la ejecución física con flotas AMR, rutas, telemetría, visitas de vehículos, garita (`EXT-05`), transportistas (`EXT-02`) y muelles (`SVC-05` Flota y Operación)[cite: 21].

## Decisión
Mantener **Flota y Operación (`SVC-05`)** como una unidad separada con propiedad exclusiva sobre `AMR`, `Misión AMR`, `Ruta`, `Telemetría`, `Visita de vehículo`, `Muelle` y `Patio`, encapsulando las integraciones externas detrás de puertos y adaptadores (`EstrategiaAsignacionAmr`, `PuertoControladorAmr`, `PuertoTransportista`, `AsignadorMuelles`) y coordinándose con `SVC-04` y `SVC-02` mediante eventos y contratos definidos[cite: 21].

## Consecuencias
- **Positivas:** Aísla fallos de hardware/telemetría AMR o integraciones externas (`FAL-03`, `FAL-09`) sin detener el registro de pedidos ni la contabilidad de inventario; permite variar estrategias de asignación de AMR y muelles sin acoplar `SVC-04`.
- **Compensaciones:** Requiere coordinación asíncrona por eventos, proyección de lectura (`TRX-04`) para vistas operativas y manejo explícito de compensaciones o cancelaciones.
