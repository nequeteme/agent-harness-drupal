# Harness user guide (Option D — implemented)

This document explains how to actually use everything implemented as part of Option D (see [implementation-options.md](implementation-options.md) and [architecture-maps.md](architecture-maps.md)). It's the operational guide; the design documents (`agents/`, `flows/`, `research/`) remain the underlying reference.

> This guide describes a worked example from a prior project's implementation. The concrete details (module names, file paths, board names) are illustrative — adapt them to your own project's structure and naming.

## 1. What gets implemented

| Piece | Where it lives | What it does |
|---|---|---|
| An editorial workflow | `config/sync/workflows.workflow.editorial.yml` (example) | States `draft` → `review` → `published`, applied to your content-bearing bundles (example: `landing_page`, `legal_page`). |
| `content_moderation` + `workflows` modules | Drupal core, enabled | The native engine behind the workflow above. |
| Execution subagents | `.claude/agents/*.md` | One per role requested (style, seo, content-writer, content-researcher, news-researcher, drupal-developer, frontend, tester), plus `pr-reviewer` (see §1.3). |
| Orchestration skills | `.claude/skills/content-flow.md` (`/content-flow`), `.claude/skills/dev-flow.md` (`/dev-flow`) | Chain the subagents in the right order, with human gates in the right place. |
| Existing content | Any nodes already published | Backfilled to `moderation_state = published` so the new workflow doesn't break anything already live (verify this with a real smoke test of your published routes after the change). |

All of this should end up committed on `develop` — check your `git log` to confirm.

## 1.1 Orchestrator, planner, documenter, and improvement researcher

Four coordination agents sit on top of the above — they don't replace anything, they're additive:

| Agent | Real type in Claude Code | What it does |
|---|---|---|
| `orchestrator` | Skill (`.claude/skills/orchestrator.md`, command `/orchestrator`) | Recommended entry point. Talks to the user, delegates planning, dispatches to flows/agents, consolidates reports. |
| `planner` | Subagent (`.claude/agents/planner.md`) | Turns a goal into a per-agent task plan. Invoked by the orchestrator. |
| `documenter` | Subagent (`.claude/agents/documenter.md`) | Records in `harness/memory/` (short- and long-term) what happened in each task/flow. |
| `harness-improvement-researcher` | Subagent (`.claude/agents/harness-improvement-researcher.md`) | Researches how to improve the harness itself; reports roughly every ~2 days in `harness/improvements/`. |

**Why the orchestrator is a skill and not a subagent**: in Claude Code a subagent can't invoke other subagents — only the main session can. The orchestrator needs to dispatch to several agents, so it has to govern the main session directly (the same way `/content-flow` and `/dev-flow` already do). See `harness/agents/agent-orchestrator.md`.

**How you use it in practice**: instead of remembering which skill or agent to invoke, you can simply talk to Claude Code normally ("I need a new FAQ about X", "how's the SEO backlog going?") and let it act as orchestrator — or invoke it explicitly with `/orchestrator` to force that mode. It decides whether planning is needed (`planner`), which flow/agent to run, and gives you a consolidated summary at the end, not a raw dump of every agent's report.

## 1.2 Quality rules: task sizing and Definition of Done

Two more rules, reflected in both the design and the runtime:

- **The planner never sizes a task above ~60% of the context window** of the agent that's going to execute it. If a goal is large, you'll see it split into several smaller sequential tasks instead of one giant task — this is deliberate, not a technical limit being measured live. Detail: `harness/agents/agent-planner.md`.
- **Definition of "Done" = 100%, verified by the `tester` with more than one method** (Drush/SQL, Playwright, log/regression review, happy path + edge case). There's no "partial pass": if a single acceptance criterion isn't met 100%, it's a complete fail and goes back to implementation, no exceptions. You'll see this reflected in the dev flow's reports — the `tester` cites evidence for every method it used, not just "should work". Detail: `harness/agents/agent-tester.md`.

## 1.3 Branches + reviewed PR, never a direct commit

Three more rules that shape the dev flow:

1. **No development task ends without a commit.** As soon as the `tester` passes, the agent that implemented the change always commits, pushes, and opens a PR — a verified branch is never left unpushed.
2. **The dev flow never commits directly to `develop`.** It's always: working branch → `tester` → commit + push + `gh pr create` against `develop` → **`pr-reviewer`** (`.claude/agents/pr-reviewer.md`) → if there are findings, back to development; if not, the orchestrator lets you know the PR is ready.
3. **You always do the merge, never an agent.** Neither `pr-reviewer` nor the orchestrator ever executes `gh pr merge` under any circumstance, even if the PR is perfect.

**Infrastructure this depends on** (all real, none simulated):
- A remote (e.g. `git@github.com:your-org/your-drupal-site.git`) configured as `origin`, with your working branches mirrored there.
- The `gh` CLI installed and authenticated.
- `phpcs` (with `Drupal`+`DrupalPractice` standards, via `drupal/coder`) installed as a Composer dev dependency, configured in your project's `phpcs.xml` (pointing at your custom modules/themes). Note that Coder covers **PHP only** — "JavaScript and CSS support have been removed from Coder" (https://www.drupal.org/project/coder), so CSS and JS need their own linters below.
- `stylelint` installed as an npm dev dependency of your theme, configured in the theme's `.stylelintrc.json` (`npm run lint:css`), and ESLint for JS — both with the official Drupal configuration files. If your project's design-system CSS is authored in SCSS and compiled, make sure the lint step targets the right layer — document your actual pipeline instead of assuming this guide's example matches it.

