# Harness architecture decisions

Consolidated knowledge — the *why* behind decisions, not the *what* (that
lives in [harness/implementation-options.md](../../implementation-options.md)
and [harness/architecture-maps.md](../../architecture-maps.md)).

This file records durable, project-wide decisions about how the harness
itself is built and run — which implementation option was chosen and why,
permanent operating rules the orchestrator/planner/tester must follow,
infrastructure that got wired up (repos, CI tooling, project boards), and
any reversal of an earlier decision. It is written by the
[documenter](../../agents/agent-documenter.md) whenever a decision is
explicit, user-requested, or stable enough to stop living only in
[short-term.md](../short-term.md).
