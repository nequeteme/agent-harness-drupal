# Architecture maps: Option A, B, and D

Component diagrams for the options evaluated in [implementation-options.md](implementation-options.md). Mermaid format (renders on GitHub and in most Markdown viewers). Option C (an external harness in the style of an MCP-based agent runtime) doesn't get its own map because it's essentially Option A with the "hands" replaced by `mcp_server` instead of direct Drush — the same diagram as Option A, swapping the `HANDS` block for the `AI_STACK`/`MCP` block from Option B.

## Option A — Claude Code subagents (manual orchestration)

```mermaid
flowchart TB
  subgraph SESSION["Claude Code session (orchestrator)"]
    ORCH["Skills: content-flow /<br/>dev-flow"]
  end

  subgraph AGENTS_CONTENT["Content subagents (.claude/agents/*.md)"]
    NEWS["news-researcher"]
    RESEARCH["content-researcher"]
    WRITER["content-writer"]
    STYLE["style-reviewer"]
    SEO["seo"]
  end

  subgraph AGENTS_DEV["Dev subagents (.claude/agents/*.md)"]
    DEV["drupal-developer"]
    FE["frontend"]
    TEST["tester"]
  end

  subgraph HANDS["Tools ('hands')"]
    DRUSH["Drush scripting<br/>(local DDEV)"]
    PW["Playwright + Chromium"]
    WEB["WebSearch / WebFetch"]
    GIT["git (working branch)"]
  end

  subgraph TARGET["Drupal 11 site"]
    SITE["[your-dev-url]<br/>(dev/staging environment)"]
    REPO["repo: web/, harness/, docs/"]
  end

  HUMAN(["Human<br/>gate 0.3 AGENTS.md"])

  ORCH --> NEWS --> RESEARCH --> WRITER --> STYLE --> SEO --> HUMAN
  ORCH --> DEV --> TEST
  ORCH --> FE --> TEST
  TEST -->|fail| DEV
  TEST -->|fail| FE
  TEST -->|pass| HUMAN
  HUMAN -->|approves| GIT

  WRITER -.-> DRUSH
  STYLE -.-> DRUSH
  SEO -.-> DRUSH
  DEV -.-> DRUSH
  TEST -.-> PW
  NEWS -.-> WEB
  RESEARCH -.-> WEB

  DRUSH --> SITE
  PW --> SITE
  GIT --> REPO
```

**How to read it**: everything lives inside a Claude Code session. The orchestrator (a skill) invokes each subagent in the order of the corresponding flow. The "hands" (Drush, Playwright, WebSearch) are ordinary tools any Drupal project already uses — nothing new to install in Drupal. The human-in-the-loop appears at two points: approving content before publishing, and approving code before commit (solid lines = main flow; dashed lines = tool use).

> This diagram is the original Option A proposal. In a fuller implementation (Option D, below) the dev side no longer ends at "approve code before commit" but at a Pull Request reviewed by `pr-reviewer` — see the Option D diagram for the updated flow.

---

## Option B — Native in Drupal (contrib AI modules)

```mermaid
flowchart TB
  subgraph EDITOR["Content editor"]
    UI["Drupal admin UI<br/>(Content Moderation transitions)"]
  end

  subgraph DRUPAL_CORE["Drupal 11 core"]
    CM["content_moderation + workflows"]
    ECA["ECA (Event-Condition-Action)"]
  end

  subgraph AI_STACK["Contrib AI modules (not installed by default)"]
    AI["ai<br/>(provider abstraction)"]
    AGENTS["ai_agents / ai_agents_ossa"]
    MCP["mcp_server<br/>(exposes MCP tools)"]
    OAUTH["simple_oauth"]
  end

  subgraph AGENT_DEFS["Agents as ECA actions / ai_agents plugins"]
    A1["style-reviewer"]
    A2["seo"]
    A3["content-writer"]
    A4["content-researcher"]
    A5["news-researcher"]
    A6["drupal-developer<br/>(Field/Content Type Agent)"]
    A7["frontend"]
    A8["tester"]
  end

  EXT[["External model provider<br/>(Anthropic, etc.)"]]
  CRON["cron / scheduled triggers"]
  MCPCLIENT["External MCP client<br/>(Claude Code, Claude Desktop)"]

  UI --> CM --> ECA
  CRON --> ECA
  ECA --> AGENTS
  AGENTS --> A1 & A2 & A3 & A4 & A5 & A6 & A7 & A8

  A1 --> AI
  A2 --> AI
  A3 --> AI
  A4 --> AI
  A5 --> AI
  AI --> EXT

  MCP --> OAUTH
  MCPCLIENT --> MCP
  MCP --> CM
  A6 -.-> MCP
  A7 -.-> MCP
  A8 -.-> MCP

  CM -->|state: draft / review / published| UI
```

