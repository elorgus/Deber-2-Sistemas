# Política de Versionamiento de Artefactos

## 1. Versionamiento Semántico (MAJOR, MINOR, PATCH)
- **MAJOR:** Cambia un límite de servicio, se elimina o renombra un SVC, EVT, INT o VL.
- **MINOR:** Se agrega algo nuevo compatible, como un EVT, INT, VL, RNF o un diagrama nuevo.
- **PATCH:** Corrección de forma sin cambiar el significado, como corregir una etiqueta en un diagrama.

## 2. Nombres y Leyendas de Diagramas
Los diagramas se nombran con el formato TIPO-ALCANCE-descripcion (por ejemplo, C4N1-contexto). 
Todos los diagramas deben llevar la leyenda obligatoria: ID · vX.Y.Z · AAAA-MM-DD · autor (Px) · HU-NN.

## 3. Ciclo de vida de los ADR
Los ADR pasan por los estados: Propuesto -> Aceptado -> Reemplazado por ADR-nnn o Deprecado. Un ADR que ya fue aceptado no se edita; si se necesita cambiar, se crea uno nuevo.

## 4. Tags por Sprint
Al cierre de cada sprint, el Scrum Master crea un tag para el paquete de diseño: diseno-v0.1.0 (S1), diseno-v0.2.0 (S2), diseno-v0.3.0 (S3), diseno-v1.0.0 (S4).
