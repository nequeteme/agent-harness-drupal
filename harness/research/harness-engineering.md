# Research: what "harness engineering" is

> Source: web research (August 2026). See source list at the end. Anything
> marked "unconfirmed" is honest about the limits of the search.

## 1. Definition

A **harness** is the software scaffolding that wraps a language model (LLM)
to turn it into an agent capable of operating reliably and sustainably,
beyond a single one-off response. Anthropic compares it to an operating
system: it curates the context window, manages prompts and hooks, and
exposes tools and "skills" to the model.

A recurring breakdown (Anthropic, WaveSpeed, Hatchworks) splits the harness
into three layers:

- **Brain** — the model plus the logic that decides the next step, routes
  tool calls, and re-injects results.
- **Hands** — sandboxes and tools that execute real actions: reading/writing
  files, running commands, calling APIs, git.
- **Session** — the append-only log of everything that happened (to resume
  context, audit, or let another session continue the work).

## 2. Standard components of a harness

| Component | What it solves |
|---|---|
| **Context/memory management** | Compaction when the window fills up, on-disk persistence (progress files, git log) so a new session can reconstruct state without prior memory. |
| **Multi-agent orchestration** | Proven patterns: (a) an *initializer* agent (prepares environment, backlog) plus an *executor* agent (advances one task per session); (b) *planner → generator → evaluator* for long-running development; (c) *handoffs* between specialist agents orchestrated by code (OpenAI's pattern), more deterministic than letting the LLM decide on its own who gets the turn. |
| **Tool definition** | Invocation layer via MCP or SDK-tools, with explicit per-tool permissions. |
| **Verification / testing loops** | Anthropic reports that end-to-end verification with browser automation catches far more real bugs than unit tests alone. The agent must verify before declaring itself "done". |
| **Feedback loops / checkpoints** | Reading progress + git log at session start, reviewing the backlog (prevents the agent from declaring itself "done" prematurely), one task per cycle. |
| **Guardrails** | Input/output validation that blocks risky actions; human approvals that pause execution before sensitive actions (exactly what an AGENTS.md-style approval rule already requires at the process level). |
| **Logging / observability** | Traces of model calls, tools, handoffs, and guardrails in a single record; useful for auditing what each agent did and why. |

These components **already exist, partially, in a typical project**: an
AGENTS.md-style file acting as guardrails plus approval rules, a progress
log/memoria acting as a checkpoint log, and a verification pattern combining
scripted changes with browser automation is, in effect, already a manual
mini-harness. See the [project analysis](../project-analysis/current-state.md).

## 3. The DeepSeek repo — confirmed

**`deepseek-ai/deepseek-harness`** (CLI `dsh`), at
https://github.com/deepseek-ai/deepseek-harness — confirmed via multiple
cross-checked sources (GitHub, tech press, DeepSeek's official page).

- Describes itself as "an open-source agent harness" with the tagline
  **"Everything is a Plugin"**.
- Built on **Cordis**, a composable-architecture framework: the model
  adapter, the tool registry, the session log, and the agent loop are all
  replaceable plugins — there's no fixed monolithic core.
- CLI + Web UI (`npx @deepseek-ai/dsh web`, on `127.0.0.1:3080`).
- **Developer preview** (MIT, Node.js 22.19+/24+), with a breaking-changes
  warning; not yet accepting external PRs (points to Discussions / your own
  plugins instead).

**Applicable lesson**: the "everything is a plugin" pattern fits naturally
with Drupal, which is already an ecosystem of interchangeable modules. It
suggests designing each piece of a Drupal harness (model adapter, Drupal
publishing tool, SEO validator, logger) as a replaceable unit instead of
coupling the agent loop to a fixed provider.

**Unconfirmed**: no DeepSeek "eval harness" distinct from the general
`deepseek-harness` was found, nor a recorded DrupalCon talk on "agent
harness" (DrupalCon Rotterdam 2026 runs Sept 28–Oct 1, hadn't happened yet
at the time of this research).

## 4. Sources

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
