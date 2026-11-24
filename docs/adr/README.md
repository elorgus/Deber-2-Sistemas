[README.md](https://github.com/user-attachments/files/35382008/README.md)
# Índice de ADR · Smart Inventory

Registro de decisiones de arquitectura. Cada ADR usa la plantilla completa: contexto,
al menos dos alternativas, decisión, consecuencias positivas y negativas, fecha y estado.

**Ciclo de vida** ([`BACKLOG.md` §7.2](../BACKLOG.md)): *Propuesto → Aceptado → Reemplazado por ADR-nnn* o *Deprecado*.
Un ADR aceptado no se edita: se crea uno nuevo que lo reemplaza. Los ADR no llevan versión.

| ADR | Título | Responsable | Historia | Estado | Fecha |
|---|---|---|---|---|---|
| [ADR-001](./ADR-001-monolito-modular-evolutivo.md) | Monolito modular evolutivo con unidades de despliegue | P1 | HU-20 | Propuesto | 2026-11-24 |
| [ADR-002](./ADR-002-propiedad-datos-por-esquema.md) | Propiedad de datos por módulo con aislamiento por esquema | P1 | HU-20 | Propuesto | 2026-11-24 |
| [ADR-003](./ADR-003-reserva-y-fefo-en-inventario.md) | La reserva de stock y la selección FEFO pertenecen a Inventario | P4 | HU-14 | Propuesto | — |
| [ADR-004](./ADR-004-sincrono-y-eventos-con-outbox.md) | Comandos síncronos y eventos de negocio con Outbox e idempotencia | P1 | HU-20 | Propuesto | 2026-11-24 |
| [ADR-005](./ADR-005-proyeccion-de-lectura-cqrs.md) | Proyección de lectura (CQRS) con consistencia eventual informada | P7 | HU-30 | Propuesto | — |
| [ADR-006](./ADR-006-deteccion-riesgo-en-reposicion.md) | La detección de riesgo de agotamiento pertenece a Reposición | P5 | HU-17 | Propuesto | — |
| [ADR-007](./ADR-007-flota-como-unidad-separada.md) | Flota y Operación como unidad de despliegue separada desde el MVP | P6 | HU-19 | Propuesto | — |
| [ADR-008](./ADR-008-versionamiento-de-artefactos.md) | Política de versionamiento de artefactos de diseño | P3 | HU-21 | Propuesto | — |
| [ADR-009](./ADR-009-replicas-locales-de-referencia.md) | Réplicas locales de datos de referencia en lugar de consultas síncronas | P1 | HU-29 | Propuesto | — |

> Los enlaces a ADR-003 y ADR-005 a ADR-009 se resuelven cuando sus PR se fusionen.
> Fuente de los títulos, responsables e historias: [`ARQUITECTURA_BASE` §12](../ARQUITECTURA_BASE.md).
