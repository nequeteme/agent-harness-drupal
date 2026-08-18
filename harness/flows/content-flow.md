# Content flow

The full editorial flow, from research to publication. Every arrow is an explicit *handoff* (code-driven orchestration, not free-form delegation by the model — see [harness-engineering.md](../research/harness-engineering.md) §2).

```
┌────────────────────────┐   ┌──────────────────────────┐
│ News researcher          │   │ Content researcher         │
│ (agent-news-             │   │ (agent-content-            │
│ researcher.md)            │   │ researcher.md)              │
└───────────┬─────────────┘   └────────────┬──────────────┘
            │  signals/topics               │  dossiers
            └───────────────┬──────────────┘
                             ▼
                 ┌─────────────────────────┐
                 │ Content writer            │
                 │ (agent-content-           │
                 │ writer.md)                 │
                 └────────────┬─────────────┘
                              │ draft
                              ▼
                 ┌─────────────────────────┐
                 │ Style reviewer            │
                 │ (agent-style-reviewer.md) │
                 └────────────┬─────────────┘
                              │ reviewed draft
                              ▼
                 ┌─────────────────────────┐
                 │ SEO / GEO / AEO          │
                 │ (agent-seo.md)          │
                 └────────────┬─────────────┘
                              │ draft + metadata
                              ▼
                 ┌─────────────────────────┐
                 │ HUMAN REVIEW (gate)      │  ← rule 0.3 AGENTS.md
                 │ approve / reject /      │
                 │ request changes          │
                 └────────────┬─────────────┘
                              │ approved
                              ▼
                 ┌─────────────────────────┐
                 │ Publication               │
                 │ (state transition        │
                 │ draft → published)        │
                 └────────────┬─────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐   ┌─────────────────────────┐
                 │ Documenter                 │   │ Historian                  │
                 │ (records in memory/,      │   │ (if there's a real story,  │
                 │ always)                    │   │ records in history/)       │
                 └─────────────────────────┘   └─────────────────────────┘
```

`project-writer` tracks the corresponding card on the project board throughout this whole flow: `Backlog` when created → `Ready`/`In progress` while it's being written/reviewed → `In review` at the human gate → `Done` when published. See [agent-project-writer.md](../agents/agent-project-writer.md).

> Now that [orchestrator](../agents/agent-orchestrator.md) and [planner](../agents/agent-planner.md) exist, this flow is normally triggered from them (see [orchestration-flow.md](orchestration-flow.md)) instead of being invoked directly — though invoking it directly is still valid for one-off tasks.

## Triggers

- **Manual**: the user requests a specific topic/update.
- **Periodic** (optional, requires a scheduler — see [implementation-options.md](../implementation-options.md)): the news researcher runs on a defined cadence (e.g. weekly) and only escalates to the content writer when it detects something with real signal — this avoids generating content just to generate content.

## Rollback points

- If the style or SEO agent rejects the draft, it goes back to the content writer with a concrete reason (never silent — rule 0.2).
- If the human rejects at the final gate, the whole cycle can repeat from research (if the problem is substantive) or only from writing (if it's a matter of form).

## Content state required

This flow assumes a *draft* state separable from *published*. This requires `content_moderation`/`workflows` to be enabled, with an editorial workflow (`draft` → `review` → `published`) — see [current-state.md](../project-analysis/current-state.md) §5 and [USER-GUIDE.md](../USER-GUIDE.md) §4.
