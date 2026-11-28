[ADR-002-propiedad-datos-por-esquema (1).md](https://github.com/user-attachments/files/35537353/ADR-002-propiedad-datos-por-esquema.1.md)
# ADR-002: Propiedad de datos por módulo con aislamiento por esquema

| Campo | Valor |
|---|---|
| Fecha | 2026-11-24 |
| Estado | Propuesto (pasa a *Aceptado* tras la revisión de P7 y el acta del equipo) |
| Responsable | P1 · Líder de arquitectura |
| Historia | [HU-20](../BACKLOG.md) |
| Revisor | P7 |

## Contexto
ular-ev
Con el monolito modular ([ADR-001](./ADR-001-monolito-modolutivo.md)), cuatro
módulos (SVC-01 a SVC-04) comparten un despliegue (UD-1). Eso hace tentador que un
módulo lea directamente las tablas de otro. Si lo hace, el acoplamiento por datos
vuelve inútil cualquier extracción futura y deja de haber una fuente única de
verdad (PRI-01).

El contrato exige un dueño único por dato (§5), cero escrituras cruzadas (PRI-02),
referencias por identificador sin claves foráneas entre módulos (PRI-03) y réplicas
locales de solo lectura alimentadas por eventos (PRI-04). También exige verificar
que ningún diagrama muestre a un módulo accediendo al esquema de otro (RNF-MAN-02).

## Alternativas consideradas

### Alternativa A: Base de datos compartida

Todos los módulos usan las mismas tablas, con joins y claves foráneas entre
entidades de distintos dueños.

- A favor: consultas cruzadas triviales; menos duplicación de datos; sin
  consistencia eventual entre módulos.
- En contra: no hay dueño único de ningún dato, y cualquier cambio de estructura
  puede romper a otro módulo. Extraer SVC-02 u otro módulo exigiría desenredar
  los datos primero. Contradice PRI-01, PRI-02 y PRI-03.

### Alternativa B: Esquema separado por módulo con credencial exclusiva (elegida)

Cada SVC tiene su propio esquema lógico, con su propia credencial con permisos solo
sobre ese esquema. En el MVP pueden compartir el mismo motor físico. El outbox
vive en el esquema de su módulo, para guardarlo en la misma transacción que el
cambio de negocio.

- A favor: la propiedad se hace cumplir por permisos y no solo por convención. El
  costo operativo del MVP se mantiene bajo, y mover un esquema a su propio motor
  después es un cambio de despliegue.
- En contra: hay que replicar datos de referencia por eventos y aceptar
  consistencia eventual en esas réplicas.

### Alternativa C: Motor de base de datos separado por módulo desde el inicio

- A favor: aislamiento físico máximo; escalado independiente de cada base.
- En contra: costo operativo desproporcionado para el MVP (SUP-01 a SUP-03) y
  más puntos de falla, sin beneficio inmediato respecto de la Alternativa B,
  que ya aísla por esquema y credencial.

## Decisión

Se adopta la **Alternativa B**: **propiedad lógica de los datos por módulo con
aislamiento por esquema**.

1. Cada SVC es dueño de su esquema (§5) y es el único que lo lee y escribe, con
   credencial propia (PRI-01, PRI-02).
2. Entre módulos solo se comparte el identificador (SKU, pedido, OC), sin claves
   foráneas ni joins entre esquemas (PRI-03).
3. Cuando un módulo necesita datos de otro, usa una operación síncrona del dueño
   (p. ej., INT-19, INT-15) o una réplica local de solo lectura alimentada por
   eventos (PRI-04; INT-04, INT-16). No consulta nunca el esquema ajeno.
4. Los datos derivados (TRX-04 y TRX-05) nunca son fuente de verdad (PRI-07).

## Consecuencias positivas

- Cada dato tiene exactamente un dueño; se puede auditar contra la matriz de §5.
- La extracción futura de un módulo (§2.4) no exige separar datos: el esquema ya es suyo.
- El aislamiento se verifica en los diagramas (RNF-MAN-02) y en la revisión cruzada.
- Un cambio interno de estructura en un esquema no rompe a otros módulos.

## Consecuencias negativas

- Las réplicas locales quedan atrasadas unos segundos (CE-02): un SKU recién creado tarda en poder recibirse.
- No hay consultas cruzadas directas: los datos compuestos se arman con eventos, réplicas o la composición de TRX-02.
- Los datos de referencia se duplican y su sincronización por eventos debe ser idempotente ([ADR-004](./ADR-004-sincrono-y-eventos-con-outbox.md)).
- Exige administrar una credencial por esquema y mantenerlas fuera del repositorio (RNF-SEG-02).

## Historias y diagramas relacionados

- Historias: [HU-20](../BACKLOG.md) (este ADR, §4.2), [HU-12](../BACKLOG.md), [HU-13](../BACKLOG.md), [HU-14](../BACKLOG.md).
- Diagramas: [`C4N2-contenedores` (fuente)](../diagramas/fuente/C4N2-contenedores.puml) · [`C4N2-contenedores` (SVG)](../diagramas/export/C4N2-contenedores.svg): un esquema lógico por SVC y nota PRI-02.
- ADR relacionados: [ADR-001](./ADR-001-monolito-modular-evolutivo.md), [ADR-004](./ADR-004-sincrono-y-eventos-con-outbox.md), [ADR-009](./ADR-009-replicas-locales-de-referencia.md).
- Contrato: [ARQUITECTURA_BASE](../ARQUITECTURA_BASE.md) §2.2 (PRI-01 a PRI-04), §2.3 (Aislamiento de datos), §5; RNF-MAN-02.
