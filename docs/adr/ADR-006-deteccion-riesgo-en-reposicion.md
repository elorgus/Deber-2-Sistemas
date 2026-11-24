# ADR-006 · La detección de riesgo de agotamiento pertenece a Reposición

| Campo | Valor |
|---|---|
| ADR | ADR-006 |
| Estado | **Propuesto** (pasa a *Aceptado* cuando se apruebe el PR de HU-17) |
| Fecha | 2026-11-23 |
| Responsable | P5 |
| Revisor | P4 |
| Historia | HU-17 (diseña HN-09 y HN-10) |
| Contrato de referencia | `docs/ARQUITECTURA_BASE.md` v1.0.0 (§2.2, §3.3, §4.1, §5, §12) |
| Diagramas relacionados | `ACT-FB-reposicion` v1.0.0 · `SEC-FB-reposicion` v1.0.0 |

> Los ADR no llevan versión (BACKLOG §7.2). Un ADR aceptado no se edita: se reemplaza con uno nuevo.

---

## 1. Contexto

- El enunciado (Flujo B, paso 1) deja abierto **quién detecta** que un producto queda bajo el mínimo: puede ser Inventario, al publicar el cambio de saldo, o la proyección de lectura.
- La regla compara dos datos con dueños distintos: el **disponible**, que es de SVC-02, y el **mínimo** (política de reposición), que es de SVC-03 (ARQUITECTURA_BASE §5).
- Principios aplicables:
  - **PRI-01:** un dueño por dato.
  - **PRI-05:** los hechos viajan por eventos y la decisión la toma el dueño.
  - **PRI-07:** el dashboard no es fuente de verdad.
- El enunciado (§5, principio S) pide separar el cálculo de reposición del registro de movimientos.
- Necesidades del negocio:
  - La alerta debe aparecer en VL-02 (HN-09).
  - Debe resolverse cuando el SKU vuelve sobre su mínimo (EVT-15).
  - Debe existir una sola alerta activa por SKU.

---

## 2. Opciones consideradas

### Opción 1 · Detección en SVC-02 Inventario

Inventario compara el nuevo disponible con el mínimo dentro de TX-01, TX-02 o TX-03 y emite el evento de riesgo.

| A favor | En contra |
|---|---|
| La detección es inmediata, en la misma transacción que el cambio de saldo y sin salto asíncrono | SVC-02 necesitaría los mínimos: o pasa a ser dueño de la política (dos dueños de reglas de reposición, contra PRI-01) o mantiene una réplica de la política de SVC-03 |
| | Mezcla el registro de movimientos con reglas de reposición (rompe SRP y contradice el principio S que pide el enunciado) |
| | Cada cambio de política (OPS-04) tocaría al primer candidato a extracción (§2.4) |

### Opción 2 · Detección en TRX-04 Proyección de lectura

La proyección ve que el disponible quedó bajo el mínimo y genera la alerta.

| A favor | En contra |
|---|---|
| Ya recibe EVT-05 y alimenta VL-02, así que la alerta llega directo al dashboard | Contradice PRI-05, PRI-07 y la definición de TRX-04 (§4.1: "no genera alertas ni decisiones") |
| | La proyección es reconstruible (FAL-06): al reprocesar eventos repetiría o perdería alertas, así que no puede ser dueña del estado de una alerta |
| | Necesitaría los mínimos, que son de SVC-03 |
| | SVC-03 igual debe enterarse para generar la sugerencia, y la decisión quedaría repartida entre dos componentes |

### Opción 3 · Detección en SVC-03 Reposición y Compras ✅

SVC-03 recibe EVT-05 (INT-10) y mantiene una réplica del disponible agregado por SKU. Su componente `EvaluadorRiesgoAgotamiento` compara ese disponible con la política propia, emite EVT-14 y EVT-15 (INT-11) y, si la estrategia lo indica, genera la sugerencia EVT-16.

| A favor | En contra |
|---|---|
| Un solo dueño para mínimo, alerta, sugerencia y OC (PRI-01) | La detección es eventual: CE-03 (≤ 5 s desde EVT-05) |
| SVC-02 no sabe nada de reposición: SRP y menor acoplamiento | Con el broker caído no hay detección hasta que se publique el outbox (FAL-01, RIE-03) |
| Cambiar la política o la estrategia no toca SVC-02 (`EstrategiaReposicion`, RNF-MAN-01) | SVC-03 mantiene una réplica más que sincronizar (disponible agregado) |
| La proyección solo consolida la alerta, como pide PRI-05 | La alerta llega a VL-02 tras dos saltos asíncronos (CE-03 + CE-01); ver H-01 de HU-08 |

---

## 3. Decisión

Se elige la **Opción 3**: la detección del riesgo de agotamiento, su resolución y la generación de la sugerencia pertenecen a **SVC-03 Reposición y Compras**. SVC-02 solo publica el hecho (EVT-05) y TRX-04 solo muestra el resultado.

---

## 4. Consecuencias

**Positivas**
- La responsabilidad queda clara: quien es dueño del mínimo es quien decide (PRI-01, PRI-05).
- `EvaluadorRiesgoAgotamiento` queda separado de `RegistroMovimientos` (SVC-02) y de `EstrategiaReposicion`, lo que sirve de evidencia de SRP y OCP (HU-31).
- Se puede agregar una estrategia nueva, como `ReposicionPorPronostico`, sin tocar Inventario ni el flujo (RNF-MAN-01).
- SVC-02 y SVC-03 se pueden extraer por separado sin rediseñar la detección (§2.4, PRI-09).

**Negativas y cómo se mitigan**

| Consecuencia | Mitigación |
|---|---|
| Detección eventual | Outbox y `ConsumidorIdempotente` (PRI-06, ADR-004), orden por entidad (§6.1) y frescura visible en VL-02 (§8.2, RNF-CON-02) |
| Sin detección mientras el broker está caído | Los eventos esperan en el outbox y, al volver el broker, se reevalúa en orden (RNF-DIS-01) |
| Latencia encadenada hasta VL-02 | Fijar en HU-27 un presupuesto de extremo a extremo para la alerta (H-01 de HU-08) |
| Réplica adicional en SVC-03 | Se alimenta solo con EVT-05 y se puede reconstruir reprocesando eventos (ADR-009) |

---

## 5. Cómo se verifica

- En el C4 N3 de SVC-03 (HU-15), `EvaluadorRiesgoAgotamiento` aparece separado de `EstrategiaReposicion` y de la gestión de sugerencias.
- Ningún diagrama muestra a SVC-02 o TRX-04 emitiendo EVT-14 o EVT-15.
- Ningún diagrama muestra a SVC-03 leyendo el esquema de SVC-02 (RNF-MAN-02).

---

## 6. Relacionados

HN-09, HN-10 · EVT-05, EVT-14, EVT-15, EVT-16 · INT-10, INT-11 · CE-03, CE-01 · PRI-01, PRI-05, PRI-07 · ADR-004, ADR-005, ADR-009 · RIE-03, RIE-05 · HU-15, HU-17, HU-27, HU-31

---

## Historial

| Fecha | Estado | Nota |
|---|---|---|
| 2026-11-23 | Propuesto | Redacción inicial (HU-17, P5) |
