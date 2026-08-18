# Flujo de contenido

Flujo editorial completo, desde investigación hasta publicación. Cada flecha
es un *handoff* explícito (patrón de orquestación por código, no delegación
libre del modelo — ver [harness-engineering.md](../investigacion/harness-engineering.md) §2).

```
┌────────────────────────┐   ┌──────────────────────────┐
│ Investigador de        │   │ Investigador de           │
│ noticias                │   │ contenidos                │
│ (agente-investigador-   │   │ (agente-investigador-     │
│ noticias.md)             │   │ contenidos.md)            │
└───────────┬─────────────┘   └────────────┬──────────────┘
            │  señales/temas               │  dossiers
            └───────────────┬──────────────┘
                             ▼
                 ┌─────────────────────────┐
                 │ Creador de contenidos    │
                 │ (agente-creador-         │
                 │ contenidos.md)           │
                 └────────────┬─────────────┘
                              │ borrador
                              ▼
                 ┌─────────────────────────┐
                 │ Corrección de estilo     │
                 │ (agente-estilo.md)       │
                 └────────────┬─────────────┘
                              │ borrador revisado
                              ▼
                 ┌─────────────────────────┐
                 │ SEO / GEO / AEO          │
                 │ (agente-seo.md)          │
                 └────────────┬─────────────┘
                              │ borrador + metadatos
                              ▼
                 ┌─────────────────────────┐
                 │ REVISIÓN HUMANA (gate)   │  ← regla 0.3 AGENTS.md
                 │ aprueba / rechaza /      │
                 │ pide cambios             │
                 └────────────┬─────────────┘
                              │ aprobado
                              ▼
                 ┌─────────────────────────┐
                 │ Publicación              │
                 │ (transición de estado    │
                 │ borrador → publicado)    │
                 └────────────┬─────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐   ┌─────────────────────────┐
                 │ Documentador              │   │ Historiador                │
                 │ (registra en memoria/,    │   │ (si hay una historia real, │
                 │ siempre)                   │   │ registra en historia/)     │
                 └─────────────────────────┘   └─────────────────────────┘
```

`project_writer` sigue la tarjeta correspondiente en el tablero
("site danemar") a lo largo de todo este flujo: `Backlog` al crearse →
`Ready`/`In progress` mientras se redacta/revisa → `In review` en el gate
humano → `Done` al publicar. Ver
[agente-project-writer.md](../agentes/agente-project-writer.md).

> Desde que existen [orquestador](../agentes/agente-orquestador.md) y
> [planificador](../agentes/agente-planificador.md), este flujo normalmente
> se dispara desde ellos (ver [flujo-orquestacion.md](flujo-orquestacion.md))
> en vez de invocarse directo — aunque invocarlo directo sigue siendo válido
> para tareas puntuales.

## Disparadores

- **Manual**: el usuario pide un tema/actualización concreta.
- **Periódico** (opcional, requiere scheduler — ver
  [opciones-implementacion.md](../opciones-implementacion.md)): el
  investigador de noticias corre con una cadencia definida (p. ej. semanal) y
  solo escala al creador de contenidos cuando detecta algo con señal real —
  evita generar contenido por generar.

## Puntos de rollback

- Si el agente de estilo o SEO rechaza el borrador, vuelve al creador de
  contenidos con motivo concreto (no silencioso — regla 0.2).
- Si el humano rechaza en el gate final, el ciclo completo puede repetirse
  desde investigación (si el problema es de fondo) o solo desde redacción (si
  es de forma).

## Estado de contenido necesario

Este flujo asume un estado *borrador* separable del *publicado*. Desde
2026-08-18 el sitio **tiene `content_moderation`/`workflows` habilitados**,
con el workflow "Editorial" (`draft` → `review` → `published`) — ver
[estado-actual.md](../analisis-proyecto/estado-actual.md) §5 y
[MANUAL-DE-USO.md](../MANUAL-DE-USO.md) §4.