Full detail: `harness/flows/dev-testing-flow.md` and `harness/agents/agent-pr-reviewer.md`.

## 1.4 GitHub Projects board + history

Two more agents:

- **`project-writer`** (`.claude/agents/project-writer.md`): creates a card on your **[Your GitHub Project name]** board for each phase/task from the planner (label `develop` or `content`, `Status` starting at `Backlog`) and moves it (`Ready` → `In progress` → `In review` → `Done`) as the flow progresses — so you can see real progress **from GitHub**, without asking Claude Code. It also works in reverse: if you create a card directly on the board, it gets detected and handed to the orchestrator as a new work request — the board is a second entry point into the harness. Set up a board with the fields described in `agent-project-writer.md` (`Backlog`/`Ready`/`In progress`/`In review`/`Done`, `Priority`, `Size`), and create the `develop`/`content` labels (or whatever labels match your flows) on the repo.
- **`historian`** (`.claude/agents/historian.md`): records in `harness/history/` — not `harness/memory/` — real stories (bugs with an interesting cause, reverted decisions, learnings), as input for a future "how we built this" blog post from `content-writer`. It doesn't record everything, only what's worth it (see the criteria in `agent-historian.md`).

**Infrastructure this depends on**: `gh` needs the `project` scope (read/write access to GitHub Projects), authorized via device flow.

Full detail: `harness/agents/agent-project-writer.md`, `harness/agents/agent-historian.md`, `harness/flows/orchestration-flow.md`.

## 2. How to trigger a full flow

The flows are **skills** — invoked as a slash command, describing the task in natural language:

```
/content-flow write a new FAQ answer about accessibility in institutional
portals, based on recent research
```

```
/dev-flow add the missing structured-data field to the service card
component, it's one of the pending SEO items
```

You can also just describe the task without the slash and Claude Code will pick the right flow if the description matches (for example, "I want new content for the services section, based on what's happening in our sector" will trigger `/content-flow` on its own).

**What you'll see while it runs**: each step of the flow invokes a different subagent (you'll see its name in the output — e.g. "content-researcher", then "content-writer", then "style-reviewer", then "seo"). At the end of the content flow, you'll be **asked for explicit approval** before anything transitions to `published` — without your "yes, publish it", it stays in `draft`/`review`. At the end of the dev flow, you'll see the `tester`'s report, then the `pr-reviewer`'s (with the PR already opened and commented), and you'll be told when it's ready for **you** to do the merge — no agent ever commits to `develop` or merges the PR (see §1.3).

**Who asks you that question**: always the orchestrator, never the raw flow — even if you invoked `/content-flow` or `/dev-flow` directly without going through `/orchestrator` first. This is a deliberate rule: all communication with you is centralized in the orchestrator, so the same judgment always decides how to present something and when to ask for approval (see "Centralized-communication rule" in `harness/agents/agent-orchestrator.md`).

## 3. How to invoke a single agent (without running the full flow)

Useful when you don't need the whole pipeline — for example, just a one-off SEO audit:

```
use the seo agent to audit the homepage metadata
```

```
have the tester agent verify the change I just requested on the mobile menu
```

Claude Code picks the right subagent from its `description` (see each file in `.claude/agents/`), but naming it explicitly ("use the X agent") is the most reliable way to force which one gets used.

## 4. How the moderation state works (`draft` → `review` → `published`)

Once enabled, any moderated content type has a **Moderation state** field on its edit form (`/node/{id}/edit` in the Drupal admin). Content agents leave their drafts in `draft` or `review` — they never publish directly.

**As a human, to review and publish a draft you have two paths:**

