# ADR-006 · La detección de riesgo de agotamiento pertenece a Reposición

| Campo | Valor |
|---|---|
| Estado | Propuesto |
| Fecha | 2026-11-23 |
| Responsable · Revisor | P5 · P4 |
| HU relacionada | HU-17 |
| Elementos del contrato afectados | `SVC-02`, `SVC-03`, `TRX-04`, `EVT-05`, `EVT-14`, `EVT-15`, `EVT-16`, `INT-10`, `INT-11`, `VL-01`, `VL-02`, `CE-03`, `RNF-CON-02`, `RNF-MAN-01` |
| Diagramas relacionados | `ACT-FB-reposicion`, `SEC-FB-reposicion` |

## 1. Contexto

El enunciado (Flujo B, paso 1) deja abierto **quién detecta** que un producto queda bajo el mínimo: puede ser Inventario, al publicar el cambio de saldo, o la proyección de lectura.

La regla compara dos datos con dueños distintos (`ARQUITECTURA_BASE.md` §5):
- el **disponible**, que es de `SVC-02`;
- el **mínimo** (política de reposición), que es de `SVC-03`.

Principios del contrato que influyen:
- **PRI-01:** un dueño por dato.
- **PRI-05:** los hechos viajan por eventos y la decisión la toma el dueño.
- **PRI-07:** el dashboard no es fuente de verdad.
- El enunciado (§5, principio S) pide separar el cálculo de reposición del registro de movimientos.

Requisitos del negocio y de calidad:
- La alerta debe aparecer en `VL-02` y retirarse cuando el SKU vuelve sobre su mínimo (HN-09, `EVT-14`, `EVT-15`).
- Toda vista muestra la antigüedad de su dato (`RNF-CON-02`).
- Agregar una política de reposición no debe cambiar el flujo (`RNF-MAN-01`).

## 2. Opciones consideradas (mínimo 2)

### Opción A · Detección en SVC-02 Inventario
- **Descripción:** Inventario compara el nuevo disponible con el mínimo dentro de `TX-01`, `TX-02` o `TX-03` y emite el evento de riesgo.
- **Ventajas:** la detección es inmediata, en la misma transacción que el cambio de saldo y sin salto asíncrono.
- **Desventajas:**
  - `SVC-02` necesitaría los mínimos: o pasa a ser dueño de la política (dos dueños de reglas de reposición, contra PRI-01) o mantiene una réplica de la política de `SVC-03`.
  - Mezcla el registro de movimientos con reglas de reposición, lo que rompe SRP y contradice el principio S que pide el enunciado.
  - Cada cambio de política (`OPS-04`) tocaría al primer candidato a extracción (§2.4).

### Opción B · Detección en TRX-04 Proyección de lectura
- **Descripción:** la proyección ve que el disponible quedó bajo el mínimo y genera la alerta.
- **Ventajas:** ya recibe `EVT-05` y alimenta `VL-02`, así que la alerta llega directo al dashboard.
- **Desventajas:**
  - Contradice PRI-05, PRI-07 y la definición de `TRX-04` (§4.1: "no genera alertas ni decisiones").
  - La proyección es reconstruible (`FAL-06`): al reprocesar eventos repetiría o perdería alertas, así que no puede ser dueña del estado de una alerta.
  - Necesitaría los mínimos, que son de `SVC-03`.
  - `SVC-03` igual tendría que enterarse para generar la sugerencia, y la decisión quedaría repartida entre dos componentes.

### Opción C · Detección en SVC-03 Reposición y Compras
- **Descripción:** `SVC-03` recibe `EVT-05` (`INT-10`) y mantiene una réplica del disponible agregado por SKU. Su componente `EvaluadorRiesgoAgotamiento` compara ese disponible con la política propia, emite `EVT-14` y `EVT-15` (`INT-11`) y, si la estrategia lo indica, genera la sugerencia `EVT-16`.
- **Ventajas:**
  - Un solo dueño para mínimo, alerta, sugerencia y OC (PRI-01).
  - `SVC-02` no sabe nada de reposición: SRP y menor acoplamiento.
  - Cambiar la política o la estrategia no toca `SVC-02` (`EstrategiaReposicion`, `RNF-MAN-01`).
  - La proyección solo consolida la alerta, como pide PRI-05.
- **Desventajas:**
  - La detección es eventual: `CE-03` (≤ 5 s desde `EVT-05`).
  - Con el broker caído no hay detección hasta que se publique el outbox (`FAL-01`, RIE-03).
  - `SVC-03` mantiene una réplica más que sincronizar (disponible agregado).
  - La alerta llega a `VL-02` tras dos saltos asíncronos (`CE-03` + `CE-01`); ver H-01 de HU-08.

## 3. Decisión

Se elige la **Opción C**. La detección del riesgo de agotamiento, su resolución y la generación de la sugerencia pertenecen a `SVC-03`, porque es el dueño del mínimo y de la alerta (PRI-01, PRI-05). `SVC-02` solo publica el hecho (`EVT-05`) y `TRX-04` solo muestra el resultado.

## 4. Consecuencias

**Positivas**
- La responsabilidad queda clara: quien es dueño del mínimo es quien decide.
- `EvaluadorRiesgoAgotamiento` queda separado de `RegistroMovimientos` (`SVC-02`) y de `EstrategiaReposicion`, lo que sirve de evidencia de SRP y OCP (HU-31).
- Se puede agregar una estrategia nueva, como `ReposicionPorPronostico`, sin tocar Inventario ni el flujo (`RNF-MAN-01`).
- `SVC-02` y `SVC-03` se pueden extraer por separado sin rediseñar la detección (§2.4, PRI-09).

**Negativas o riesgos**
- Detección eventual. Se mitiga con Outbox y `ConsumidorIdempotente` (PRI-06, ADR-004), orden por entidad (§6.1) y frescura visible en `VL-02` (§8.2, `RNF-CON-02`).
- Sin detección mientras el broker está caído. Se mitiga porque los eventos esperan en el outbox y, al volver el broker, se reevalúa en orden (`RNF-DIS-01`).
- Latencia encadenada hasta `VL-02`. Se mitiga fijando en HU-27 un presupuesto de extremo a extremo para la alerta (H-01 de HU-08).
- Una réplica adicional en `SVC-03`. Se alimenta solo con `EVT-05` y se puede reconstruir reprocesando eventos (ADR-009).

## 5. Notas

- **Cómo se verifica:**
  - En el C4 N3 de `SVC-03` (HU-15), `EvaluadorRiesgoAgotamiento` aparece separado de `EstrategiaReposicion` y de la gestión de sugerencias.
  - Ningún diagrama muestra a `SVC-02` o `TRX-04` emitiendo `EVT-14` o `EVT-15`.
  - Ningún diagrama muestra a `SVC-03` leyendo el esquema de `SVC-02` (`RNF-MAN-02`).
- **Relacionados:** HN-09, HN-10 · PRI-01, PRI-05, PRI-07 · ADR-004, ADR-005, ADR-009 · RIE-03, RIE-05 · HU-15, HU-27, HU-31.
- Un ADR *Aceptado* no se edita: si la decisión cambia, se crea un ADR nuevo que lo reemplaza (`docs/VERSIONAMIENTO.md` §6).
- Fuera de alcance: endpoints, payloads, CI/CD y pruebas.
