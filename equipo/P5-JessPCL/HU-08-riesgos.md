# Smart Inventory · Registro inicial de riesgos

| Campo | Valor |
|---|---|
| Documento | `equipo/P5-JessPCL/HU-08-riesgos.md` (ubicación según `CONTRIBUTING.md` §2) |
| Issue | #9 · HU-08: Registro inicial de riesgos del proyecto |
| Versión | **1.0.0** |
| Estado | Propuesto para revisión (Sprint 1 · HU-08) |
| Dueño | P5 · Diseñador de procesos y calidad |
| Revisor | P4 |
| Fecha | 2026-11-23 |
| Contrato de referencia | [`docs/ARQUITECTURA_BASE.md`](../../docs/ARQUITECTURA_BASE.md) v1.0.0 (§11.2 RNF semilla, §11.3 FAL) |
| Backlog de referencia | [`docs/BACKLOG.md`](../../docs/BACKLOG.md) v1.0.0 (HU-08, §6, §7) |
| Criterio de la rúbrica | Gestión del proyecto y sprints |

> **Identificadores locales.** Este registro numera los riesgos como `RIE-nn` y los hallazgos como `H-nn`.
> «PROPUESTA DE CAMBIO»: registrar `RIE-nn` como identificador de gestión en el encabezado de `BACKLOG.md` y en la fila de gestión de ARQUITECTURA_BASE §0.2 (cambio MINOR), para poder citarlo desde las actas de sprint. Mientras no se apruebe, `RIE-nn` solo se usa dentro de este documento. `H-nn` es numeración interna y no se cita fuera.

---

## 1. Escala de evaluación

**Categorías.** *Técnico* = riesgo de arquitectura (CA2: vinculado a SVC, FAL o RNF). *Equipo* y *Alcance* = riesgos de proyecto.

| Valor | Probabilidad (en los 4 sprints) | Impacto |
|---|---|---|
| **1 · Baja** | Poco probable; requiere varias fallas a la vez | Retrabajo menor dentro del sprint; no afecta la rúbrica |
| **2 · Media** | Puede ocurrir en algún sprint | Retrasa una HU o un incremento; afecta parte de un criterio de la rúbrica |
| **3 · Alta** | Probable si no se actúa | Rompe el flujo vertical, contradice el contrato o compromete un criterio completo de la rúbrica |

**Exposición = P × I** → **Alta** (6–9) · **Media** (3–4) · **Baja** (1–2).
**Estados:** Abierto · En mitigación · Materializado · Cerrado. Un riesgo cerrado no se borra: queda con su fecha.

---

## 2. Matriz resumen