1. **From the Drupal UI**: go to `/admin/content` (or `/admin/content/moderated` to see only what's pending review), open the node, change the moderation-state dropdown to `Published`, and save.
2. **Ask Claude Code**: "approve and publish the draft for node X" — but only do this once you've actually reviewed it; the agent won't force you to look at it first.

Valid transitions are: `draft → review`, `review → draft` (request changes), `review → published`, `draft → published` (via "create new draft" followed by publishing directly, for urgent cases), and back to `draft` from any state to resume editing.

## 4.1 Project memory (`harness/memory/`)

The `documenter` maintains two levels:

- `harness/memory/short-term.md` — what's happened in recent tasks, what's still pending. Updated after every flow/task.
- `harness/memory/long-term/*.md` — consolidated knowledge by topic (the documenter creates files here as needed: `content.md`, `seo.md`, `architecture-decisions.md`, etc.).

You don't need to explicitly ask for it to update — the orchestrator invokes the documenter at the close of every flow. If you want to check the state without triggering a new task, you can ask directly "what does short-term memory say?" or just read the files yourself — they're plain Markdown.

## 4.2 Harness improvement report (roughly every ~2 days)

**Default: manual trigger, not scheduled.** The `harness-improvement-researcher` is designed for a ~2-day cadence, but by default it only runs when you explicitly ask for it — there's no `/loop` or cron routine configured out of the box. Invoke it with something like "have the harness improvement researcher generate its report now".

If you want it automatic later, both options are documented in `harness/agents/agent-harness-improvement-researcher.md` (`/loop`, tied to an active session; or a real cron routine via the `schedule` skill, which does run without an open session but is a recurring automation with ongoing consumption — set this up deliberately when you want to take that step).

## 5. Recommendations for getting the most out of it

- **Don't run `/content-flow` on autopilot.** Generate content when there's a real reason (a detected SEO gap, relevant news, a concrete request), not just to "have activity". The `news-researcher` agent is instructed to filter noise, but the final call on generating content is yours.
- **Use `/loop` for periodic news monitoring** if you want `news-researcher` to run on a regular cadence inside an active session (for example, `/loop 1d news-researcher: check for sector news`). Remember this only runs while the session exists — it's not 24/7 unattended autonomy (that's the difference with Option B, see [implementation-options.md](implementation-options.md)).
- **Keep `harness/agents/*.md` as the source of truth** for each role. If you change how you want an agent to work, edit the corresponding `.md` in `harness/agents/` first, then reflect the change in `.claude/agents/*.md` (the actual subagent) — that keeps both in sync.
- **The human gate is not negotiable for publishing/merging** — it's there by design (rule 0.3 of `AGENTS.md`). Don't try to skip it by asking an agent to "just publish it" or "just merge the PR" — no agent will do it, no matter how you phrase the request; it's a hard rule.
- **If a `tester` report says "fail", don't ask it to round up to a pass** — it's 100% or it's not done (see §1.2). A well-reported fail with evidence from several methods is more useful than an optimistic pass.
- **If a task looks huge when you ask for it, expect the planner to split it** (the 60% rule, §1.2) — it's not that the harness is slow, it prefers several small, well-verified tasks over one large, half-verified one.
- **For new content that doesn't fit your existing Paragraph/content types** (e.g. if you eventually want a blog/insights section), that's a `drupal-developer` task first (create the content type/Paragraph type), not something `content-writer` should improvise.

## 6. Quick sanity check

```bash
ddev drush php:eval "print_r(\Drupal::entityTypeManager()->getStorage('workflow')->load('editorial')->getTypeSettings());"
```

Should show your three states (`draft`, `review`, `published`) and the bundles they apply to. To see content pending review: `/admin/content/moderated` in the browser, against your dev environment (`[your-dev-url]`; bring up your tunnel/local environment first if it doesn't respond).

## 7. What this implementation does NOT do (real limits)

- Nobody outside a Claude Code session can trigger an agent — an editor without Claude Code access can **review and publish** drafts (step 4 above) but can't generate new content on their own. That leap is Option B, documented as an evolution path.
- No JSON:API/REST is required — all automation can stay Drush-scripting-based inside DDEV, if that's the pattern your project already uses.
- No Drupal AI module (`ai`, `ai_agents`, `mcp_server`) needs to be installed — the agents live entirely in `.claude/`, not inside Drupal.
- PR merges stay 100% manual — unless you configure branch protection/CI on GitHub, nothing technically stops someone from merging a PR without `pr-reviewer` weighing in; the control is process (a harness rule), not platform enforcement. If you want a hard control at the GitHub level, that's branch-protection configuration — not part of this implementation by default.

## 8. File map of this implementation

```
your-drupal-site/
├── .claude/
│   ├── agents/            ← subagents: the execution agents + pr-reviewer +
│   │                          planner, documenter,
│   │                          harness-improvement-researcher,
│   │                          project-writer, historian
│   └── skills/             ← orchestration skills: content-flow, dev-flow,
│                              orchestrator (commands /content-flow, /dev-flow, /orchestrator)
├── harness/
│   ├── README.md           ← general index
│   ├── USER-GUIDE.md       ← this document
│   ├── implementation-options.md
│   ├── architecture-maps.md
│   ├── agents/              ← design specification for each agent (source of truth)
│   ├── flows/                ← design specification for each flow (includes orchestration-flow.md)
│   ├── memory/                ← short-term.md + long-term/ (project-tracker.md,
│   │                             roadmap.md, architecture-decisions.md, ...)
│   ├── history/                ← real narrative, maintained by the historian
│   ├── improvements/           ← reports from the harness-improvement-researcher
│   ├── research/              ← background research
│   └── project-analysis/     ← snapshot of the project at design time
└── web/ (or your Drupal docroot) ← the actual Drupal 11 codebase, in its own git remote
    ├── phpcs.xml             ← Drupal/DrupalPractice standards (pr-reviewer)
    └── themes/custom/custom_theme/
        └── package.json + .stylelintrc.json  ← CSS linting (pr-reviewer)
```

**Live board**: set this to your own GitHub Projects URL once you create it (**[Your GitHub Project name]**).
