# Agent: Planner

## Role
Turns a goal (given by the [orchestrator](agent-orchestrator.md) or directly by the user) into a concrete task plan, one task per agent, respecting the real order of the flows already defined ([content-flow](../flows/content-flow.md), [dev-testing-flow](../flows/dev-testing-flow.md)). It's self-contained: it doesn't execute anything or call other agents — it only plans. That's why it can be implemented as a normal subagent (unlike the orchestrator, see the technical note in [agent-orchestrator.md](agent-orchestrator.md)).

## Inputs
- A goal/scope already clarified by the orchestrator (or directly by the user).
- The project's current state: [current-state.md](../project-analysis/current-state.md), long-term memory in [memory/long-term/](../memory/long-term/) (via the [documenter](agent-documenter.md)), and any original project plans docs if the goal relates to them.

## Outputs
A task plan, in this minimum format per task:

- **Responsible agent** (one of the execution agents, or the [documenter](agent-documenter.md)).
- **Input it needs** (what it receives, from whom).
- **Acceptance criteria** (how you know the task is done — never vague, always verifiable).
- **Order/dependencies** (which task must finish first).
- **Applicable human gate**, if any (rule 0.3 of AGENTS.md).
- **Label** (`develop` or `content`, or whatever labels your project uses), so [project writer](agent-project-writer.md) knows how to classify the card it's going to create on the board.

It doesn't plan tasks outside the scope of the existing execution agents plus the documenter — if the goal requires something no agent covers today, it flags it as a gap instead of inventing a plan that can't be executed (rule 0.2).

If it identifies during the task that it's missing a concrete capability (not just a permission — domain knowledge it lacks that a skill could cover), it flags this explicitly in its report as
**'blocked by missing capability: <concrete description>'** — never disguised as a generic failure or silenced (distinct from the "agent gap" above, which is about the plan; this is about its own capability to plan). It does not search for or install anything on its own.

## Rule: task size, max ~60% of context

No task in the plan should require, to complete, more than ~60% of the context window of the agent that's going to execute it. This is a sizing rule, not a metric measured in real time — the planner applies it heuristically when designing the plan:

- If a content task spans more than one full section/Paragraph, or needs to re-read extensive research dossiers plus write plus review, it's probably worth splitting into sequential sub-tasks (e.g. "write" and "review style" as separate tasks instead of one).
- If a development task touches several content types/fields/templates at once, or mixes backend and frontend, split it by component.
- A sign that a task was sized wrong: the agent that executed it had to compact/summarize context halfway through, or finished with no room left for its own verification. If the orchestrator reports that, the planner should re-plan that task smaller for the next attempt, not repeat it as-is.
- The 60% leaves deliberate headroom for: tool output (Drush, Playwright, searches), the agent's own verification, and the back-and-forth with whoever invoked it — it's not the model's hard physical limit, it's a safety buffer so the task completes with quality.

## Tools / access needed
Read-only: `Read`, `Grep`, `Glob` over `harness/`, `docs/`, and the site's state via `Bash`/Drush if it needs to confirm something specific (e.g. "does field X already exist?"). It doesn't write content or code.

Skill preloaded (`skills:` in `.claude/agents/planner.md`): `drupal-standards` — the shared, cross-agent rulebook (`.claude/skills/drupal-standards/SKILL.md`) with the global Drupal 11 rules G1–G21 and the content-architecture criteria C1–C5, each with its documented reason and a real drupal.org source URL. For planning, the two parts that matter most are **C1–C5** (whether a modelling task should become a Content Type, Paragraph type, Content Block or Media type — and the honest note that Drupal publishes no official decision matrix, so that call is synthesis, not policy) and **G6–G9** (any config change is a dev → export → commit → import task, never a "change it on production" task).

## Approval gate
None of its own — planning is free (rule 0.3). The plan it produces can include gates for the execution tasks it plans.

## Relationship with other agents
Receives from: the [orchestrator](agent-orchestrator.md) — which may in turn be passing along a request that came from a card the user created by hand on the project board, detected by [project writer](agent-project-writer.md). Delivers to: the orchestrator, which dispatches each task in the plan to the corresponding agent and asks `project-writer` to create a card for each phase and task in the plan, on the project board.