| ID | Cat. | Riesgo | P | I | Exposición | Vínculo (SVC · FAL · RNF) | Resp. | Estado | Última revisión |
|---|---|---|---|---|---|---|---|---|---|
| RIE-01 | Técnico | Erosión de límites dentro de UD-1 | 2 | 3 | **6 · Alta** | SVC-01 a SVC-04 · RNF-MAN-02 · PRI-02 | P1 | Abierto | 2026-11-23 · S1 |
| RIE-02 | Técnico | UD-1 como punto único de fallo; FAL-02 sin definir para el MVP | 2 | 2 | 4 · Media | SVC-02, SVC-04 · FAL-02 · RNF-DIS-02, RNF-DIS-04 | P1 | Abierto | 2026-11-23 · S1 |
| RIE-03 | Técnico | Eventos perdidos, duplicados o fuera de orden | 1 | 3 | 3 · Media | TRX-03 · FAL-01, FAL-08 · RNF-DIS-01 | P7 | Abierto | 2026-11-23 · S1 |
| RIE-04 | Técnico | Sobrerreserva por concurrencia o reintentos de INT-19 | 1 | 3 | 3 · Media | SVC-02, SVC-04 · FAL-02 · RNF-CON-01, RNF-REN-02 | P4 | Abierto | 2026-11-23 · S1 |
| RIE-05 | Técnico | Frescura del dashboard: latencia encadenada del flujo B y retraso no visible | 3 | 2 | **6 · Alta** | SVC-03, TRX-04 · FAL-06 · RNF-REN-03, RNF-CON-02 | P5 | Abierto | 2026-11-23 · S1 |
| RIE-06 | Técnico | Réplicas locales atrasadas (catálogo y OC) | 2 | 2 | 4 · Media | SVC-02, SVC-03 · FAL-01 · CE-02, CE-06 | P4 | Abierto | 2026-11-23 · S1 |
| RIE-07 | Técnico | Dependencias externas inestables (EXT-01 a EXT-05) | 2 | 2 | 4 · Media | SVC-03, SVC-05 · FAL-03, FAL-05, FAL-07, FAL-09 · RNF-DIS-03 | P6 | Abierto | 2026-11-23 · S1 |
| RIE-08 | Técnico | Autorización solo en la UI o separación de funciones no aplicable | 2 | 2 | 4 · Media | Todos los SVC · RNF-SEG-01, RNF-SEG-02 · OPS-03 | P2 | Abierto | 2026-11-23 · S1 |
| RIE-09 | Equipo | Nombres, límites o interacciones contradictorios entre integrantes e IAs | 3 | 3 | **9 · Alta** | Contrato §0.2, §0.3 · RNF-MAN-02 | P1 | Abierto | 2026-11-23 · S1 |
| RIE-10 | Equipo | Cuello de botella en revisiones y PR acumulados al cierre del sprint | 3 | 2 | **6 · Alta** | BACKLOG §1.2, §6.2 | P3 | Abierto | 2026-11-23 · S1 |
| RIE-11 | Equipo | Baja disponibilidad (fin de semestre, feriados) y carga desigual | 2 | 2 | 4 · Media | BACKLOG §5.3 | P3 | Abierto | 2026-11-23 · S1 |
| RIE-12 | Equipo | Contribuciones individuales sin evidencia | 1 | 2 | 2 · Baja | BACKLOG HU-39, HU-40 | P3 | Abierto | 2026-11-23 · S1 |
| RIE-13 | Equipo | Fuente, exportación y versión de diagramas desincronizadas | 2 | 2 | 4 · Media | ADR-008 · BACKLOG §7.2, §7.3 | P3 | Abierto | 2026-11-23 · S1 |
| RIE-14 | Alcance | Contenido de Desarrollo de Software Moderno dentro de los artefactos | 2 | 2 | 4 · Media | Contrato §14 · BACKLOG §6.3 | P3 | Abierto | 2026-11-23 · S1 |
| RIE-15 | Alcance | Sobrediseño frente a la capacidad del equipo | 2 | 3 | **6 · Alta** | UD-2, UD-3 · BACKLOG §2.2, §5.3 | P2 | Abierto | 2026-11-23 · S1 |
| RIE-16 | Alcance | Requisitos de calidad incompletos o no verificables | 3 | 2 | **6 · Alta** | FAL-04 a FAL-09 · RNF-ESC-01, RNF-DIS-04 | P5 | Abierto | 2026-11-23 · S1 |
| RIE-17 | Alcance | Decisiones de arquitectura percibidas como desvío del enunciado | 2 | 2 | 4 · Media | ADR-001, ADR-003, ADR-006 | P1 | Abierto | 2026-11-23 · S1 |
| RIE-18 | Alcance | Trazabilidad incompleta: HN sin artefacto o artefactos huérfanos | 2 | 2 | 4 · Media | Todos los SVC · BACKLOG §2.2, HU-35 | P4 | Abierto | 2026-11-23 · S1 |

**Totales:** 18 riesgos (8 técnicos, 5 de equipo, 5 de alcance) · 6 de exposición alta, 11 media, 1 baja.

### 2.1 Mapa de calor (probabilidad × impacto)

| Probabilidad ↓ / Impacto → | 1 · Bajo | 2 · Medio | 3 · Alto |
|---|---|---|---|
| **3 · Alta** | — | RIE-05, RIE-10, RIE-16 | **RIE-09** |
| **2 · Media** | — | RIE-02, RIE-06, RIE-07, RIE-08, RIE-11, RIE-13, RIE-14, RIE-17, RIE-18 | RIE-01, RIE-15 |
| **1 · Baja** | — | RIE-12 | RIE-03, RIE-04 |

---

## 3. Detalle por riesgo

Cada riesgo indica causa, consecuencia, indicador de alerta (cuándo actuar), mitigación preventiva, contingencia (si ocurre) y HU donde se trata.

### 3.1 Riesgos técnicos (arquitectura)

