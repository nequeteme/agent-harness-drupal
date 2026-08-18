# Project analysis (for harness design)

This document captures a one-time snapshot of the target project — done once
at harness setup time, before the agents/flows are designed, so that every
subsequent decision (which agents to create, which flows to wire up, what
memory to seed) is grounded in the real state of the codebase rather than
assumptions. It should be filled in early in a new project, typically by
whoever bootstraps the harness (orchestrator/planner), and re-read (not
re-written) by any agent that needs project context before its first task.

Sections to fill in:

## 1. What the site/product is

What it is, who it's for, what kind of site it is (landing page, blog,
portal, application), what stack it's built on, and what languages/locales
it serves.

## 2. Status of existing implementation plans

Table or list of any pre-existing implementation plans/phases and their
completion status, with known pending items per phase.

## 3. Content architecture

The real data model as confirmed in the codebase/config — content types,
their fields, translatability, and any structured-content pattern in use
(e.g. nested paragraph/component references).

## 4. SEO/GEO/AEO — what already exists

Metadata, structured data, sitemap, and AI-bot-facing configuration already
in place.

## 5. Installed modules/dependencies

What's installed vs. available-but-disabled, and anything notably absent
(AI, MCP, moderation, workflows, etc.).

## 6. Automation already proven

Any automation pattern already used successfully in this project (e.g.
scripted changes via the CMS's own APIs) that an agent can reuse instead of
inventing a new integration path.

## 7. Testing already proven

Any testing/verification tooling already installed and used (e.g. browser
automation) that should be the basis for a tester agent.

## 8. Theme and frontend

Theme/frontend structure, key directories, and where the visual
source-of-truth (design handoff, style guide) lives.

## 9. Search for existing AI/MCP/harness integration

Result of checking whether the project already has any AI/agent/MCP
integration, so the harness isn't duplicating or conflicting with something
that already exists.

## 10. Process rules every harness agent must respect

Summary, oriented at automated agents, of the project's own process rules
(from its own AGENTS.md/CONTRIBUTING or equivalent) — documentation/code
language conventions, verification requirements, approval gates, test
environment, commit policy.
