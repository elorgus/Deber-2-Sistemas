# Revisión cruzada de PR

| Campo | Valor |
|---|---|
| PR | #nn · `HU-nn: Descripción breve` |
| Autor · Revisor | Px · Px |
| HU y artefactos | HU-nn · `ruta/del/artefacto` |

## Checklist de revisión
- [ ] El artefacto está en la carpeta correcta (`equipo/P?-...` o `docs/...`).
- [ ] Los nombres de SVC, TRX, EXT, EVT, INT, VL, estados y puertos son idénticos a `ARQUITECTURA_BASE.md`.
- [ ] Ningún diagrama muestra a un módulo leyendo o escribiendo datos de otro (PRI-02).
- [ ] Cada interacción indica S o A y coincide con su INT.
- [ ] Los eventos están en pasado y los emite solo su dueño.
- [ ] El dashboard y la proyección no aparecen como origen de decisiones ni como fuente de verdad.
- [ ] Fuente y exportación presentes y sincronizadas; leyenda completa (`ID · vX.Y.Z · AAAA-MM-DD · autor · HU-nn`).
- [ ] Sin contenido fuera de alcance: endpoints, payloads, JSON/OpenAPI, broker, CI/CD, pruebas, despliegue.
- [ ] Se citan los IDs correctos y se vincula a su HU/HN.
- [ ] `CHANGELOG.md` actualizado (si aplica).
- [ ] Sin secretos ni datos personales reales.
- [ ] Issue referenciado (`Closes #nn`).

## Resultado
- [ ] Aprobado
- [ ] Cambios solicitados

Comentarios:
