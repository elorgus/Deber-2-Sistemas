# Política de Versionamiento de Artefactos

## 1. Versionamiento Semántico (MAJOR, MINOR, PATCH)
[Aquí debes explicar, según el Backlog §7.2, cuándo un cambio es de cada tipo. Ejemplo:]
- **MAJOR:** Cuando cambia un límite de servicio o se elimina/renombra un SVC, EVT, INT o VL.
- **MINOR:** Cuando se agrega algo nuevo compatible (un EVT, INT, diagrama).
- **PATCH:** Corrección de forma sin cambiar el significado (etiquetas, errores de ortografía).

## 2. Nombres y Leyendas de Diagramas
[Aquí explica cómo se nombran los archivos, ej: TIPO-ALCANCE-descripcion. Y la leyenda obligatoria: ID · vX.Y.Z · AAAA-MM-DD · autor (Px) · HU-NN]

## 3. Ciclo de vida de los ADR
[Explica que pasan por: Propuesto -> Aceptado -> Reemplazado o Deprecado. Y que un ADR aceptado no se edita].

## 4. Tags por Sprint
[Explica que al cierre de cada sprint se crea un tag: diseno-v0.1.0 (S1), diseno-v0.2.0 (S2), etc.]

## 5. Ejemplo real de cambio versionado
[El CA2 pide un ejemplo real del Sprint 2. Puedes poner: "En el Sprint 2, se agregó el flujo A (HU-16) como un cambio MINOR, pasando el paquete a la versión diseno-v0.2.0..."]