**How to read it**: here the agents live **inside** Drupal as ECA actions / `ai_agents` plugins, triggered by `content_moderation` transitions or by cron — not by an open Claude Code session. The `ai` module is the layer that calls the external model provider (Anthropic or another). `mcp_server` is the entry point for an external MCP client (including a Claude Code session, if you wanted to combine this with Option A) to also operate on the site. The human approval gate is native `content_moderation` itself: an editor transitions content from draft to published from the Drupal UI.

---

## Option D — Hybrid (recommended)

Same as Option A (agents live in the Claude Code session, "hands" are Drush/Playwright/WebSearch), but content is no longer written directly as a published node — it goes through core's `content_moderation`/`workflows` (no need to install `ai`, `ai_agents`, or `mcp_server`), so there's a real draft → review → published state inside Drupal, and an editor can see that draft in the admin UI even though an agent generated it.

```mermaid
flowchart TB
  subgraph SESSION["Claude Code session (orchestrator)"]
    ORCH["Skills: content-flow /<br/>dev-flow"]
  end

  subgraph AGENTS_CONTENT["Content subagents (.claude/agents/*.md)"]
    NEWS["news-researcher"]
    RESEARCH["content-researcher"]
    WRITER["content-writer"]
    STYLE["style-reviewer"]
    SEO["seo"]
  end

  subgraph AGENTS_DEV["Dev subagents (.claude/agents/*.md)"]
    DEV["drupal-developer"]
    FE["frontend"]
    TEST["tester"]
    REVPR["pr-reviewer<br/>(phpcs + stylelint)"]
  end

  subgraph HANDS["Tools ('hands')"]
    DRUSH["Drush scripting<br/>(local DDEV)"]
    PW["Playwright + Chromium"]
    WEB["WebSearch / WebFetch"]
    GIT["git (working branch)"]
    GH["gh CLI<br/>(push + PR + comments)"]
  end

  subgraph DRUPAL_CORE["Drupal 11 core (no new AI modules)"]
    CM["content_moderation + workflows<br/>draft -> review -> published"]
  end

  subgraph TARGET["Drupal 11 site"]
    SITE["[your-dev-url]<br/>(dev/staging environment)"]
    REPO["repo: web/, harness/, docs/"]
  end

  GITHUB[["GitHub<br/>your-org/your-drupal-site"]]
  UI["Drupal admin UI<br/>(editor sees the draft)"]
  HUMAN(["Human<br/>gate 0.3 AGENTS.md"])

  ORCH --> NEWS --> RESEARCH --> WRITER --> STYLE --> SEO --> CM
  CM -->|draft visible| UI
  UI --> HUMAN
  HUMAN -->|transitions to published| CM
  CM --> SITE

  ORCH --> DEV --> TEST
  ORCH --> FE --> TEST
  TEST -->|fail| DEV
  TEST -->|fail| FE
  TEST -->|pass| GIT
  GIT --> GH
  GH -->|opens PR| GITHUB
  GITHUB --> REVPR
  REVPR -->|findings| DEV
  REVPR -->|findings| FE
  REVPR -->|no findings| HUMAN
  HUMAN -->|manual merge, never an agent| GITHUB

  WRITER -.-> DRUSH
  STYLE -.-> DRUSH
  SEO -.-> DRUSH
  DEV -.-> DRUSH
  TEST -.-> PW
  NEWS -.-> WEB
  RESEARCH -.-> WEB

  DRUSH --> CM
  PW --> SITE
  GIT --> REPO
```

