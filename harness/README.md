# Harness engineering

A complete map of the research, analysis, and design behind an AI agent harness for a Drupal 11 project.

> **This is a starter kit.** It was extracted from a real Drupal 11 project where an "Option D" hybrid harness was implemented end to end: `content_moderation`/`workflows` enabled with an editorial workflow (draft/review/published), a set of subagents in `.claude/agents/`, and orchestration skills (`/orchestrator`, `/content-flow`, `/dev-flow`) in `.claude/skills/`. The dev flow always ends in a reviewed Pull Request (`gh`, `phpcs`, `stylelint`), never a direct commit — the user always does the merge. Each task can also live as a card on a GitHub Projects board (maintained by `project-writer`), and cards the user creates there by hand feed back into the harness on their own. See the **[USER-GUIDE.md](USER-GUIDE.md)** for how to use it. The rest of this folder is background design/research documentation — adapt all of it to your own project rather than treating it as fixed.

## How to navigate this folder

1. **[research/](research/)** — what a "harness" is and what already exists in the ecosystem.
   - [harness-engineering.md](research/harness-engineering.md) — the concept, the standard components of a harness, and prior art worth knowing about.
   - [drupal-ai-mcp.md](research/drupal-ai-mcp.md) — Drupal.org AI/MCP modules, Drupal's official AI initiative, the "Outside AI" architecture pattern.

2. **[project-analysis/current-state.md](project-analysis/current-state.md)** — a real snapshot of a project at design time: what exists (content architecture, SEO, modules, theme), what doesn't exist yet (content moderation, external API, AI), and what already-proven patterns (Drush, Playwright) the harness builds on. Treat this as a worked example, not a template to copy verbatim — write your own snapshot for your project.

3. **[agents/](agents/)** — one document per agent, with role, inputs, outputs, required tools, approval gate, and relationship to the others. The execution agents:
   - [agent-style-reviewer.md](agents/agent-style-reviewer.md)
   - [agent-seo.md](agents/agent-seo.md)
   - [agent-content-writer.md](agents/agent-content-writer.md)
   - [agent-content-researcher.md](agents/agent-content-researcher.md)
   - [agent-news-researcher.md](agents/agent-news-researcher.md)
   - [agent-drupal-developer.md](agents/agent-drupal-developer.md)
   - [agent-frontend.md](agents/agent-frontend.md)
   - [agent-tester.md](agents/agent-tester.md)
   - [agent-pr-reviewer.md](agents/agent-pr-reviewer.md) — reviews every PR (phpcs/stylelint + best practices), comments on findings, never merges.

   Plus the coordination/meta agents:
   - [agent-orchestrator.md](agents/agent-orchestrator.md) — the single entry point, "product manager" for the rest.
   - [agent-planner.md](agents/agent-planner.md) — turns a goal into a per-agent task plan.
   - [agent-documenter.md](agents/agent-documenter.md) — short/long-term memory, keeps platform knowledge current.
   - [agent-harness-improvement-researcher.md](agents/agent-harness-improvement-researcher.md) — researches how to improve the harness itself, roughly every ~2 days.
   - [agent-project-writer.md](agents/agent-project-writer.md) — creates and syncs cards on the GitHub Projects board; detects user-created cards and hands them to the orchestrator.
   - [agent-historian.md](agents/agent-historian.md) — records real learnings/failures in [history/](history/), input for future blog content.

4. **[flows/](flows/)** — how the agents connect to each other:
   - [orchestration-flow.md](flows/orchestration-flow.md) — the meta-flow: user/board ↔ orchestrator ↔ planner → project-writer → execution → documenter + historian.
   - [content-flow.md](flows/content-flow.md) — research → writing → style → SEO → human review → publication → documenter.
   - [dev-testing-flow.md](flows/dev-testing-flow.md) — development/frontend → testing (loop) → commit+push+PR → PR reviewer (loop) → human merge → documenter.

5. **[memory/](memory/)** — the short- and long-term memory the documenter maintains, available to any agent or future session (includes `project-tracker.md` and `roadmap.md`, from project-writer).

6. **[history/](history/)** — narrative (not operational state) of real learnings/failures, maintained by the historian, meant as input for the blog that `content-writer`/`content-researcher` will eventually produce.

7. **[implementation-options.md](implementation-options.md)** — the final deliverable: concrete ways to build this in a project (A. Claude Code subagents, B. native Drupal AI modules, C. an external harness with MCP, D. the recommended hybrid), with scope, pros, cons, and effort for each.

8. **[architecture-maps.md](architecture-maps.md)** — Mermaid component diagrams for options A, B, D, and expanded D (with orchestrator/planner/documenter), with a comparison table of where each agent lives, what triggers it, and where the human gate sits.

9. **[USER-GUIDE.md](USER-GUIDE.md)** — how to actually use what's implemented: how to trigger the flows, how to invoke a standalone agent, how the moderation state works, and the real limits of this implementation.

## Cross-cutting principle

Every agent and flow designed here inherits the rules from your project's `AGENTS.md`: honesty (never claim "it works" without verifying, nothing invented), explicit human approval before touching code or publishing content, and real verification against your dev environment before calling anything done. The harness doesn't replace those rules — it automates work within them.