#### RIE-01 · Erosión de límites dentro de UD-1
- **Causa:** SVC-01 a SVC-04 comparten despliegue y motor físico en el MVP (UD-1, ADR-001); es tentador dibujar una lectura o un join directo entre esquemas.
- **Consecuencia:** se rompen PRI-01 y PRI-02; extraer SVC-02 (§2.4) obligaría a rediseñar flujos; pérdida en "Límites de servicio y arquitectura C4".
- **Indicador:** un C4 N2/N3 o una secuencia con una flecha de un módulo hacia el esquema de otro, o con una interacción que no existe en §7.2.
- **Mitigación:** credencial por esquema (§2.3); punto 2 de la checklist §6.3 en cada PR; RNF-MAN-02 como criterio de revisión en HU-12 y HU-34; los módulos de UD-1 se comunican por puertos y eventos (PRI-09).
- **Contingencia:** se corrige antes del merge; si el acceso es realmente necesario, se abre «PROPUESTA DE CAMBIO» con ADR (§0.3).
- **Se trata en:** HU-12, HU-13, HU-34.

#### RIE-02 · UD-1 como punto único de fallo; FAL-02 sin definir para el MVP
- **Causa:** FAL-02 y RNF-DIS-02 describen a SVC-02 caído mientras SVC-04 sigue operando, pero en el MVP ambos viven en UD-1 e INT-19 es una llamada interna (§10).
- **Consecuencia:** RNF-DIS-02 no es verificable tal como está redactado; una caída de UD-1 detiene a la vez la recepción (flujo A), la aprobación de sugerencias (flujo B) y las reservas (flujo C), y solo RNF-DIS-04 lo acota.
- **Indicador:** HU-29 describe FAL-02 sin decir cómo se manifiesta dentro del monolito.
- **Mitigación:** en HU-29 definir "SVC-02 no disponible" dentro de UD-1 (por ejemplo, tiempo límite del puerto `GestionReservas` excedido o esquema de SVC-02 no accesible) y aclarar que RNF-DIS-02 aplica completo tras la extracción de SVC-02 (§2.4). Ver H-03.
- **Contingencia:** reformular RNF-DIS-02 en HU-27 y registrar el cambio (HU-27 CA4).
- **Se trata en:** HU-27, HU-29.

#### RIE-03 · Eventos perdidos, duplicados o fuera de orden
- **Causa:** TRX-03 entrega *al menos una vez*; puede caer (FAL-01) o un evento puede fallar repetidamente (FAL-08).
- **Consecuencia:** vistas VL con datos erróneos; mientras el broker esté caído, SVC-03 no recibe EVT-05 y deja de detectar riesgo de agotamiento (INT-10) sin otro aviso que la frescura de VL-02; un EVT-27 duplicado podría registrar dos salidas (INT-30).
- **Indicador:** secuencias A, B o C sin outbox marcado, o consumidores dibujados sin `ConsumidorIdempotente`; ACT-FA sin el camino de broker caído.
- **Mitigación:** PRI-06 (`PublicadorOutbox`, `ConsumidorIdempotente`), orden por entidad (§6.1), DLQ con alerta operativa (FAL-08); HU-16 CA3 modela FAL-01; RNF-DIS-01 se verifica en el diseño.
- **Contingencia:** se publican los pendientes desde el outbox, se reprocesa la DLQ sin duplicar efectos y TRX-04 se reconstruye (FAL-06).
- **Se trata en:** HU-16, HU-17, HU-29, HU-30.

#### RIE-04 · Sobrerreserva por concurrencia o reintentos de INT-19
- **Causa:** en carga pico (SUP-03: unos 900 pedidos/hora) varias reservas compiten por el mismo SKU · ubicación · lote; si `AdaptadorReservasInventario` reintenta tras un tiempo límite, la misma reserva podría aplicarse dos veces.
- **Consecuencia:** disponible negativo (viola RNF-CON-01), despacho de stock inexistente o FEFO mal aplicado.
- **Indicador:** TX-02 dibujada sin la verificación del disponible dentro de la transacción; reintentos sin clave de idempotencia.
- **Mitigación:** TX-02 (verificación + reserva + saldo reservado + outbox juntos); identificador del pedido como clave de idempotencia (§3.4, §7.1); reserva todo o nada en el MVP; Circuit Breaker en el adaptador; RNF-REN-02 como señal de extracción (§2.4).
- **Contingencia:** el pedido queda *PendienteReserva* (FAL-02); las diferencias se corrigen con un ajuste auditado (OPS-01).
- **Se trata en:** HU-14, HU-18, HU-27.

