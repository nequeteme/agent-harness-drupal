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
- Skills preloaded (`skills:` in `.claude/agents/frontend.md`): `drupal-sdc` — directory structure, `component.yml`/JSON Schema for props, Twig slot patterns — and `drupal-standards`.
- `drupal-standards` is the shared, cross-agent rulebook (`.claude/skills/drupal-standards/SKILL.md`): the global Drupal 11 rules G1–G21 (never hack core, Composer-managed contrib and patches, config-as-code, security — including the output-filtering and sanitizer rules that Twig work depends on — plus PHP/docblock/YAML standards) and the content-architecture criteria C1–C5. Every rule there carries its documented reason and a real drupal.org source URL. The frontend-only rules live below, in "Drupal-specific practices".

## Approval gate
Same as [Drupal developer](agent-drupal-developer.md): no change is implemented without explicit approval (rule 0.3), and it passes through the [tester agent](agent-tester.md) before being considered done.

## Acceptance criteria
- Verified visually in the browser against `[your-dev-url]` (rule 0.2 — "for UI/frontend changes, test in the browser before reporting as complete").
- Works across every visual direction/variant the theme supports, if the component applies to more than one.
- Does not introduce duplicate CSS/JS — reuses existing CSS tokens and classic Paragraph templates rather than reinventing them.

  **Structural pattern worth knowing (example — adapt to your project):** it's common for a theme not to have its own live SDC components yet (no `*.component.yml` actually wired into the theme) — the SDC-format mockups in the design handoff are the visual reference spec, hand-translated into classic Twig Paragraph templates (`templates/paragraph/*.html.twig`), not live components invoked from the theme. If that's the case in your project, document it in `harness/memory/long-term/development.md` so agents don't assume components exist that are really just reference mockups.

  **Structural pattern worth knowing (example — adapt to your project):** if the project's own design-system CSS (tokens, fonts, base, sections) is authored in `scss/` and compiled to `css/` via a Sass CLI, make sure `stylelint`/`lint:css` is pointed at the right layer (compiled CSS vs. source SCSS) and that vendored/third-party CSS (e.g. anything copied from Drupal core/Classy) is excluded from the SASS build. Document the actual pipeline for your project in the theme's README and in `harness/memory/long-term/frontend.md`.

## Drupal-specific practices (frontend)

Global rules (never hack core, Composer/patches, config-as-code, security, coding standards) live in the `drupal-standards` skill. What follows is frontend-only, and every rule carries its documented source.

### F1. Templates print markup; logic goes in preprocess
Keep `.html.twig` files to markup + simple `{% if %}` / `{% for %}`. Anything that requires computing, querying or transforming goes into a preprocess function in `.theme` / `.module`.
> "The point is to make the template files *just print markup*. All the magic that needs to go into generating that markup — the stuff that we do need a programming language to construct — should remain done in PHP and be moved to a more suitable place in the theme layer."

Source: https://www.drupal.org/docs/theming-drupal/twig-in-drupal/twig-best-practices-preprocess-functions-and-templates

### F2. Preprocess returns render arrays — never call `render()`/`theme()` there
In a preprocess function, build `['#theme' => 'table', ...]`; do not call `drupal_render()` or `theme()`. Twig renders automatically.
> "Twig renders everything automatically so there is no need to call `drupal_render()` or `theme()` within a preprocess function."
> "Return render arrays instead of calling `theme()` or `drupal_render()`"

Practical consequence documented on the same page: pre-rendering in preprocess destroys downstream customisation opportunities.
Source: https://www.drupal.org/docs/theming-drupal/twig-in-drupal/twig-best-practices-preprocess-functions-and-templates

### F3. Defer `t()` and `url()` to the template, not preprocess
Keep raw data raw for as long as possible; call `|t` and URL helpers in the Twig template.
> "To provide raw data to templates for as long as possible, theme developers should call filters such as t and utility functions such as url() from within Twig templates."
> "Defer calls to filters like t() to Twig templates; Try to avoid calling these in preprocess."

Also documented there: use the Twig `without` filter instead of `drupal_render_children()` to selectively hide elements.
Source: https://www.drupal.org/docs/theming-drupal/twig-in-drupal/twig-best-practices-preprocess-functions-and-templates

### F4. Twig autoescapes — do not defeat it with `|raw`
Everything printed with `{{ }}` is escaped automatically. `|raw` disables that and must not be used on anything user-influenced.
> "The Twig theme engine now auto escapes everything by default. That means, every string printed from a Twig template (anything between `{{ }}`) gets automatically sanitized if no filters are used."
> "This filter should be avoided whenever possible, particularly if you're outputting data that could be user-entered." (about `|raw`)

Related hard rule from the same security page: always wrap attributes with quotes — `class="{{ class }}"` is safe; `class={{ class }}` is not.
Sources: https://www.drupal.org/docs/security-in-drupal/sanitizing-output , https://www.drupal.org/docs/administering-a-drupal-site/security-in-drupal/writing-secure-code-for-drupal , https://www.drupal.org/docs/develop/theming-drupal/twig-in-drupal/filters-modifying-variables-in-twig-templates

