# Dev & testing flow

Flow for code/config changes (backend or frontend), until "everything works correctly" before being considered done — this is literally what the user asked for and matches rule 0.2 of AGENTS.md. This flow always ends in a **reviewed Pull Request**, never a direct commit — the final merge is done by the user.

```
┌─────────────────────────┐   ┌──────────────────────────┐
│ Drupal developer          │   │ Frontend                   │
│ (agent-drupal-            │   │ (agent-frontend.md)        │
│ developer.md)              │   │                             │
└────────────┬──────────────┘   └────────────┬───────────────┘
             │  implements on a working branch │
             └────────────────┬───────────────┘
                               ▼
                  ┌──────────────────────────┐
                  │ Tester                    │
                  │ (agent-tester.md)        │
                  │ Drush + Playwright against │
                  │ [your-dev-url]              │
                  └────────────┬───────────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
                  FAIL                  PASS
                    │                     │
                    ▼                     ▼
        back to the agent that    ┌──────────────────────────┐
        implemented it, with      │ Commit + push + PR         │  ← ALWAYS,
        concrete evidence         │ (no task ends without       │    no exceptions
        of the failure            │ this)                        │
                                   └────────────┬─────────────────┘
                                                │
                                                ▼
                                   ┌──────────────────────────┐
                                   │ PR reviewer                 │
                                   │ (agent-pr-reviewer.md)      │
                                   │ phpcs + stylelint + reasoned │
                                   │ review, comments on the PR   │
                                   └────────────┬─────────────────┘
                                                │
                                     ┌──────────┴──────────┐
                                     │                     │
                          blocking findings         no blocking findings
                                     │                     │
                                     ▼                     ▼
                       back to the agent that    ┌──────────────────────────┐
                       implemented it (new         │ Orchestrator tells the    │
                       commit on the same          │ user: PR ready             │
                       branch, re-tested if it     └────────────┬─────────────┘
                       affects behavior)                       │
                                                              ▼
                                                 ┌──────────────────────────┐
                                                 │ USER merges                │  ← human gate,
                                                 │ (never an agent)            │    rules 0.3/0.5
                                                 └────────────┬─────────────────┘
                                                              │
                                                              ▼
                                    ┌──────────────────────────┐   ┌──────────────────────────┐
                                    │ Documenter                  │   │ Historian                    │
                                    │ (records in memory/,        │   │ (if there's a real story,    │
                                    │ always)                     │   │ records in history/)         │
                                    └──────────────────────────┘   └──────────────────────────┘
```

`project-writer` tracks the corresponding card on the project board throughout this whole flow: `Backlog` when created → `Ready`/`In progress` while implementing/testing → `In review` while the PR is open → `Done` when merged. See [agent-project-writer.md](../agents/agent-project-writer.md).

> Now that [orchestrator](../agents/agent-orchestrator.md) and [planner](../agents/agent-planner.md) exist, this flow is normally triggered from them (see [orchestration-flow.md](orchestration-flow.md)) instead of being invoked directly — though invoking it directly is still valid for one-off tasks.

## Cycle rule

The `implement → test → fix` loop repeats as many times as needed **before** the PR is opened — the user shouldn't see a failed iteration, only the version already verified by the tester. This reduces noise and respects rule 0.2 ("when something is working, it works 100%").

**"PASS" in this diagram means Done at 100%, verified by the tester with more than one method** (Drush/SQL, Playwright, log/regression review — see [agent-tester.md](../agents/agent-tester.md)). There's no path to PR with a partial result; if any acceptance criterion isn't met 100%, it's a FAIL and goes back to implementation, no exceptions.

## Rule: no task ends without a commit

Once the tester gives a PASS, the task **always** closes with commit + push + PR — a working branch with verified changes is never left unpushed. This holds even if the user didn't explicitly say "commit this": finishing a development task implies leaving it in a PR, unless the user explicitly said not to.

## Rule: branches + PR, never a direct commit to `develop`

This flow never commits directly to `develop`:

1. The agent that implemented the change (`drupal-developer`/`frontend`) commits its changes on a working branch (created from `develop`, with a descriptive name like `fix/whatever` or `feat/whatever`), in English (rules 0.1/0.5 of AGENTS.md).
2. The branch is pushed to `origin` (remote: `git@github.com:your-org/your-drupal-site.git`).
3. A PR is opened against `develop` with `gh pr create` — title and description in English, including what changed and a summary of the tester's verification.
4. The [PR reviewer](../agents/agent-pr-reviewer.md) runs `phpcs` (Drupal/DrupalPractice standards) on PHP, `stylelint` on CSS and ESLint on JS — Coder/`phpcs` no longer covers CSS or JS (https://www.drupal.org/project/coder) — plus a reasoned review, and comments on the PR.
5. If there are blocking findings: back to the agent that implemented it, a new commit on the same branch (the PR updates automatically), re-tested if the fix could affect behavior, and back to the reviewer.
6. When the reviewer has no blocking findings: the orchestrator tells the user the PR is ready, with the link.
7. **The user does the merge.** No agent ever executes `gh pr merge` or commits directly to `develop` — this is the one step in this flow strictly reserved for a manual human action outside the agents' control.

This applies, for this project, a reviewed version of rule 0.5 of AGENTS.md ("commit to the develop branch"): the commit still lands on `develop`, but only after going through a PR + reviewer, before a human merge.

## Triggers

- A new plan in `docs/plans/` (whatever planning pattern your project uses).
- A finding from another agent (e.g. the [SEO agent](../agents/agent-seo.md) requests a missing structured-data field → a task for [Drupal developer](../agents/agent-drupal-developer.md)).
- A direct request from the user.

## Key difference from the content flow

This flow has **one human gate at the end** (the PR merge), but the correction loop with the tester and the PR reviewer can iterate freely without human intervention — the user only sees the already-verified, already-reviewed version, ready to merge.
