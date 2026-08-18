# Flujo de orquestación (meta-flujo)

Este es el flujo que envuelve a los otros dos
([flujo-contenido](flujo-contenido.md),
[flujo-desarrollo-testing](flujo-desarrollo-testing.md)) — describe cómo el
usuario interactúa con el harness en el día a día, ahora que existen
[orquestador](../agentes/agente-orquestador.md),
[planificador](../agentes/agente-planificador.md),
[documentador](../agentes/agente-documentador.md),
[project_writer](../agentes/agente-project-writer.md) y
[historiador](../agentes/agente-historiador.md).

```
Usuario ◄──────────────────────────────────────────────────┐
  │  pedido en lenguaje natural            tarjeta creada a mano
  ▼                                        en GitHub Projects
┌─────────────────────────┐                        │
│ Orquestador               │  (skill, gobierna la  │
│ (agente-orquestador.md)   │   sesión principal)   │
└────────────┬───────────────┘                        │
             │ objetivo clarificado           ┌───────┴─────────┐
             ▼                                │ project_writer     │
┌─────────────────────────┐                   │ (agente-project-   │
│ Planificador               │  (subagente —   │ writer.md)          │
│ (agente-planificador.md)   │   ≤60% contexto)│ detecta tarjeta     │
└────────────┬───────────────┘                 │ nueva del usuario   │
             │ plan de fases/tareas            └───────┬─────────┘
             ▼                                          │ se la entrega
     ┌───────┴────────┐                                al orquestador
     │                │  project_writer crea una
     │                │  tarjeta por tarea en
     │                │  "site danemar" (Backlog)
     ▼                ▼
flujo-contenido   flujo-desarrollo-testing
(si aplica,        (si aplica, tag `develop`)
 tag `contenido`)
     │                │
     │  project_writer mueve la tarjeta:
     │  Ready → In progress → In review → Done
     └───────┬────────┘
             │ reportes (pass/fail, borradores, hallazgos)
             ▼
┌─────────────────────────┐   ┌──────────────────────────┐
│ Documentador               │   │ Historiador                │
│ (agente-documentador.md)   │   │ (agente-historiador.md)    │
│ registra en memoria/       │   │ si hay una historia real,  │
│ (siempre)                   │   │ registra en historia/      │
└────────────┬───────────────┘   └────────────┬────────────────┘
             │                                 │
             └────────────────┬────────────────┘
                               ▼
                          Orquestador
                               │  resumen consolidado + preguntas de aprobación si hacen falta
                               ▼
                            Usuario
```

En paralelo, sin depender de un pedido del usuario: el
[investigador de mejoras del harness](../agentes/agente-investigador-mejoras-harness.md)
corre cada ~2 días (ver cadencia en su propio documento) y entrega su informe
directo al Orquestador, que lo suma al siguiente resumen de estado que le dé
al usuario — no interrumpe el flujo principal.

## Cuándo saltarse el orquestador (y qué no se puede saltar nunca)

Sigue siendo válido invocar `/dp-cont`, `/dp-dev`,
o un agente suelto directamente (ver
[MANUAL-DE-USO.md](../MANUAL-DE-USO.md) §3) — útil para tareas puntuales
donde no hace falta planificación previa. Lo que **nunca** se salta es la
comunicación con el usuario: si ese flujo invocado directo llega a un punto
donde necesita presentar algo o pedir una aprobación, ese paso se maneja con
las reglas del orquestador (ver "Regla de comunicación centralizada" en
[agente-orquestador.md](../agentes/agente-orquestador.md)), no con el flujo
improvisando la pregunta por su cuenta. En la práctica: saltarse el
orquestador ahorra el paso de planificación, no cambia quién le habla al
usuario.

## Qué cambia respecto al diseño original (Opción D)

Antes, la "orquestación" era implícita: el usuario invocaba una skill de
flujo y esa skill dispatchaba directo a los agentes de ejecución. Ahora hay
una capa explícita de planificación (Planificador) y de reporte/memoria
(Documentador) entre el usuario y esa ejecución — el objetivo es que el
usuario deje de tener que saber qué flujo/agente usar para cada cosa, y en
cambio hable con un solo punto de entrada que sabe delegar y consolidar.
