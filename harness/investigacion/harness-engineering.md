# Investigación: qué es "harness engineering"

> Fuente: investigación web (agosto 2026). Ver lista de fuentes al final. Todo lo
> marcado como "no confirmado" es honesto sobre los límites de la búsqueda (ver
> [AGENTS.md](../../AGENTS.md) 0.2).

## 1. Definición

Un **harness** ("arnés") es el andamiaje de software que envuelve a un modelo de
lenguaje (LLM) para convertirlo en un agente capaz de operar de forma fiable y
sostenida, más allá de una sola respuesta suelta. Anthropic lo compara con un
sistema operativo: cura la ventana de contexto, gestiona prompts y hooks, y
expone herramientas y "skills" al modelo.

Un desglose recurrente (Anthropic, WaveSpeed, Hatchworks) separa el harness en
tres capas:

- **Cerebro** — el modelo + la lógica que decide el siguiente paso, enruta
  llamadas a herramientas y reinyecta resultados.
- **Manos** — sandboxes y herramientas que ejecutan acciones reales: leer/escribir
  archivos, correr comandos, llamar APIs, git.
- **Sesión** — el log append-only de todo lo ocurrido (para retomar contexto,
  auditar, o que otra sesión continúe el trabajo).

## 2. Componentes estándar de un harness

| Componente | Qué resuelve |
|---|---|
| **Gestión de contexto/memoria** | Compactación cuando la ventana se llena, persistencia en disco (archivos de progreso, git log) para que una sesión nueva reconstruya estado sin memoria previa. |
| **Orquestación multi-agente** | Patrones probados: (a) agente *inicializador* (prepara entorno, backlog) + agente *ejecutor* (avanza una tarea por sesión); (b) *planner → generator → evaluator* para desarrollo largo; (c) *handoffs* entre agentes especialistas orquestados por código (patrón de OpenAI), más determinista que dejar que el LLM decida solo a quién pasarle el turno. |
| **Definición de herramientas** | Capa de invocación vía MCP o SDK-tools, con permisos explícitos por herramienta. |
| **Verificación / testing loops** | Anthropic reporta que la verificación end-to-end con automatización de navegador detecta muchos más bugs reales que solo tests unitarios. El agente debe verificar antes de darse por "terminado". |
| **Feedback loops / checkpoints** | Leer progreso + git log al iniciar sesión, revisar backlog (evita que el agente se declare "listo" prematuramente), una tarea por ciclo. |
| **Guardrails** | Validación de entrada/salida que bloquea acciones riesgosas; aprobaciones humanas que pausan la ejecución antes de acciones sensibles (esto es exactamente lo que ya exige [AGENTS.md](../../AGENTS.md) 0.3 en este proyecto). |
| **Logging / observabilidad** | Trazas de llamadas al modelo, herramientas, handoffs y guardrails en un registro único; útil para auditar qué hizo cada agente y por qué. |

Estos componentes **ya existen, parcialmente, en este proyecto**: `AGENTS.md`
funciona como guardrail + reglas de aprobación, `docs/plan-memoria.md` funciona
como log de progreso/checkpoint, y el patrón de verificación con Drush +
Playwright (usado en los 13 planes) ya es, de hecho, un mini-harness manual. Ver
[análisis del proyecto](../analisis-proyecto/estado-actual.md).

## 3. El repo de DeepSeek — confirmado

**`deepseek-ai/deepseek-harness`** (CLI `dsh`), en
https://github.com/deepseek-ai/deepseek-harness — confirmado con múltiples
fuentes cruzadas (GitHub, prensa técnica, página oficial de DeepSeek).

- Se autodescribe como "an open-source agent harness" con lema **"Everything is
  a Plugin"**.
- Construido sobre **Cordis**, un framework de arquitecturas componibles: el
  adaptador de modelo, el registro de herramientas, el log de sesión y el loop
  del agente son todos plugins reemplazables — no hay núcleo monolítico fijo.
- CLI + Web UI (`npx @deepseek-ai/dsh web`, en `127.0.0.1:3080`).
- **Developer preview** (MIT, Node.js 22.19+/24+), con advertencia de breaking
  changes; no acepta PRs externos todavía (dirige a Discussions / plugins
  propios).

**Lección aplicable**: el patrón "todo es un plugin" encaja de forma natural con
Drupal, que ya es un ecosistema de módulos intercambiables. Sugiere diseñar cada
pieza del harness (adaptador de modelo, herramienta de publicación en Drupal,
validador de SEO, logger) como una unidad reemplazable en vez de acoplar el loop
del agente a un proveedor fijo.

**No confirmado**: no se encontró un "eval harness" de DeepSeek distinto del
`deepseek-harness` general, ni una charla de DrupalCon ya grabada sobre "agent
harness" (DrupalCon Rotterdam 2026 es del 28 sept al 1 oct, aún no ha ocurrido).

## 4. Fuentes

- [Anthropic — Building Effective AI Agents](https://www.anthropic.com/research/building-effective-agents)
- [Anthropic — Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic — Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- [WaveSpeed — Claude Code Agent Harness: Architecture Breakdown](https://wavespeed.ai/blog/posts/claude-code-agent-harness-architecture/)
- [Hatchworks — Claude Agent SDK: Build Your Own Agent Harness](https://hatchworks.com/blog/claude/agent-sdk/)
- [Udacity — What is the Claude Agent SDK](https://www.udacity.com/blog/what-is-the-claude-agent-sdk-and-why-engineers-are-building-their-own-harnesses/)
- [vtrivedy — The Claude Code SDK and the Birth of HaaS](https://www.vtrivedy.com/posts/claude-code-sdk-haas-harness-as-a-service/)
- [AddyOsmani.com — Agent Harness Engineering](https://addyosmani.com/blog/agent-harness-engineering/)
- [GitHub — ai-boost/awesome-harness-engineering](https://github.com/ai-boost/awesome-harness-engineering)
- [OpenAI — Agents SDK: Orchestration and handoffs](https://developers.openai.com/api/docs/guides/agents/orchestration)
- [OpenAI Agents SDK — Guardrails](https://openai.github.io/openai-agents-python/guardrails/)
- [OpenAI — A practical guide to building AI agents](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- [GitHub — deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)
- [DeepSeek — Harness developer preview](https://deepseek.com/harness/en/)
- [The New Stack — DeepSeek open sources an agent harness where everything is a plugin](https://thenewstack.io/deepseek-harness-open-source-plugins/)
