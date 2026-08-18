# Métricas de salud del harness

Esqueleto de métricas — arranca vacío, se llena a medida que ocurren
tareas/flujos reales. No es un log de todo lo que pasa (eso es
`memoria/corto-plazo.md`); es un registro estructurado y agregable de las 3
métricas que hoy son realmente medibles con las herramientas de este
entorno (ver `harness/mejoras/2026-08-18-propuestas.md`, sección 6.5, para
el razonamiento de por qué estas 3 y no otras — p. ej. tiempo/turnos por
tarea queda fuera por ahora, no hay telemetría de sesión disponible).

## Quién la llena y quién la lee

- **La llena**: el [documentador](../../agentes/agente-documentador.md), al
  cerrar cada tarea/flujo — mismo momento en que agrega la entrada
  correspondiente a `memoria/corto-plazo.md`. Para la métrica 2 (rondas
  `tester`↔`revisor-pr`), el documentador depende de que el orquestador o
  `project_writer` (que sí tiene `gh`) le reporten el dato — el documentador
  no corre `gh` él mismo.
- **La lee**: el [investigador de mejoras del harness](../../agentes/agente-investigador-mejoras-harness.md),
  en cada corrida periódica (~2 días), como parte de su informe — sección de
  "salud del sistema": tendencias, y en particular si después de precargar
  una skill nueva en un agente las métricas de ese agente mejoraron o no.

## 1. Pass/fail del `tester` por tipo de tarea, y cuántos intentos hasta pass

| Fecha | Tarea | Tipo (contenido/desarrollo/frontend) | Agente que implementó | Intentos hasta pass | Resultado final |
|---|---|---|---|---|---|
| | | | | | |

## 2. Rondas `tester`↔`revisor-pr` antes de un PR limpio

| Fecha | PR (link) | Rondas de tester | Rondas de revisor-pr | Hallazgos bloqueantes encontrados |
|---|---|---|---|---|
| | | | | |

## 3. Tareas re-planificadas por mal dimensionamiento (regla del 60% de contexto)

| Fecha | Tarea original | Motivo (compactó contexto / sin margen para verificación) | Cómo se re-dimensionó |
|---|---|---|---|
| | | | |
