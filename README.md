# Agent harness (alpha)

A multi-agent architecture for Claude Code (orchestrator, planner,
documenter, 14 specialized execution agents, content and development flows,
short/long-term memory system) extracted from its first real use case: a
Drupal 11 site build. This repo is a **starter kit**, not a record of that
original project — all project-specific content (client name, domain,
repo, GitHub Projects board, memory entries) has been stripped or emptied
to templates, and everything has been renamed/translated to English.

It's an **alpha**: the mechanism (how agents, flows, memory, and skills fit
together) is real and battle-tested on one project; the "fill in your own
project" parts are still templates, not yet proven on a second project.

## Structure

- `harness/agents/` — natural-language spec for each agent (role, inputs,
  outputs, approval gates).
- `harness/flows/` — the two main flows (content, development+testing) that
  chain agents together.
- `harness/memory/` — short/long-term memory system (what worked, what
  didn't, architecture decisions, patterns to repeat, metrics). Starts
  empty — fill it in as you use the harness on your project.
- `harness/improvements/` — periodic research reports on how to improve the
  harness itself. Starts empty.
- `harness/research/` — general background research (harness engineering,
  AI+MCP patterns) that isn't tied to one project.
- `harness/project-analysis/` — where you document your own project's
  current state before planning work. Starts empty.
- `harness/history/` — real stories (bugs, reverted decisions, lessons
  learned) worth feeding into future content/case studies. Starts empty.
- `.claude/agents/` — the runtime implementation (Claude Code subagents) of
  each agent described in `harness/agents/`.
- `.claude/skills/` — main-session skills (`/orchestrator`, `/content-flow`,
  `/dev-flow`) plus the Drupal/frontend domain skills preloaded into the
  relevant subagents. `drupal-standards/` is authored here directly (not a
  symlink): the sourced Drupal 11 rulebook — global rules G1–G21 plus the
  content-architecture criteria C1–C5, each with its documented reason and a
  drupal.org URL — shared by `drupal-developer`, `frontend`, `tester`,
  `pr-reviewer` and `planner`.
- `.agents/skills/` — actual content of the skills installed from the
  external catalog (those `.claude/skills/<name>` entries are symlinks into
  here).
- `AGENTS.md` — base rules (language, honesty, prior approval). Section 0.4
  (test environment) ships as a placeholder — fill it in with your own
  project's real data, and never commit real credentials to a shared repo.

## What's still needed to be a fully generic template

See the "(example — adapt to your project)" notes scattered through
`harness/agents/` and `harness/flows/` — a few patterns (nested Paragraphs
as a content model, classic Twig templates translated from SDC mockups, a
CSS-custom-property theme toggle) are kept as illustrative examples from the
original project, not as prescriptions. Infrastructure references (repo
name, theme machine name, GitHub Projects board) use bracketed placeholders
or generic example values — replace them with your own project's real
values as you adopt this harness.
