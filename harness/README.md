# Harness engineering — Danemar Parceros

Mapa completo de la investigación, análisis y diseño de un harness de agentes
de IA para este proyecto.

> **Estado: la Opción D está implementada.** `content_moderation`/`workflows`
> habilitados con workflow "Editorial" (draft/review/published), 14
> subagentes en `.claude/agents/` y 3 skills de orquestación
> (`/dp`, `/dp-cont`, `/dp-dev`) en `.claude/skills/`. El flujo de desarrollo
> ahora termina en Pull Request revisado (`gh`, `phpcs`, `stylelint`), nunca
> en commit directo — el merge lo hace siempre el usuario. Cada tarea vive
> también como tarjeta en el tablero de GitHub Projects "site danemar"
> (mantenido por `project_writer`), y las tarjetas que el usuario cree ahí a
> mano entran solas al harness. Ver el **[MANUAL-DE-USO.md](MANUAL-DE-USO.md)**
> para cómo usarlo. El resto de esta carpeta sigue siendo la documentación
> de diseño/investigación de fondo.

## Cómo navegar esta carpeta

1. **[investigacion/](investigacion/)** — qué es un "harness" y qué existe hoy
   en el ecosistema.
   - [harness-engineering.md](investigacion/harness-engineering.md) — concepto,
     componentes estándar de un harness, y el repo `deepseek-harness`
     confirmado en GitHub.
   - [drupal-ia-mcp.md](investigacion/drupal-ia-mcp.md) — módulos de IA/MCP de
     drupal.org, iniciativa oficial de IA de Drupal, arquitectura "Outside AI".

2. **[analisis-proyecto/estado-actual.md](analisis-proyecto/estado-actual.md)**
   — foto real de este proyecto: qué existe (arquitectura de contenido, SEO,
   módulos, theme), qué no existe todavía (moderación de contenido, API
   externa, IA), y qué patrones ya probados (Drush, Playwright) sirven de
   base al harness.

3. **[agentes/](agentes/)** — un documento por cada agente, con rol,
   entradas, salidas, herramientas necesarias, gate de aprobación y relación
   con los demás. Los 8 de ejecución:
   - [agente-estilo.md](agentes/agente-estilo.md)
   - [agente-seo.md](agentes/agente-seo.md)
   - [agente-creador-contenidos.md](agentes/agente-creador-contenidos.md)
   - [agente-investigador-contenidos.md](agentes/agente-investigador-contenidos.md)
   - [agente-investigador-noticias.md](agentes/agente-investigador-noticias.md)
   - [agente-desarrollo-drupal.md](agentes/agente-desarrollo-drupal.md)
   - [agente-frontend.md](agentes/agente-frontend.md)
   - [agente-tester.md](agentes/agente-tester.md)
   - [agente-revisor-pr.md](agentes/agente-revisor-pr.md) — revisa cada PR
     (phpcs/stylelint + buenas prácticas), comenta hallazgos, nunca mergea.

   Más 4 de coordinación/meta:
   - [agente-orquestador.md](agentes/agente-orquestador.md) — punto de
     entrada único, "product manager" del resto.
   - [agente-planificador.md](agentes/agente-planificador.md) — convierte un
     objetivo en un plan de tareas por agente.
   - [agente-documentador.md](agentes/agente-documentador.md) — memoria de
     corto/largo plazo, mantiene el conocimiento de la plataforma al día.
   - [agente-investigador-mejoras-harness.md](agentes/agente-investigador-mejoras-harness.md)
     — investiga cómo mejorar el harness mismo, informe cada ~2 días.
   - [agente-project-writer.md](agentes/agente-project-writer.md) — crea y
     sincroniza tarjetas en el tablero de GitHub Projects "site danemar";
     detecta tarjetas del usuario y se las entrega al orquestador.
   - [agente-historiador.md](agentes/agente-historiador.md) — registra en
     [historia/](historia/) los aprendizajes/fallos reales, insumo para
     futuro contenido de blog.

4. **[flujos/](flujos/)** — cómo se conectan los agentes entre sí:
   - [flujo-orquestacion.md](flujos/flujo-orquestacion.md) — el meta-flujo:
     usuario/tablero ↔ orquestador ↔ planificador → project_writer →
     ejecución → documentador + historiador.
   - [flujo-contenido.md](flujos/flujo-contenido.md) — investigación → creación
     → estilo → SEO → revisión humana → publicación → documentador.
   - [flujo-desarrollo-testing.md](flujos/flujo-desarrollo-testing.md) —
     desarrollo/frontend → testing (loop) → commit+push+PR → revisor de PRs
     (loop) → merge humano → documentador.

5. **[memoria/](memoria/)** — la memoria de corto y largo plazo que
   mantiene el documentador, disponible para cualquier agente o sesión
   futura (incluye `project-tracker.md` y `roadmap.md`, del project_writer).

6. **[historia/](historia/)** — narrativa (no estado operativo) de
   aprendizajes/fallos reales, mantenida por el historiador, pensada como
   insumo para el blog que hagan `creador-contenidos`/`investigador-contenidos`.

7. **[opciones-implementacion.md](opciones-implementacion.md)** — el
   entregable final: 4 formas concretas de construir esto en el proyecto
   (A. subagentes de Claude Code, B. nativo en Drupal con módulos de IA,
   C. harness externo tipo `deepseek-harness` con MCP, D. híbrida
   recomendada), con alcance, ventajas, desventajas y esfuerzo de cada una.

8. **[mapas-arquitectura.md](mapas-arquitectura.md)** — diagramas Mermaid de
   componentes para las opciones A, B, D y D ampliada (con orquestador/
   planificador/documentador), con tabla comparativa de dónde vive cada
   agente, qué lo dispara y dónde está el gate humano.

9. **[MANUAL-DE-USO.md](MANUAL-DE-USO.md)** — cómo usar en la práctica lo ya
   implementado: cómo disparar los flujos, cómo invocar un agente suelto,
   cómo funciona el estado de moderación, y los límites reales de esta
   implementación.

## Principio transversal

Todos los agentes y flujos diseñados aquí heredan las reglas de
[AGENTS.md](../AGENTS.md): honestidad (0.2 — nada de "funciona" sin
verificar, nada inventado), aprobación humana explícita antes de tocar código
o publicar contenido (0.3), y verificación real contra
`https://site-dev.danemarparceros.net/` antes de dar algo por terminado (0.4).
El harness no reemplaza esas reglas — las automatiza dentro de ellas.
