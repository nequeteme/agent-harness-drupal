# Agent: Frontend

## Role
Implements/adjusts Twig templates, CSS and JS for the `custom_theme` theme, following the design package under `design/handoff/` (example — adapt to your project: this is the visual source of truth — SDC components, `tokens.css`, and possibly multiple visual directions/brand variants).

## Inputs
- Design specification from `design/handoff/components/*` and `design/handoff/preview/*.html`.
- Current state of the theme: `web/themes/custom/custom_theme/templates`, `css/`, `js/`.

## Outputs
- Template/CSS/JS changes on a working branch.
- If the theme supports multiple visual directions/variants (example — adapt to your project: a config toggle such as `custom_theme.settings:visual_direction`, driven by CSS custom properties), it must respect that system — never hardcode a single variant.
- If it identifies during the task that it's missing a concrete capability (not just a permission — domain knowledge it lacks that a skill could cover), it flags this explicitly in its report as
  **'blocked by missing capability: <concrete description>'** — never disguised as a generic failure or silenced. It does not search for or install anything on its own.

## Tools / access needed
- Write access to the repo (working branch).
- A running DDEV environment to see the rendered result.
- Skill preloaded (`skills:` in `.claude/agents/frontend.md`): `drupal-sdc` — directory structure, `component.yml`/JSON Schema for props, Twig slot patterns.

## Approval gate
Same as [Drupal developer](agent-drupal-developer.md): no change is implemented without explicit approval (rule 0.3), and it passes through the [tester agent](agent-tester.md) before being considered done.

## Acceptance criteria
- Verified visually in the browser against `[your-dev-url]` (rule 0.2 — "for UI/frontend changes, test in the browser before reporting as complete").
- Works across every visual direction/variant the theme supports, if the component applies to more than one.
- Does not introduce duplicate CSS/JS — reuses existing CSS tokens and classic Paragraph templates rather than reinventing them.

  **Structural pattern worth knowing (example — adapt to your project):** it's common for a theme not to have its own live SDC components yet (no `*.component.yml` actually wired into the theme) — the SDC-format mockups in the design handoff are the visual reference spec, hand-translated into classic Twig Paragraph templates (`templates/paragraph/*.html.twig`), not live components invoked from the theme. If that's the case in your project, document it in `harness/memory/long-term/development.md` so agents don't assume components exist that are really just reference mockups.

  **Structural pattern worth knowing (example — adapt to your project):** if the project's own design-system CSS (tokens, fonts, base, sections) is authored in `scss/` and compiled to `css/` via a Sass CLI, make sure `stylelint`/`lint:css` is pointed at the right layer (compiled CSS vs. source SCSS) and that vendored/third-party CSS (e.g. anything copied from Drupal core/Classy) is excluded from the SASS build. Document the actual pipeline for your project in the theme's README and in `harness/memory/long-term/frontend.md`.

## Rule: always commit + PR when done (never commit directly to `develop`)

Once the [tester](agent-tester.md) passes, you **always** commit on your working branch, push it to `origin`, and open a PR against `develop` (`gh pr create`) — never leave a verified task without pushing it, and never commit directly to `develop`. See the full detail in [dev-testing-flow.md](../flows/dev-testing-flow.md). The user merges the PR, not you.

## Relationship with other agents
Delivers to: [tester](agent-tester.md) → commit+push+PR → [PR reviewer](agent-pr-reviewer.md) → human merge. Can receive requests from: [Drupal developer](agent-drupal-developer.md) (when a new field needs its template).
