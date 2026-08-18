# Harness memory

Short-term and long-term memory system, maintained by the
[documenter agent](../agents/agent-documenter.md). It's knowledge of the
**project/platform**, not of Claude Code as a tool — it lives in the repo so
that any agent (or any future session) can read it, unlike Claude Code's own
memory (which belongs to the user, not the project).

## How it's organized

- **[short-term.md](short-term.md)** — rolling log of what was done in the
  most recent tasks/sessions and what was left pending. It doesn't grow
  forever: the documenter "graduates" what's stable into long-term memory
  and trims what's obsolete.
- **[long-term/](long-term/)** — knowledge consolidated by topic, one file
  per area (`content.md`, `development.md`, `architecture-decisions.md`,
  etc.). This is what a new agent should read before starting a task in that
  area.

## How each agent should use it

Before starting a task, an agent should check `long-term/<its-area>.md` if
it exists (avoids repeating decisions already made or mistakes already
known). When finished, it reports to the
[orchestrator](../agents/agent-orchestrator.md), which asks the
[documenter](../agents/agent-documenter.md) to record what's relevant. No
execution agent writes memory directly — that's the documenter's job, to
keep a single, consistent judgment of what's worth keeping.