#### RIE-05 · Frescura del dashboard: latencia encadenada del flujo B y retraso no visible
- **Causa:** HN-09 exige que la alerta aparezca en VL-02 en ≤ 5 s, pero el camino tiene dos saltos asíncronos con objetivo propio: EVT-05 → SVC-03 (CE-03, ≤ 5 s) y EVT-14 → TRX-04 (CE-01, ≤ 5 s). En el peor caso suman cerca de 10 s. Además, ningún RNF de §11.2 mide la alerta de extremo a extremo: RNF-REN-03 solo cubre recepción → VL-01.
- **Consecuencia:** el criterio de HN-09 no se puede cumplir con los objetivos actuales del contrato; el Comprador decide con una vista atrasada si la frescura no se ve.
- **Indicador:** HU-27 copia los umbrales sin sumarlos; un boceto del dashboard sin el indicador de §8.2.
- **Mitigación:** en HU-27 fijar un presupuesto de extremo a extremo para el flujo B y repartirlo entre CE-03 y CE-01 (H-01); RNF-CON-02 y los 3 estados de §8.2 visibles en VL-02 (HU-32 CA2); toda acción se valida en SVC-03, nunca contra la proyección (PRI-07).
- **Contingencia:** «PROPUESTA DE CAMBIO» a HN-09 (dueño P2) para alinear su criterio con el presupuesto real.
- **Se trata en:** HU-17, HU-22, HU-27, HU-32.

#### RIE-06 · Réplicas locales atrasadas (catálogo y OC)
- **Causa:** SVC-02 valida recepciones contra sus réplicas de catálogo (EVT-01 a EVT-03) y de OC (EVT-19) (INT-06, PRI-04, ADR-009); SVC-03 decide con el disponible agregado que recibe por EVT-05.
- **Consecuencia:** un SKU recién creado o una OC recién emitida no son recibibles durante unos segundos (CE-02); con el broker caído (FAL-01) el atraso se alarga y el operador no puede registrar la recepción contra la OC (HN-12).
- **Indicador:** el flujo A no tiene camino alternativo para "OC todavía no replicada".
- **Mitigación:** objetivo CE-02 ≤ 5 s; ACT-FA (HU-16) modela la validación opcional de OC y el aviso al operador; ADR-009 documenta este costo frente a la consulta síncrona.
- **Contingencia:** la UI conserva los datos de la recepción y el operador reintenta cuando la réplica se actualiza (mismo comportamiento que FAL-02).
- **Se trata en:** HU-13, HU-16, HU-29.

#### RIE-07 · Dependencias externas inestables (EXT-01 a EXT-05)
- **Causa:** los sistemas externos no son del equipo y su comportamiento real se desconoce en esta materia.
- **Consecuencia:** flujos B y C bloqueados: OC sin enviar (INT-14), vehículo sin confirmar (INT-27), AMR sin misión (INT-22), sin nuevos inicios de sesión (INT-02).
- **Indicador:** una secuencia que llama a un EXT sin adaptador o sin camino de degradación.
- **Mitigación:** integraciones solo por adaptadores (`PuertoErp`, `PuertoTransportista`, `PuertoControladorAmr`; PRI-08); degradación manual de FAL-03, FAL-05, FAL-07 y FAL-09 (INT-33, ROL-04); modo A con reintentos en INT-14 e INT-27.
- **Contingencia:** operación manual desde el dashboard por INT-33; alerta en VL-02 cuando una OC sigue pendiente de envío (FAL-05).
- **Se trata en:** HU-17, HU-18, HU-19, HU-29.

#### RIE-08 · Autorización solo en la UI o separación de funciones no aplicable
- **Causa:** se diseñan permisos como botones ocultos en TRX-01; ROL-01 a ROL-07 no declaran combinaciones incompatibles; el umbral de monto de OPS-03 aún no existe.
- **Consecuencia:** RNF-SEG-01 no es verificable; un usuario con ROL-02 y ROL-03 a la vez podría aprobar una OC y registrar su recepción, contra la separación de funciones de §11.4.
- **Indicador:** la matriz ROL × operación de HU-28 no tiene pares incompatibles ni umbral de OPS-03.
- **Mitigación:** autorización en la capacidad dueña (§11.4); HU-28 define el umbral de OPS-03 y los pares de roles incompatibles (H-05); auditoría de OPS-01 a OPS-08; RNF-SEG-02 revisado en la checklist de cada PR.
- **Contingencia:** si HU-28 no cierra el umbral en el Sprint 3, se declara como supuesto explícito de RNF-SEG-01 y se registra en el acta.
- **Se trata en:** HU-27, HU-28.

