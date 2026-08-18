# Harness health metrics

Metrics skeleton — starts empty, fills in as real tasks/flows happen. This is
not a log of everything that happens (that's
[short-term.md](../short-term.md)); it's a structured, aggregable record of
the metrics that are actually measurable with the tools available in this
environment.

## Who fills it in and who reads it

- **Fills it in**: the [documenter](../../agents/agent-documenter.md), when
  closing out each task/flow — the same moment it appends the corresponding
  entry to [short-term.md](../short-term.md). For metric 2
  (`tester`↔`pr-reviewer` rounds), the documenter depends on the
  orchestrator or `project-writer` (which has `gh` access) reporting the
  data — the documenter does not run `gh` itself.
- **Reads it**: the
  [harness-improvement-researcher](../../agents/agent-harness-improvement-researcher.md),
  on each periodic run, as part of its "system health" report section —
  trends, and in particular whether an agent's metrics improved after
  preloading a new skill for it.

## 1. `tester` pass/fail by task type, and attempts until pass

| Date | Task | Type (content/development/frontend) | Implementing agent | Attempts until pass | Final result |
|---|---|---|---|---|---|
| | | | | | |

## 2. `tester`↔`pr-reviewer` rounds before a clean PR

| Date | PR (link) | Tester rounds | PR-reviewer rounds | Blocking findings |
|---|---|---|---|---|
| | | | | |

## 3. Tasks re-planned due to sizing (60% context rule)

| Date | Original task | Reason (context compacted / no verification margin) | How it was re-sized |
|---|---|---|---|
| | | | |
