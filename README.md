# Smart Inventory — Diseño de Sistemas (USFQ)

Plataforma empresarial integrada de inventario, fulfillment y operación logística, diseñada como monolito modular evolutivo hacia microservicios.

**Materia:** Diseño de Sistemas — Colegio Politécnico, Universidad San Francisco de Quito
**Entrega:** enlace al repositorio + informe PDF/Word + presentación final
**Fecha base del deber:** 2026-11-17
**Versión vigente:** contrato `ARQUITECTURA_BASE.md` v1.0.0. El último paquete de diseño publicado está en [CHANGELOG.md](https://github.com/elorgus/Deber-2-Sistemas/blob/main/CHANGELOG.md).

## Documentos rectores

| Documento | Qué contiene |
|---|---|
| [docs/ARQUITECTURA_BASE.md](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/ARQUITECTURA_BASE.md) | Contrato maestro: capacidades (SVC), datos propietarios, eventos (EVT), interacciones (INT), vistas (VL), RNF, ADR iniciales. Fuente única de nombres. |
| [docs/BACKLOG.md](https://github.com/elorgus/Deber-2-Sistemas/blob/main/docs/BACKLOG.md) | Equipo, roles, épicas (EP), historias (HN/HU), plan de 4 sprints, Definition of Ready/Done y estructura del repo. |
| [CONTRIBUTING.md](https://github.com/elorgus/Deber-2-Sistemas/blob/main/CONTRIBUTING.md) | Convención de ramas, PR, revisión cruzada, cómo se agrega un ADR y qué queda fuera de alcance. |
| [CHANGELOG.md](https://github.com/elorgus/Deber-2-Sistemas/blob/main/CHANGELOG.md) | Historial versionado de los artefactos de diseño. |

## Regla de contribución

Nada entra a `docs/` ni a `diagramas/` sin un Pull Request aprobado por el revisor asignado. Cada cambio parte de una rama `feat/HU-nn-...` y sigue las reglas de [CONTRIBUTING.md](https://github.com/elorgus/Deber-2-Sistemas/blob/main/CONTRIBUTING.md).

## Alcance de la entrega

Se evalúa desde Diseño de Sistemas: arquitectura C4, límites y responsabilidades, propiedad de datos, flujos de negocio (recepción, reposición, reserva-FEFO-despacho), interacciones conceptuales, patrones, SOLID, requisitos de calidad verificables, ADR, gestión de sprints y versionamiento.

**Fuera de alcance:** diseño detallado de APIs, contratos técnicos de mensajes/colas, CI/CD, DevOps, pruebas y despliegue (corresponden a Desarrollo de Software Moderno).

## Equipo

| ID | Usuario GitHub | Rol | Dueño de |
|---|---|---|---|
| P1 | elorgus | Líder de arquitectura | Contrato maestro, C4 N1–N2, ADR de límites, disponibilidad y escalabilidad |
| P2 | EstebanEsp07 | Product Owner | Visión, MVP, épicas e historias, dashboard, presentación final |
| P3 | RichJ-100 | Scrum Master | Repositorio, ceremonias, versionamiento, release |
| P4 | joshdavr | Diseño Catálogo e Inventario | SVC-01, SVC-02, C4 N3 Inventario, matriz de trazabilidad |
| P5 | JessPCL | Diseño Reposición y calidad | SVC-03, flujo B, RNF, riesgos, informe final |
| P6 | CamGranda | Diseño Pedidos y Flota | SVC-04, SVC-05, flujo C, matriz de interacciones |
| P7 | luisdavidborjap-web | Patrones y SOLID | C4 N1, CQRS, informe de patrones, evidencia SOLID |

## Plan de sprints (resumen)

| Sprint | Objetivo | Tag |
|---|---|---|
| S1 | Contrato y backlog aprobados, repo y tablero listos | `diseno-v0.1.0` |
| S2 | C4 N1–N3, propiedad de datos, flujos A/B/C, ADR iniciales | `diseno-v0.2.0` |
| S3 | Interacciones, calidad, patrones, SOLID, estados | `diseno-v0.3.0` |
| S4 | Revisión integral, trazabilidad, release, presentación | `diseno-v1.0.0` |

## Estructura del repositorio

```text
/
├── docs/
│   ├── ARQUITECTURA_BASE.md
│   ├── BACKLOG.md
│   ├── VERSIONAMIENTO.md
│   ├── adr/                 (ADR-001...009.md)
│   ├── diagramas/
│   │   ├── fuente/          (drawio, puml, mmd)
│   │   └── export/          (png, svg)
│   └── informe/             (informe consolidado)
├── equipo/
│   ├── P1-elorgus/
│   ├── P2-EstebanEsp07/
│   ├── P3-RichJ-100/
│   ├── P4-joshdavr/
│   ├── P5-JessPCL/
│   ├── P6-CamGranda/
│   └── P7-luisdavidborjap-web/
├── plantillas/              (plantilla-adr, plantilla-sprint, plantilla-revision-pr)
├── CONTRIBUTING.md
├── CHANGELOG.md
└── README.md
```
