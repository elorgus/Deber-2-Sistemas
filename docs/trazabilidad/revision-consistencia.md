# Revisión integral de consistencia · Smart Inventory

| Campo | Valor |
|---|---|
| Documento | `docs/trazabilidad/revision-consistencia.md` |
| Versión | **1.0.0** (Cerrado) |
| Estado | Finalizado: hallazgos **cerrados** |
| Dueño | P1 · Líder de arquitectura |
| Revisión | P7 |
| Fecha | 2026-11-27 |
| Historia | HU-34 · Rúbrica: Trazabilidad y presentación |
| Contrato revisado | [`ARQUITECTURA_BASE.md`](../ARQUITECTURA_BASE.md) v1.1.0 |
| Checklist aplicada | [`BACKLOG.md` §6.3](../BACKLOG.md) |

---

## 1. Alcance y método

### 1.1 Método
Se aplicó la checklist de la sección 6.3 del BACKLOG a todos los artefactos del repositorio. Se cruzaron los identificadores, nombres canónicos, modos (S/A) y productores/consumidores de eventos directamente contra el contrato y el código PlantUML.

### 1.2 Artefactos revisados
Se auditaron todos los artefactos generados en los Sprints 1 a 4 (C4 N1 a N3, ADRs, flujos UML, matriz de interacciones, requisitos de calidad, patrones, SOLID y dashboard).

---

## 2. Resultado de la checklist §6.3

| # | Ítem de la checklist §6.3 | Estado | Evidencia |
|---|---|---|---|
| 1 | Nombres idénticos al contrato | **Aprobado** | 0 identificadores inexistentes; 9 de 9 nombres de §10 coinciden. |
| 2 | Sin escritura cruzada (PRI-02) | **Aprobado** | Cada esquema recibe solo la flecha de su dueño. |
| 3 | Interacciones S/A con su INT | **Aprobado** | Se completaron los INT faltantes en el contrato (v1.1.0). |
| 4 | Eventos en pasado y emitidos por su dueño | **Aprobado** | 0 productores incorrectos. |
| 5 | Dashboard no decide ni es fuente de verdad | **Aprobado** | Las acciones se validan en el dueño. |
| 6 | Fuente y exportación sincronizadas | **Aprobado** | Se estandarizó la ruta a `docs/diagramas/`. |
| 7 | Sin contenido fuera de alcance | **Aprobado** | No hay APIs, JSON, ni CI/CD. |
| 8 | Vinculado a HU y CHANGELOG actualizado | **Aprobado** | ADRs pasaron a *Aceptado* y CHANGELOG actualizado. |
| 9 | Sin secretos ni datos personales | **Aprobado** | 0 coincidencias. |

---

## 3. Registro de hallazgos y resoluciones (CA2)

Todos los hallazgos detectados durante la auditoría fueron resueltos en la versión 1.1.0 del contrato y en los artefactos correspondientes.

| ID | Severidad | Qué se encontró | Resolución aplicada | Estado |
|---|---|---|---|---|
| **H-01** | Alta | Tres rutas distintas para los diagramas (`BACKLOG` vs `CONTRIBUTING`). | Se decidió estandarizar en `docs/diagramas/`. Se actualizó el `BACKLOG.md` §8.1 y los enlaces en todos los ADRs. | **Cerrado** |
| **H-02** | Media | SVC-06 consumía eventos no listados en §6.2. | Se actualizó §6.2 del contrato para incluir a SVC-06 como consumidor de EVT-24, EVT-29 y EVT-34. | **Cerrado** |
| **H-03** | Media | 10 eventos no tenían INT y faltaba INT para Usuarios → TRX-01. | Se agregaron los INT faltantes al contrato (v1.1.0). Se actualizó el C4 N2. | **Cerrado** |
| **H-04** | Media | Faltaba escenario de fallo para UD-1. | Se agregó el escenario FAL-10 (Caída de UD-1) en `disponibilidad-escalabilidad.md`. | **Cerrado** |
| **H-05** | Baja | Umbral de RNF-ESC-01 contradecía CE-07. | Se alineó el umbral de escalado de SVC-06 a 15 min en el contrato (v1.1.0) para coincidir con CE-07. | **Cerrado** |
| **H-06** | Baja | Parámetros marcados [PROPUESTA]. | Se definieron los valores en el acta de cierre y se removió la marca [PROPUESTA] de los documentos. | **Cerrado** |
| **H-07** | Baja | Convenciones de ramas/commits divergían entre `CONTRIBUTING` y `BACKLOG`. | Se actualizó el `CONTRIBUTING.md` para alinearse con el `BACKLOG.md` §7.1. | **Cerrado** |
| **H-08** | Media | ADRs en estado *Propuesto*. | Se cambió el estado de ADR-001, 002, 004 y 009 a *Aceptado* y se actualizó el índice. | **Cerrado** |

---

## 4. Acciones correctivas aplicadas (CA3)

Se realizaron los siguientes cambios para asegurar la consistencia del paquete de diseño:

1. **Contrato actualizado a v1.1.0:** Se agregaron los consumidores faltantes (H-02), los INT (H-03) y el ajuste del umbral de escalabilidad (H-05). El historial del contrato refleja estos cambios.
2. **Estandarización de rutas:** Todos los diagramas ahora viven en `docs/diagramas/` (H-01).
3. **ADRs Aceptados:** Los 9 ADRs se encuentran en estado *Aceptado* en el índice.
4. **CHANGELOG actualizado:** Se agregó la entrada correspondiente a la versión `diseno-v1.0.0` con los artefactos finales.

## 5. Conclusión
El paquete de diseño de Smart Inventory es consistente, trazable y está alineado con el contrato maestro. La auditoría no encontró artefactos huérfanos ni contradicciones de límites. La arquitectura está lista para ser entregada.

---

## Historial de cambios

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 0.1.0 | 2026-11-27 | P1 | Registro inicial: 8 hallazgos abiertos. | HU-34 |
| 1.0.0 | 2026-11-27 | P1 | Auditoría cerrada. Hallazgos resueltos en contrato v1.1.0. | HU-34 |
