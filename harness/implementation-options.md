# Harness implementation options

Four concrete ways to build this for a project, from lower to higher infrastructure investment. None require new modules to *start* — all of them can begin with what most Drupal projects already have (Drush, Playwright, an `AGENTS.md`) and grow from there.

---

## Option A — Claude Code subagents and skills, manual orchestration

**What it is**: each harness agent (style, SEO, content writer, content researcher, news researcher, Drupal developer, frontend, tester) is defined as a Claude Code subagent (`.claude/agents/*.md`) and the flows as skills (`.claude/skills/content-flow.md`, `dev-flow.md`, commands `/content-flow`/`/dev-flow`) that invoke the subagents in sequence. The "hands" are Drush scripting + Playwright. The human approval gate is, literally, the user approving inside the Claude Code session (rule 0.3 already works this way).

| | |
|---|---|
| **Scope** | Every agent and flow requested, operating on this same repo. |
| **Capabilities it enables** | Assisted content generation/review, SEO auditing, development+testing with an automatic loop, all triggerable from a Claude Code session. |
| **Advantages** | Zero installation in Drupal. Reuses already-validated patterns (Drush, Playwright, `AGENTS.md`). Fastest to get running (days, not weeks). Prompt iteration is trivial (they're Markdown files in the repo). |
| **Disadvantages** | Doesn't run "on its own" — needs someone to open a Claude Code session to trigger each step (though `ScheduleWakeup`/`/loop` allow autonomous pacing within an active session). No UI for a non-technical editor to use the agents without going through Claude Code. Memory/state limited to repo files, not a native Drupal moderation system. |
| **Estimated effort** | Low. Write the subagent files + skills, and (if you want a real draft state) enable `content_moderation`+`workflows` (core, no composer required). |

---

## Option B — Native in Drupal: `ai` + `ai_agents` + `mcp_server`

**What it is**: install Drupal's AI ecosystem directly on the site (`ai`, `ai_agents`/`ai_agents_ossa`, `mcp_server`, `simple_oauth`), so the agents live **inside** Drupal, triggered by `content_moderation` transitions or by ECA (Event-Condition-Action), and accessible from the admin panel itself. See [drupal-ai-mcp.md](research/drupal-ai-mcp.md).

| | |
|---|---|
| **Scope** | The development/config agents (Field Type Agent, Content Type Agent, Taxonomy Agent from `ai_agents`) come "out of the box"; the rest (style, SEO, content, news) would need to be built as custom agents on top of `ai_agents_ossa` + ECA. |
| **Capabilities it enables** | Any content editor (without Claude Code access) can trigger an agent from the Drupal UI. Real automation via cron/ECA without depending on an open external session. Aligns with Drupal core's 2026 AI roadmap. |
| **Advantages** | Integrated into the product: outlives this project, available to any future human editor. Uses Drupal's native governance (permissions, content_moderation) as a guardrail, aligned with the 5-stage "Outside AI" pattern. |
| **Disadvantages** | `ai_agents_ossa`, `mcp_server`, and the experimental meta-module are early in maturity — confirm version/compatibility with your Drupal version before committing. Requires managing model-provider API keys inside Drupal (a new security surface). Learning curve for ECA + `ai_agents` for the team. More moving parts to maintain/update going forward. |
| **Estimated effort** | High. Installation, OAuth configuration, compatibility testing, and you still need to build most of the 8 agents from scratch. |

---

## Option C — External harness with MCP as a bridge

**What it is**: build an independent agent runtime (outside Drupal, e.g. a Node/Python service in this same repo or a new one), with a plugin architecture, that connects to Drupal via the `mcp_server` module (Drupal exposes nodes/Paragraphs as MCP tools) instead of direct Drush.

| | |
|---|---|
| **Scope** | The agents as provider-independent plugins, reusable across future projects (not just this one). Supports real scheduled execution (the runtime's own cron, not dependent on an interactive session). |
| **Capabilities it enables** | A content/news pipeline running unattended 24/7. Full client decoupling (usable from Claude Code, Claude Desktop, or any MCP client). Reusable as your agency's own product/offering toward other Drupal clients. |
| **Disadvantages** | Higher engineering effort: you have to build and maintain the harness runtime yourself (there's no mature, stable "off-the-shelf" one). Still requires enabling `mcp_server`+OAuth in Drupal (same maturity risk as Option B at that layer). Over-engineering for a small site — only worth it if the real goal is a reusable product, not just automating one site. |
| **Estimated effort** | Very high. Weeks, not days; only worth it if the actual goal is a reusable product. |

---

## Option D — Hybrid (recommended)

**What it is**: start with **Option A** for everything that benefits from close human supervision and broad repo access (content creation, style, SEO, development, frontend, testing), and reserve native Drupal automation (`content_moderation` + `workflows`, both core, nothing to install via Composer) **only** to give content a real draft/review/published state. Don't install `ai_agents`/`mcp_server` yet — keep them documented as an evolution path (Option B) for when there's a real need for a non-technical editor to trigger agents without a Claude Code session, or for the news researcher to run unattended on a fixed cadence.

| | |
|---|---|
| **Scope** | All the requested agents and flows, operational in days, with a real draft state via `content_moderation`. |
| **Advantages** | Lower risk (nothing new to maintain in production yet). Reuses 100% of already-proven patterns (Drush, Playwright, `AGENTS.md`). Leaves a clear path to B or C when the project justifies it, without having thrown away work. |
| **Disadvantages** | Same limitations as Option A until you take the next step: nobody outside a Claude Code session can trigger an agent; the news researcher doesn't run 24/7 unless someone keeps a session active or configures `/loop`/external cron. |
| **Estimated effort** | Low-medium. Same as A, plus enabling and configuring `content_moderation`/`workflows` (core). |

---

## Comparison summary

| | A. Claude Code | B. Native Drupal | C. External harness | D. Hybrid |
|---|---|---|---|---|
| Installation in Drupal | None (core optional) | High (several contrib modules) | Medium (mcp_server) | None (core optional) |
| Runs unattended 24/7 | No (requires a session) | Yes | Yes | No (same as A) |
| Usable by a non-technical editor | No | Yes | Partial (via an MCP client) | No |
| Maturity of the pieces used | High (all proven patterns) | Low-medium (new/experimental modules) | Low (early-stage external runtimes) | High |
| Reusable across other projects | Partial | Partial | High | Partial |
| Time to first working version | Days | Weeks | Weeks-months | Days |

**Recommendation**: Option D. It's the one that delivers all the requested agents and flows fastest, with the lowest risk, without closing the door to evolving toward B or C later if content volume or the need for 24/7 autonomy justifies it.

---

## Evolution path: from D to B

D isn't a separate alternative to B — it's deliberately designed as B's foundation. Nothing implemented in D blocks or needs undoing to reach B, but it isn't free either: there's real work involved.

**Reused without changes**

- `content_moderation` + `workflows` (core), already enabled in D, are exactly the same draft → review → published gate used in B's diagram (see [architecture-maps.md](architecture-maps.md)). Nothing to touch there.
- Content already created under moderation in D (drafts, revision history) remains valid — no data migration needed.
- Each agent's specification in [agents/](agents/) (inputs, outputs, acceptance criteria, gates) is the *design*, independent of the runtime — it works equally well for writing a Claude Code subagent or an ECA action.

**Not reused — has to be rebuilt**

- The orchestration itself: a Claude Code `.md` subagent and an `ai_agents` plugin/ECA action are two different runtimes. Moving from D to B means reimplementing each agent's logic in PHP/ECA, not just moving a file.
- Installing and configuring `ai`, `ai_agents`/`ai_agents_ossa`, `mcp_server`, `simple_oauth` (composer + new config), and verifying real compatibility with your Drupal version (see [drupal-ai-mcp.md](research/drupal-ai-mcp.md)).
- Managing model-provider API keys *inside* Drupal (a new security surface that doesn't exist today).
- Defining the ECA/cron triggers that replace "a human runs a Claude Code session".

**It's not all-or-nothing**: the most likely migration is partial — keep development/frontend/testing in Claude Code (they benefit from full repo access) and move only the content/news agents to B once there's a real need for them to run 24/7 without an open session.
