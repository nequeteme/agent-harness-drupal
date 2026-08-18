# Orchestration flow (meta-flow)

This is the flow that wraps the other two ([content-flow](content-flow.md), [dev-testing-flow](dev-testing-flow.md)) — it describes how the user interacts with the harness day to day, now that [orchestrator](../agents/agent-orchestrator.md), [planner](../agents/agent-planner.md), [documenter](../agents/agent-documenter.md), [project-writer](../agents/agent-project-writer.md), and [historian](../agents/agent-historian.md) exist.

```
User ◄──────────────────────────────────────────────────┐
  │  natural-language request               card created by hand
  ▼                                        on the GitHub Project
┌─────────────────────────┐                        │
│ Orchestrator              │  (skill, governs the  │
│ (agent-orchestrator.md)   │   main session)        │
└────────────┬───────────────┘                        │
             │ clarified goal                  ┌───────┴─────────┐
             ▼                                │ project-writer      │
┌─────────────────────────┐                   │ (agent-project-     │
│ Planner                    │  (subagent —    │ writer.md)           │
│ (agent-planner.md)         │   ≤60% context) │ detects new card     │
└────────────┬───────────────┘                 │ from the user         │
             │ phase/task plan                 └───────┬─────────┘
             ▼                                          │ hands it to
     ┌───────┴────────┐                                the orchestrator
     │                │  project-writer creates a
     │                │  card per task on the
     │                │  project board (Backlog)
     ▼                ▼
content-flow      dev-testing-flow
(if applicable,    (if applicable, tag `develop`)
 tag `content`)
     │                │
     │  project-writer moves the card:
     │  Ready → In progress → In review → Done
     └───────┬────────┘
             │ reports (pass/fail, drafts, findings)
             ▼
┌─────────────────────────┐   ┌──────────────────────────┐
│ Documenter                 │   │ Historian                  │
│ (agent-documenter.md)      │   │ (agent-historian.md)       │
│ records in memory/         │   │ if there's a real story,   │
│ (always)                    │   │ records in history/        │
└────────────┬───────────────┘   └────────────┬────────────────┘
             │                                 │
             └────────────────┬────────────────┘
                               ▼
                          Orchestrator
                               │  consolidated summary + approval questions if needed
                               ▼
                            User
```

In parallel, without depending on a user request: the [harness improvement researcher](../agents/agent-harness-improvement-researcher.md) runs roughly every ~2 days (see the cadence in its own document) and delivers its report directly to the Orchestrator, which folds it into the next status summary it gives the user — it doesn't interrupt the main flow.

## When it's OK to skip the orchestrator (and what can never be skipped)

It's still valid to invoke `/content-flow`, `/dev-flow`, or a standalone agent directly (see [USER-GUIDE.md](../USER-GUIDE.md) §3) — useful for one-off tasks where prior planning isn't needed. What is **never** skipped is communication with the user: if that directly-invoked flow reaches a point where it needs to present something or ask for approval, that step is handled by the orchestrator's rules (see "Centralized-communication rule" in [agent-orchestrator.md](../agents/agent-orchestrator.md)), not by the flow improvising the question on its own. In practice: skipping the orchestrator saves the planning step, it doesn't change who talks to the user.

## What changes from the original design

Originally, "orchestration" was implicit: the user invoked a flow skill and that skill dispatched directly to the execution agents. Now there's an explicit layer of planning (Planner) and reporting/memory (Documenter) between the user and that execution — the goal is for the user to stop having to know which flow/agent to use for each thing, and instead talk to a single entry point that knows how to delegate and consolidate.