### 3.2 Riesgos de equipo

#### RIE-09 · Nombres, límites o interacciones contradictorios entre integrantes e IAs
- **Causa:** siete integrantes trabajan en paralelo, cada uno con su IA; las IAs tienden a inventar sinónimos o traducir nombres al inglés.
- **Consecuencia:** diagramas que no coinciden con el contrato, trazabilidad rota (HU-35) y pérdida de puntos en casi todos los criterios.
- **Indicador:** un ID o nombre que no aparece en ARQUITECTURA_BASE; un elemento nuevo sin la marca «PROPUESTA».
- **Mitigación:** protocolo de IA (BACKLOG §9) con el contrato adjunto; punto 1 de la checklist §6.3; cambios solo por §0.3; prompts registrados en `equipo/Px/registro-ia.md`.
- **Contingencia:** HU-34 aplica la checklist a todos los artefactos y se corrige antes de `diseno-v1.0.0`.
- **Se trata en:** todas las HU; control en HU-34.

#### RIE-10 · Cuello de botella en revisiones y PR acumulados al cierre del sprint
- **Causa:** P1 aprueba todo PR que toca el contrato (§1.2) y tiene la mayor carga (24 pts); el Sprint 2 concentra 45 pts y los PR tienden a abrirse el último día.
- **Consecuencia:** HU que no cumplen la DoD a tiempo, tags `diseno-v0.x.0` tardíos y revisiones superficiales.
- **Indicador:** más de 3 PR sin revisión 48 h antes de la review; tarjetas detenidas en *En revisión*.
- **Mitigación:** acordar en la planning un plazo de revisión de 48 h; PR pequeños, uno por HU; usar el refinamiento de mitad de sprint (§3.2) como fecha límite para abrir PR; acordar un revisor alterno de P1 para cambios de forma (se registra en el acta y se actualiza §1.2 por PR).
- **Contingencia:** P3 expone el bloqueo en la sincronización y redistribuye revisiones.
- **Se trata en:** HU-05, HU-10, HU-23.

#### RIE-11 · Baja disponibilidad y carga desigual
- **Causa:** los sprints coinciden con el cierre de semestre (exámenes y entregas de otras materias) y, según las fechas que fije HU-05, con los feriados de fin de año; P1 (24 pts) y P2 (22 pts) tienen la mayor carga.
- **Consecuencia:** se retrasan HU de las que dependen otras (HU-06, HU-12, HU-37).
- **Indicador:** capacidad declarada menor que los puntos asignados; un integrante sin movimiento en el tablero durante una semana. **Subir a Alta** si el calendario de HU-05 cruza los feriados.
- **Mitigación:** capacidad real declarada en HU-05 (CA2); marcar feriados al fijar fechas; parejas de revisión que conocen el trabajo del otro (§1.2); rotación de roles permitida por el enunciado.
- **Contingencia:** reasignar HU con acta y actualizar BACKLOG §1.1 por PR.
- **Se trata en:** HU-05 y cada planning.

#### RIE-12 · Contribuciones individuales sin evidencia
- **Causa:** el trabajo con IA y los PR compartidos diluyen la autoría; el enunciado exige identificar autores y contribuciones.
- **Consecuencia:** el informe y la presentación no demuestran quién hizo qué.
- **Indicador:** `equipo/Px/CONTRIBUCIONES.md` vacío al cierre de un sprint.
- **Mitigación:** actualizar `CONTRIBUCIONES.md` en cada review; commits con el formato `HU-NN:`; HU-39 CA1 y HU-40 CA2.
- **Contingencia:** P3 reconstruye las contribuciones desde el historial de PR antes de HU-39.
- **Se trata en:** HU-10, HU-23, HU-33, HU-42, HU-39.