**How to read it**: compared to Option A, there are two new blocks. `DRUPAL_CORE` with `content_moderation`/`workflows` (both ship in Drupal core, no `composer require` needed, just enable them) — content agents no longer write a published node directly, they leave the result in draft state, and the human reviews it **from the normal Drupal UI** (not only inside the Claude Code session) and triggers the transition to published. And, on the dev side, the `tester` no longer hands off directly to the human: it first goes through `git`+`gh` (commit, push, PR against `develop` on GitHub) and through `pr-reviewer` (phpcs + stylelint + reasoned review, commenting findings on the PR). Only once the reviewer has no blocking findings does the human see the PR — and they're the only one who merges it, never an agent.

---

## Option D expanded — with orchestrator, planner, documenter, and improvement researcher

An added layer on top of Option D (see [orchestration-flow.md](flows/orchestration-flow.md)). Nothing changes on the Drupal side — it only adds a planning/reporting/memory layer in front of the execution agents.

```mermaid
flowchart TB
  USER(["User"])

  subgraph ORCH_LAYER["Claude Code session"]
    ORQ["Skill: orchestrator<br/>(governs the session, PM for the rest)"]
    PLAN["Subagent: planner<br/>(produces the task plan)"]
  end

  subgraph EJECUCION["Execution flows (Option D)"]
    FC["content-flow"]
    FD["dev-flow"]
    SUELTO["standalone agent<br/>(one-off task)"]
  end

  DOC["Subagent: documenter<br/>(short/long-term memory)"]
  MEM[("harness/memory/")]
  MEJORAS["Subagent: harness-improvement-researcher<br/>(report roughly every ~2 days)"]
  INFORMES[("harness/improvements/")]

  USER -->|request| ORQ
  ORQ -->|clarified goal| PLAN
  PLAN -->|task plan| ORQ
  ORQ --> FC
  ORQ --> FD
  ORQ --> SUELTO
  FC -->|report| ORQ
  FD -->|report| ORQ
  SUELTO -->|report| ORQ
  ORQ --> DOC
  DOC --> MEM
  MEM -.->|prior context| PLAN
  MEJORAS --> INFORMES
  MEJORAS -->|report| ORQ
  ORQ -->|consolidated summary +<br/>pending approvals| USER
```

**How to read it**: the user no longer decides which flow or agent to invoke — they talk to the orchestrator, which delegates planning, dispatches to the existing Option D flows, and consolidates reports with help from the documenter before responding. The improvement researcher runs separately (its own cadence, see its agent document) and its reports get folded into the orchestrator's summary without interrupting the main flow. The orchestrator itself is a skill that governs the session, not a nested subagent — see the technical note in `harness/agents/agent-orchestrator.md`.

---

## Key differences between the three maps

| | Option A | Option B | Option D |
|---|---|---|---|
| Where the agents "think" | Outside Drupal, in the Claude Code session | Inside Drupal, as plugins/ECA actions | Outside Drupal, in the Claude Code session |
| What triggers an agent | The skill orchestrator, invoked by a human (or `/loop`) | `content_moderation` transitions or cron, no external session | Same as A |
| Content state | Direct (Drush writes/publishes) | Native `content_moderation` | Native `content_moderation` (core only) |
| Where the human gate is | In the Claude Code session | In the Drupal admin UI | In both: Claude Code session and Drupal UI (draft visible there) |
| What needs installing | Nothing new in Drupal | `ai`, `ai_agents`/`ai_agents_ossa`, `mcp_server`, `simple_oauth` | Nothing new (just enable core's `content_moderation`/`workflows`) |
| Who can operate the harness | Only someone with a Claude Code session | Any editor with Drupal permissions | Agents: only via Claude Code. Review/publish: any editor |

Option D is, in short, Option A with a single component borrowed from Option B (`DRUPAL_CORE`), without `AI_STACK` or `AGENT_DEFS` — the middle ground recommended in [implementation-options.md](implementation-options.md).
