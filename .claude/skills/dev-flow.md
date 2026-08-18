---
name: dev-flow
description: Orchestrates the [Your Project] harness's development+testing flow (development/frontend → testing loop → commit+push+PR → PR review → human merge). Use it when the user asks to implement and verify a code change until it works correctly, or invokes /dev-flow.
---

Orchestrates the flow documented in `harness/flows/dev-testing-flow.md`.
This flow always ends in a reviewed pull request, never a direct commit to
`develop`.

1. Confirm the task has explicit user approval to implement (rule 0.3 of
   `AGENTS.md`) — if it doesn't, stop and ask before invoking
   `drupal-developer`/`frontend`.
2. Depending on the nature of the change, invoke `drupal-developer`
   (backend: content types, fields, config, hooks) and/or `frontend`
   (theme templates/CSS/JS), on a feature branch created from `develop`
   (descriptive name: `fix/...` or `feat/...`).
3. Invoke `tester` on the result.
4. If `tester` reports a fail: go back to the agent that implemented it
   with the concrete failure evidence, and repeat 2-3. This loop can
   iterate freely without human intervention — the user shouldn't see a
   failed iteration.
5. **If `tester` reports a pass: never leave the task without a commit.**
   Commit the changes on the feature branch (message in English, rule
   0.5), push to `origin` (`your-org/your-drupal-site`), and open a PR
   against `develop` with `gh pr create` (title/description in English,
   including a summary of the tester's verification).
6. Invoke `pr-reviewer` on the newly opened PR (runs `phpcs`/`stylelint` +
   reasoned review, comments on the PR).
7. If `pr-reviewer` reports blocking findings: go back to the agent that
   implemented it (new commit on the same branch — the PR updates
   itself), re-run `tester` if the fix could have affected behavior, and
   go back to `pr-reviewer`.
8. If `pr-reviewer` has no blocking findings: **don't ask the user for
   approval yourself, and NEVER run `gh pr merge`.** Return the PR link +
   the tester/reviewer summary to the `orchestrator` (skill
   `/orchestrator`) — it's the orchestrator that tells the user the PR is
   ready. If this flow was invoked directly (without going through
   `/orchestrator` first), act yourself under the rules in
   `harness/agents/agent-orchestrator.md` at this step — the user should
   only receive that notice from the orchestrator role, never from the raw
   flow. **The user does the merge manually, never an agent.**
9. Invoke `documenter` to record what happened in `harness/memory/`
   (including the PR number/link).

This flow never talks to the user directly — it always reports back to the
orchestrator (real or assumed, see step 8).