#### RIE-13 · Fuente, exportación y versión de diagramas desincronizadas
- **Causa:** fuente PlantUML y exportación SVG en PR distintos, leyenda sin versión o `CHANGELOG.md` sin actualizar.
- **Consecuencia:** el informe muestra figuras desactualizadas; pérdida en "Versionamiento de artefactos de diseño".
- **Indicador:** un diagrama fuente sin su par exportado, o con versión distinta en la leyenda; carpetas o tags que no coinciden entre `BACKLOG.md` y `README.md`/`CONTRIBUTING.md` (H-07).
- **Mitigación:** convención de BACKLOG §7.3 (mismo nombre, mismo PR, leyenda `ID · vX.Y.Z · AAAA-MM-DD · autor (Px) · HU-NN`); punto 3 de la DoD; ADR-008.
- **Contingencia:** HU-36 audita y sincroniza todo antes de `diseno-v1.0.0`.
- **Se trata en:** HU-21, HU-36.

### 3.3 Riesgos de alcance

#### RIE-14 · Contenido de Desarrollo de Software Moderno dentro de los artefactos
- **Causa:** al hablar de interacciones y eventos, las IAs proponen endpoints, formatos de mensaje, configuración de colas o pipelines.
- **Consecuencia:** el enunciado (§9) lo excluye; resta claridad y puede penalizarse.
- **Indicador:** términos como endpoint, método HTTP, código de estado, OpenAPI, topic, Docker o pipeline en `docs/` o `diagramas/`.
- **Mitigación:** ARQUITECTURA_BASE §14 y punto 7 de la checklist §6.3; las interacciones se describen solo con su INT (origen, destino, propósito, modo).
- **Contingencia:** se elimina en la revisión; HU-41 lo verifica antes de entregar.
- **Se trata en:** todas las HU; control en HU-41.

#### RIE-15 · Sobrediseño frente a la capacidad del equipo
- **Causa:** 19 HN de prioridad M, 42 HU y 141 pts en 4 sprints; tendencia a detallar SVC-05 (telemetría, rutas) o SVC-06 (predicciones) más allá del MVP.
- **Consecuencia:** el Sprint 4 no alcanza para la revisión integral, la trazabilidad y la presentación del flujo vertical.
- **Indicador:** el Sprint 2 cierra con menos del 80 % de los puntos planificados, o se arrastran HU a S3.
- **Mitigación:** priorizar el flujo vertical (ARQUITECTURA_BASE §6.3) y las HN de prioridad M; HN-22 y HN-23 (prioridad C) solo si sobra capacidad; refinamiento a mitad de sprint.
- **Contingencia:** P2 recorta el alcance de prioridad S/C y lo registra en el acta.
- **Se trata en:** HU-02, HU-04, cada planning.

#### RIE-16 · Requisitos de calidad incompletos o no verificables
- **Causa:** en §11.2 solo FAL-01, FAL-02 y FAL-03 tienen un RNF propio (RNF-DIS-01 a RNF-DIS-03); FAL-04 a FAL-09 no. Escalabilidad tiene un único RNF (RNF-ESC-01) y HU-27 CA1 exige al menos 2 por atributo. RNF-DIS-04 usa "horario operativo" sin definirlo en SUP-01 a SUP-03.
- **Consecuencia:** pérdida en "Requisitos de calidad" (1,0 pt); fallos sin un requisito que los haga verificables.
- **Indicador:** la tabla de cobertura FAL → RNF (§4.1) sigue con huecos al cerrar HU-27.
- **Mitigación:** HU-27 completa la cobertura (H-02 y H-04) con el formato fuente · estímulo · entorno · respuesta · medida y justifica umbrales con SUP-01 a SUP-03.
- **Contingencia:** los huecos que queden se declaran como supuestos y se registran como riesgo aceptado en este documento.
- **Se trata en:** HU-27, HU-29.

#### RIE-17 · Decisiones de arquitectura percibidas como desvío del enunciado
- **Causa:** el contrato arranca como monolito modular (ADR-001) cuando el enunciado habla de evolucionar hacia microservicios, y ubica la detección de riesgo en SVC-03 (ADR-006) cuando el Flujo B del enunciado admite que la detecte la proyección.
- **Consecuencia:** el evaluador puede leerlo como incumplimiento si no está justificado.
- **Indicador:** ADR sin alternativas o sin referencia al enunciado.
- **Mitigación:** ADR-001, ADR-003 y ADR-006 con al menos dos opciones y la cita explícita del enunciado (§1 "Regla de diseño": no se califica la cantidad de microservicios); ruta de evolución §2.4 visible en el C4 N2 (HU-12 CA4).
- **Contingencia:** el razonamiento se explica en la presentación (HU-40).
- **Se trata en:** HU-14, HU-17, HU-20, HU-40.

