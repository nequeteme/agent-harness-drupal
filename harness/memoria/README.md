# Memoria del harness

Sistema de memoria de corto y largo plazo, mantenido por el
[agente documentador](../agentes/agente-documentador.md). Es conocimiento del
**proyecto/plataforma**, no de Claude Code como herramienta — vive en el
repo para que cualquier agente (o cualquier sesión futura) lo pueda leer, a
diferencia de la memoria propia de Claude Code (que es del usuario, no del
proyecto).

## Cómo está organizada

- **[corto-plazo.md](corto-plazo.md)** — log rotativo de qué se hizo en las
  últimas tareas/sesiones y qué quedó pendiente. No crece para siempre: el
  documentador "gradúa" lo estable hacia largo plazo y recorta lo obsoleto.
- **[largo-plazo/](largo-plazo/)** — conocimiento consolidado por tema, un
  archivo por área (`contenido.md`, `seo.md`, `desarrollo.md`,
  `decisiones-arquitectura.md`, etc.). Esto es lo que un agente nuevo debería
  leer antes de empezar una tarea en esa área.

## Cómo debe usarla cada agente

Antes de empezar una tarea, un agente debería revisar
`largo-plazo/<su-área>.md` si existe (evita repetir decisiones ya tomadas o
errores ya conocidos). Al terminar, reporta al
[orquestador](../agentes/agente-orquestador.md), que le pide al
[documentador](../agentes/agente-documentador.md) registrar lo relevante.
Ningún agente de ejecución escribe memoria directamente — eso es trabajo del
documentador, para mantener un solo criterio de qué vale la pena guardar.