**Correction to a common misconception:** Drupal's documented template-security mechanism is **autoescaping**, not a "Twig security sandbox". Twig's `SandboxExtension` is an upstream Twig feature; no drupal.org page presents it as a Drupal theming rule. Don't reason about template security in terms of a sandbox.

### F5. Always print the full `{{ attributes }}`
If you print individual attributes explicitly, still emit `{{ attributes }}` at the end of the tag.
> "If you choose to print out individual attributes within a HTML tag, you should still include the complete `{{ attributes }}` at the end, so that attributes added by modules are still also printed."

Core's own pattern: `<div class="{{ attributes.class }}"{{ attributes }}>`.
Source: https://project.pages.drupalcode.org/coding_standards/twig/coding/ (legacy: https://www.drupal.org/docs/develop/coding-standards/twig-coding-standards)

### F6. Twig formatting details
No spaces around the filter pipe (`{{ 'Original'|t }}`, not `{{ 'Original' | t }}`). Prefer the `spaceless` filter over `-` whitespace-control dashes. Never add whitespace controllers around classes. Multi-line comments under 80 chars/line. Template docblocks list variables **without** type information, and inline variable references are single-quoted (`'node.body'`).
> "Please do not put spaces on either side of the pipe."
> "The `spaceless` filter … is Drupal core's preferred method for controlling whitespace in blocks of code."
> "*We do not need to use the `-` dash modifier* very often. Code is usually easier to read when using the spaceless filter."
> "Never remove spaces or add a whitespace controllers around classes"
> "Variable definitions should not include any information about the type of variable (array, object, string)."

Source: https://project.pages.drupalcode.org/coding_standards/twig/coding/

### F7. Single Directory Components are the current core component pattern
New UI components should be built as SDC: one directory per component holding `NAME.component.yml` + `NAME.twig` (+ optional `NAME.css`, `NAME.js`), under `themes/custom/custom_theme/components/NAME/`. Include them as `{{ include('custom_theme:chip', {...}, with_context = false) }}`.
> "All files necessary to render the component are grouped together in a single directory (hence the name)."
> "Starting with Drupal 10.3, Single-Directory Components became part of Drupal Core's render system." (before 10.3 it required manually enabling the SDC module)
> "Grouping all necessary code into one directory makes finding and jumping between relevant files easier."
> "SDC will automatically look for a `my-component.css` and `my-component.js` and add to an automatically generated library if found." ← a real, concrete benefit: you don't hand-write a library entry for a component's own assets.

The `[machine-name]:[component-machine-name]` reference format is guaranteed stable: "This way of specifying the component… will not change, as it is considered an API."
Sources: https://www.drupal.org/docs/develop/theming-drupal/using-single-directory-components/about-single-directory-components , https://www.drupal.org/docs/develop/theming-drupal/using-single-directory-components/quickstart , https://www.drupal.org/docs/develop/theming-drupal/using-single-directory-components/api-for-single-directory-components

### F8. SDC props are JSON-Schema-typed; slots take arbitrary content
`props` are typed inputs (`null`, `boolean`, `object`, `array`, `number`, `string`), optionally constrained with `enum`. `slots` are for "unknown or nested structures" — renderable content passed in.
> "The top level *props* key is always an object that contains the *properties* which are mapped to variables in the component's template file."
> Component metadata "is described, and optionally validated, by the schema in `metadata.schema.json`."

Example of a constrained prop:
```yaml
color:
  type: string
  enum: ['primary', 'secondary']
```

Sources: https://www.drupal.org/docs/develop/theming-drupal/using-single-directory-components/quickstart and https://www.drupal.org/docs/develop/theming-drupal/using-single-directory-components/api-for-single-directory-components
**Gap:** the dedicated "props and slots" doc page 404s on both `www.drupal.org` and `new.drupal.org`. The props/slots distinction above comes from the quickstart + API pages, which do cover it.

### F9. CSS/JS go in `*.libraries.yml` and are attached — never hardcoded `<script>`/`<link>`
Declare assets in `custom_theme.libraries.yml` / `MYMODULE.libraries.yml`, then attach with `#attached['library']` in a render array or `{{ attach_library('custom_theme/mylib') }}` in Twig. Declare dependencies explicitly.
> assets "are still only loaded if you tell Drupal it should load them" — loading everything globally "is bad for front-end performance."
> "Inline JavaScript is highly discouraged."
> File-based assets let "JavaScript to be aggregated and cached on the client side" and support "a stronger Content Security Policy to protect against XSS and other vulnerabilities."
> Dependencies are mandatory, not optional: a library needing jQuery must declare `dependencies: - core/jquery` "since Drupal no longer loads jQuery on all pages by default." Same for `core/drupalSettings`.
> "Realize that just using hook_library_info_build() or hook_library_info_alter() to append a library will not automatically make the library appear in the page."

Source: https://www.drupal.org/docs/develop/creating-modules/adding-assets-css-js-to-a-drupal-module-via-librariesyml