#### RIE-18 · Trazabilidad incompleta: HN sin artefacto o artefactos huérfanos
- **Causa:** los artefactos se producen en paralelo y la matriz de trazabilidad se arma recién en el Sprint 4 (HU-35).
- **Consecuencia:** HN de prioridad M sin diagrama, RNF o ADR, o artefactos sin HN; pérdida en "Trazabilidad y presentación".
- **Indicador:** PR sin HU/HN vinculada (punto 8 de la checklist §6.3); RNF sin SVC, INT o VL (HU-27 CA3).
- **Mitigación:** cada PR enlaza su HU y HN desde el Sprint 1; la columna "Diseñada en" de BACKLOG §2.2 se mantiene al día; este registro vincula cada riesgo a SVC, FAL o RNF.
- **Contingencia:** HU-35 identifica huérfanos y HU-34 los cierra antes del release.
- **Se trata en:** HU-27, HU-34, HU-35.

---

## 4. Alineación con ARQUITECTURA_BASE §11.2 y §11.3

### 4.1 Cobertura FAL → RNF → riesgo

| FAL | RNF semilla que lo verifica | Riesgos de este registro | Brecha |
|---|---|---|---|
| FAL-01 Broker no disponible | RNF-DIS-01 (y RNF-CON-02 para el aviso en la vista) | RIE-03, RIE-06 | — |
| FAL-02 SVC-02 no disponible | RNF-DIS-02 | RIE-02, RIE-04 | Definir qué significa dentro de UD-1 (H-03) |
| FAL-03 Telemetría o EXT-03 sin señal | RNF-DIS-03 | RIE-07 | — |
| FAL-04 SVC-06 no disponible | — | RIE-16 | Sin RNF (H-02) |
| FAL-05 EXT-01 ERP no disponible | — | RIE-07, RIE-16 | Sin RNF (H-02) |
| FAL-06 TRX-04 caída o corrupta | RNF-CON-02 (solo el aviso) | RIE-05 | Sin RNF de reconstrucción (H-02) |
| FAL-07 EXT-04 IdP no disponible | — | RIE-07, RIE-16 | Sin RNF (H-02) |
| FAL-08 Evento falla repetidamente | — (RNF-DIS-01 cubre solo la caída del broker) | RIE-03 | Sin RNF (H-02) |
| FAL-09 EXT-02 transportista sin respuesta | — | RIE-07 | Sin RNF (H-02) |

### 4.2 RNF semilla → riesgo que lo amenaza

| RNF | Qué exige (resumen) | Riesgos que lo amenazan |
|---|---|---|
| RNF-REN-01 | Consulta de existencias ≤ 500 ms p95 | RIE-02 |
| RNF-REN-02 | Reserva FEFO ≤ 1 s p95 normal, ≤ 2 s pico | RIE-02, RIE-04 |
| RNF-REN-03 | 95 % de recepciones en VL-01 en < 5 s | RIE-03, RIE-05 |
| RNF-REN-04 | Posición AMR en VL-04 ≤ 2 s p95 | RIE-07 |
| RNF-DIS-01 | Broker caído sin pérdida ni duplicados | RIE-03 |
| RNF-DIS-02 | Ningún pedido se pierde si SVC-02 no responde | RIE-02 |
| RNF-DIS-03 | AMR sin señal > 10 s marcado y sin misiones | RIE-07 |
| RNF-DIS-04 | UD-1 ≥ 99,5 % del horario operativo | RIE-02, RIE-16 |
| RNF-CON-01 | Disponible nunca negativo | RIE-04 |
| RNF-CON-02 | Toda vista muestra la antigüedad del dato | RIE-05 |
| RNF-SEG-01 | 100 % de OPS con rol verificado en el dueño y auditadas | RIE-08 |
| RNF-SEG-02 | 0 secretos en el repositorio | RIE-08 |
| RNF-ESC-01 | Escalamiento de TRX-04, SVC-06, UD-2 y UD-4 por señal | RIE-16 |
| RNF-MAN-01 | Nueva política = nueva estrategia, sin tocar el flujo | — (se evidencia en HU-31) |
| RNF-MAN-02 | 0 dependencias de datos cruzadas en los C4 | RIE-01, RIE-09 |

---

## 5. Hallazgos y propuestas de cambio

Ninguno de estos hallazgos modifica el contrato por sí mismo: cada uno se tramita por ARQUITECTURA_BASE §0.3 con el dueño indicado.

| ID | Hallazgo | Propuesta | Dueño / HU |
|---|---|---|---|
| H-01 | HN-09 pide la alerta en VL-02 en ≤ 5 s, pero CE-03 (≤ 5 s) + CE-01 (≤ 5 s) pueden sumar unos 10 s. No hay RNF de extremo a extremo para la alerta del flujo B. | «PROPUESTA DE CAMBIO»: en HU-27, agregar un RNF de extremo a extremo para la alerta del flujo B y ajustar CE-03 o el criterio de HN-09 para que sean coherentes (MINOR). | P5 (HU-27), P2 (HN-09), P1 (§7.3) |
| H-02 | FAL-04 a FAL-09 no tienen RNF semilla que los haga verificables. | «PROPUESTA DE CAMBIO»: agregar RNF de disponibilidad para esos fallos en HU-27 (MINOR). | P5 (HU-27), P1 (HU-29) |
| H-03 | FAL-02 y RNF-DIS-02 suponen SVC-04 operativo con SVC-02 caído; en el MVP ambos están en UD-1. | Aclarar en HU-29 cómo se manifiesta FAL-02 dentro de UD-1 y desde cuándo aplica RNF-DIS-02 completo. | P1 (HU-29) |
| H-04 | Escalabilidad tiene un solo RNF (RNF-ESC-01) y HU-27 CA1 exige 2; RNF-DIS-04 usa "horario operativo" sin definirlo en SUP-01 a SUP-03. | «PROPUESTA DE CAMBIO»: un segundo RNF de escalabilidad y la definición del horario operativo como supuesto en HU-27 (MINOR). | P5 (HU-27) |
| H-05 | §11.4 exige separación de funciones, pero no se declaran combinaciones de roles incompatibles (p. ej., ROL-02 con ROL-03). | Definir pares incompatibles junto con el umbral de OPS-03. | P2 (HU-28) |
| H-06 | §9 Pedido no define cómo sale un pedido de *EnEspera* (¿se reintenta la reserva al llegar stock?) ni explicita *PendienteReserva* → *Reservado*. Conecta el cierre del flujo B (recepción de la OC) con el flujo C. | Revisar las transiciones en HU-18 y HU-26. | P6 (HU-18, HU-26) |
| H-07 | `README.md` y `CONTRIBUTING.md` del repositorio no coinciden con BACKLOG §3.1, §7.1 y §8.1: formato de rama (`feat/HU-nn-...` vs `hu-NN-...`), commit, título de PR, carpetas de diagramas (`docs/diagramas/fuente` vs `diagramas/.../src`) y tag del Sprint 4 (`diseno-v0.4.0` vs `diseno-v1.0.0`). | Unificar en una sola convención por PR y registrarlo en el acta. Mientras tanto, este PR sigue `CONTRIBUTING.md`. | P3 (HU-01, HU-21) |

---

## 6. Seguimiento

- **Cuándo:** en cada sprint review (HU-08 CA3). P5 actualiza probabilidad, impacto, estado y la columna *Última revisión*; los cambios se registran en el registro del sprint (HU-10, HU-23, HU-33, HU-42).
- **Riesgos nuevos:** se agregan con el siguiente `RIE-nn`; nunca se reutiliza un número.
- **Prioridad para el Sprint 2:** RIE-09, RIE-01, RIE-05, RIE-10 y RIE-15 (exposición alta y afectan a los artefactos de S2). RIE-16 se ataca en el Sprint 3 con HU-27.

---

## Historial de cambios de este documento

| Versión | Fecha | Autor | Cambio | HU |
|---|---|---|---|---|
| 1.0.0 | 2026-11-23 | P5 | Registro inicial: 18 riesgos, alineación con §11.2 y §11.3, hallazgos H-01 a H-07 | HU-08 |