### F10. CSS naming: Drupal prescribes a BEM-style convention
- component: `.component-name`
- variant: `.component-name--variant`
- sub-object: `.component-name__sub-object`
- state: `.is-`-prefixed classes (or pseudo-classes)
- JS hooks: `.js-`-prefixed, and **must not appear in stylesheets**

Full words, not abbreviations; dashes between words.
> "Class names should use full words rather than abbreviations" with "a dash between words."
> JavaScript hooks "should not appear in stylesheets."

Source: https://project.pages.drupalcode.org/coding_standards/css/architecture/ (legacy: https://www.drupal.org/docs/develop/standards/css/css-architecture-for-drupal-9)

### F11. CSS architecture: SMACSS categories, no ID selectors, `!important` almost never
Organise CSS into Base / Layout / Component / State / Theme. Components must be context-independent. No `#id` selectors for styling. `!important` is restricted.
> "Base rules consist of styling for HTML elements only, such as used in a CSS reset or Normalize.css."
> "Reusable, discrete UI elements; components should form the bulk of Drupal's CSS."
> "Avoid using the id selector in CSS." — IDs are only for JS hooks, document anchors, and label/`for` associations.
> "The `!important` flag should be used sparingly … restricted to themes"
> "CSS should define the appearance of an element anywhere and everywhere it appears." — hence avoid `.sidebar .component {}`, which "reduces predictability and portability".
> "CSS rules should be abstract and decoupled enough that you can build new components quickly from existing parts without having to recode patterns and problems you've already solved."

Source: https://project.pages.drupalcode.org/coding_standards/css/architecture/

### F12. CSS file organisation
CSS lives in a `css/` subdirectory. Themes: "Always separate Base, Layout, and Component styles into their own files." Modules: **"Modules should never have any base styles."** Module CSS is named `module_name.module.css` (functional), `module_name.theme.css` (aesthetic), `module_name.admin.css`, `module_name.admin.theme.css`. Template-specific CSS takes the template's name.
> "Modules should never have any base styles."
> "Always separate Base, Layout, and Component styles into their own files."
> "State rules, including media queries, should be included with the component to which they apply."
> "Module styles are always loaded before theme styles and the SMACSS groups cannot be interlaced between modules and themes." ← this is the load-order reason the separation matters.
> for template-specific CSS, "the CSS file should be named the same as the template file"

Source: https://project.pages.drupalcode.org/coding_standards/css/file-organization/

### F13. JavaScript standards
2-space indent, no tabs. Semicolons required (except after `for`/`function`/`if`/`switch`/`try`/`while` blocks; but required after an anonymous function assigned to a variable). `let`/`const` only, declared before use, ideally at the top of the function. All file code inside a closure. jQuery-object variables prefixed `$`. `lowerCamelCase` for multi-word names; `UpperCamelCase` for constructors; `ALL_UPPER` for constants. Function names should begin with the module/theme name.
**Hard prohibition:** **"Drupal JavaScript MUST NOT define global variables."**
**Base standard:** "Drupal uses 'eslint-config-airbnb' as ESLint shareable config. Therefore it's reasonable to use 'Airbnb JavaScript Style Guide' as Drupal JS coding standard."
Source: https://project.pages.drupalcode.org/coding_standards/javascript/coding/ (legacy: https://www.drupal.org/docs/develop/standards/javascript/javascript-coding-standards)

### F14. JavaScript best practices (security-relevant subset)
> "JavaScript code SHOULD NOT be embedded in the HTML where possible, as it adds significantly to page weight with no opportunity for mitigation by caching and compression."
> "All output to the browser that has been provided by a user SHOULD be escaped through `Drupal.checkPlain()` first."
> "All strings in JavaScript files SHOULD be wrapped in `Drupal.t()`"
> "`eval()` SHOULD NOT be used." … no `Function` constructor, no strings passed to `setTimeout()`/`setInterval()`.
> "The `with` statement MUST NOT be used, since it is not possible to use `with` with enabled strict mode."
> "Use `[]` instead of `new Array()`" / "Use `{}` instead of `new Object()`"

Source: https://project.pages.drupalcode.org/coding_standards/javascript/best-practice/
**Caution — that page is partly dated:** it still says "When adding new HTML elements to the DOM, you SHOULD NOT use `document.createElement()`" and to use the jQuery equivalent. That guidance is stale relative to Drupal 11's jQuery-reduction direction — treat it as legacy, don't propagate it. The page also does **not** cover `Drupal.behaviors`, `once()` or `drupalSettings`; there is no authoritative standards-page rule for those, so don't cite this URL for them.

## Rule: always commit + PR when done (never commit directly to `develop`)

Once the [tester](agent-tester.md) passes, you **always** commit on your working branch, push it to `origin`, and open a PR against `develop` (`gh pr create`) — never leave a verified task without pushing it, and never commit directly to `develop`. See the full detail in [dev-testing-flow.md](../flows/dev-testing-flow.md). The user merges the PR, not you.

## Relationship with other agents
Delivers to: [tester](agent-tester.md) → commit+push+PR → [PR reviewer](agent-pr-reviewer.md) → human merge. Can receive requests from: [Drupal developer](agent-drupal-developer.md) (when a new field needs its template).
