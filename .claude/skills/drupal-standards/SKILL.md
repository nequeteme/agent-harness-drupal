---
name: drupal-standards
description: Mandatory and recommended Drupal 11 practices sourced from official drupal.org documentation — never hack core, Composer-managed contrib/patches, configuration management, security (SQL placeholders, filter-on-output, sanitizers, CSRF, route access), coding/documentation/YAML standards, and content-architecture criteria (Content Types vs Paragraphs vs Blocks vs Media). Load it before writing or reviewing any Drupal code, config, or content model.
---

# Drupal 11 Standards — Global Rules and Content Architecture

Every rule below carries (a) the rule, (b) the *actual documented* reason, usually
verbatim, and (c) a real source URL. Where no clear official source was found, the item
is explicitly labelled — do not present those parts as documented Drupal policy.

## 0. Read this first: where the coding standards actually live now

The old canonical URLs under `drupal.org/docs/develop/standards/*` are now **marked
"[Obsolete]"** on drupal.org itself. They redirect readers to a GitLab-Pages site which is
now the canonical home of the standards.

- Old index (now flagged obsolete): https://www.drupal.org/docs/develop/standards
- **Current canonical standards:** https://project.pages.drupalcode.org/coding_standards/

Authority statement, verbatim from the canonical site:
> "The Drupal coding standards apply to code within Drupal and its contributed modules.
> These standards are version-independent and 'always-current.'" … "all new code should
> follow the current standards, regardless of (core) version"
> — https://project.pages.drupalcode.org/coding_standards/

**Implication:** cite `project.pages.drupalcode.org/coding_standards/...`, never the
`drupal.org/docs/develop/standards/...` equivalents — Drupal itself labels those obsolete.

Also note: a large part of drupal.org docs now 302-redirects to `new.drupal.org`, and
several `new.drupal.org` paths 404. If you fetch Drupal docs at runtime, expect redirects
and fall back to search rather than reporting a dead link as "the doc doesn't exist".

---

## 1. Global rules — apply to any work that touches Drupal code

### G1. Never modify Drupal core files. Ever.
**Rule:** Do not edit anything under `core/`. Core is off-limits for direct modification,
including "small" fixes.
**Why (documented reasons, three of them):**
> "You will make it complicated, difficult, or nearly impossible to apply site updates such as Security and bug fixes."
> "You will make it difficult for those that come after to maintain the site."
> "You could possibly leave your site vulnerable to exploits."
> "The Drupal core has been designed to be modular, so there should be no reason to hack it."

Asked whether there are exceptions, the page answers: **"Nope"**, adding that if you have
to ask, "the answer should almost always be 'no.'"
**Source:** https://www.drupal.org/node/144376 (also surfaced as
https://www.drupal.org/docs/7/site-building-best-practices/never-hack-core — the page is
D7-era but is still the canonical "never hack core" statement on drupal.org)

### G2. The documented alternatives to hacking core (in order of preference)
**Rule:** To change core behaviour, use — in this order — (1) the extension APIs core
exposes (hooks, plugins, event subscribers, service decoration/overriding), (2) a
contributed module, (3) as a genuine last resort, a **versioned patch applied by
Composer**. Never an in-place edit.
**Why:** Core is explicitly designed as a modular/pluggable system; the Plugin API exists
precisely so functionality is swappable without touching core:
> "Plugins are small pieces of functionality that are swappable."
> — https://www.drupal.org/docs/drupal-apis/plugin-api/plugin-api-overview

And the official escalation path for a genuinely-needed core change is to file an issue and
contribute the patch upstream: "If there is a feature you want and it cannot be
accomplished outside of modifying core, consider submitting your hack as a patch." —
https://www.drupal.org/node/144376

### G3. If a patch is truly unavoidable, apply it via `cweagans/composer-patches` and commit the patch file
**Rule:** Patches to core or contrib go in `composer.json` under `extra.patches`, using
`cweagans/composer-patches`. The `.patch` file itself must be **downloaded and committed to
the repo**, not hotlinked to drupal.org.
**Why (verbatim warning in the official docs):**
> "Do not hotlink patches from Drupal.org! Download them and commit them to your local
> repository to prevent build failures if the remote file changes or is deleted."

Also documented there: for the recommended webroot layout, core patches need `-p2` patch
level, because "the project composer.json is a level above" the webroot.
**Source:** https://www.drupal.org/docs/develop/using-composer/manage-dependencies

### G4. Same rule applies to contrib: manage modules/themes only through Composer
**Rule:** Contributed modules and themes are installed and updated exclusively via
Composer; never hand-edit files inside a contrib module, and always commit
`composer.lock`.
**Why:** The docs state Composer is the supported and exclusive filesystem manager for
modules/themes:
> "The standard way to install and update Drupal sites is using **Composer**."

with the checklist item "Always commit the updated `composer.lock` file to Git", plus
`drush updatedb` and `drush cache:rebuild` afterwards. Legacy Drush package-management
commands are explicitly deprecated.
**Source:** https://www.drupal.org/docs/updating-drupal/updating-modules-and-themes-using-composer
**Note (honesty):** this page does **not** contain an explicit "never edit contrib code"
sentence. The prohibition is the same mechanical consequence as G1 (the next
`composer update` overwrites your edit), and the composer-patches doc (G3) is the
documented mechanism for changing contrib. Treat "never edit contrib in place" as
*strongly implied* by the Composer-managed workflow, not as a verbatim quote.

### G5. Custom code lives in `modules/custom/` and `themes/custom/`, never in `core/`
**Rule:** Custom modules go in `/modules/custom/<machine_name>`; core is isolated in
`/core`.
**Why (documented):**
> "All core modules and libraries files are now located in the `/core` directory."

…which is exactly what freed `/modules` for contributed and custom code.
Machine-name rules (documented, and enforced by Drupal): must start with a letter; only
lowercase letters, digits and underscores; max 50 chars; unique across
modules/themes/profiles; and cannot be one of the reserved terms `src`, `lib`, `vendor`,
`assets`, `css`, `files`, `images`, `js`, `misc`, `templates`, `includes`, `fixtures`,
`Drupal`.
> "Be sure to not use upper-case letters in your module's machine name as Drupal will not
> recognize your hook implementations."

**Source:** https://www.drupal.org/docs/develop/creating-modules/naming-and-placing-your-drupal-module

### G6. Site configuration is code: export it to YAML and version it; do not "configure production"
**Rule:** Configuration changes are made in a dev environment, exported with
`drush config:export` to the sync directory, committed to git, and imported
(`drush config:import`) on the way up through environments. Never make config changes
directly on the live site.
**Why (verbatim):**
> "Making configuration changes on a live site is not recommended. The system is designed
> to make it easy to take the live configuration, test changes locally, export them to
> files, and deploy to production."
> "Exporting and importing configuration changes between a Drupal installation in
> different environments, such as Development, Staging, and Production, allows you to make
> and verify your changes with a comfortable distance from your site's live environment."

Also documented: the *active* configuration lives in the database (`config` table); the
YAML files are the versionable representation of it.
**Source:** https://www.drupal.org/docs/administering-a-drupal-site/configuration-management/managing-your-sites-configuration

### G7. Do not try to change a live site's config by editing `config/install` YAML
**Rule:** Editing a module's `config/install/*.yml` does not change an existing site. To
change installed config you need config import, or an update hook.
**Why (verbatim):**
> "Don't try to change the active configuration on your site by changing files in a
> module's config/install directory. This will NOT work, because Drupal only reads from
> that directory when the module is installed."

**Source:** https://www.drupal.org/docs/administering-a-drupal-site/configuration-management/managing-your-sites-configuration

### G8. Take a database dump before every config import
**Rule:** Snapshot the DB before `config:import` / config synchronization.
**Why (verbatim):**
> "it's strongly recommended that you do a database-dump before each synchronization of
> the staging and the active directory, as the database-dump could save your life on a
> potentially needed rollback-strategy."

Related requirement: "In order for config import to work, the exported config files must
contain a matching Site UUID."
**Source:** https://www.drupal.org/docs/administering-a-drupal-site/configuration-management/managing-your-sites-configuration

### G9. Environment-specific config differences → Config Split, not manual divergence
**Rule:** When dev needs modules/settings that prod must not have (devel, debug,
experimental views), use the Config Split contrib module rather than letting the
environments' config drift or hand-editing YAML per environment.
**Why (from the project page):**
> "sometimes developers like to opt out of the robustness of CM and have a super-set of
> configuration active on their development machine and deploy only a subset."

And crucially the module preserves CM's guarantees:
> "Importing configuration still removes configuration not present in the files. Thus, the
> robustness and predictability of the configuration management remains."

**Source:** https://www.drupal.org/project/config_split
**Caveat:** Config Split is a **contributed module**, not core policy. The *policy* (config
in YAML under version control) is core-documented (G6); the split pattern is the
community-standard solution to the environment-difference problem, documented on the
module's own project page.

### G10. Never concatenate data into SQL — use Database API placeholders
**Rule:** All user-influenced values go into queries as named placeholders (`:name`),
passed as the args array. Placeholders are **not** quoted in the SQL string. Table/column
names cannot be placeholders — run them through `escapeTable()`. LIKE values go through
`escapeLike()`. Never let user input choose a query operator; use a predefined operator
list.
**Why (verbatim):**
> "Data must **never** be concatenated directly into SQL queries" — https://www.drupal.org/docs/administering-a-drupal-site/security-in-drupal/writing-secure-code-for-drupal
> "placeholders should *not* be escaped or quoted regardless of their type. Because they
> are passed to the database server separately, the server is able to differentiate between
> the query string and the value on its own." — i.e. `WHERE type = :type`, **not**
> `WHERE type = ':type'`.
> "Due to the nature of static queries, it can be very easy for people to forget to
> sanitize their query for user inputs and open your application to sql-injection attacks."
> … "only very simple SELECT queries should use the static `query()` method."

**Sources:** https://www.drupal.org/docs/drupal-apis/database-api/static-queries and
https://www.drupal.org/docs/administering-a-drupal-site/security-in-drupal/writing-secure-code-for-drupal

### G11. Drupal filters on **output**, not on input — never "clean" data on the way in
**Rule:** Store user input as submitted; escape/filter at the moment of output,
appropriately for the output context.
**Why (verbatim):**
> "The type of filtering needed depends on the output context. Acting on input can be quite
> problematic because you do not know what characters are forbidden without knowing the
> context in which they will appear."
> "Encoding creates another problem, in that processing the escaped or encoded text is very
> cumbersome."
> "Variables should be escaped prior to the theme layer in a way appropriate for their most
> likely use."

**Sources:** https://www.drupal.org/docs/develop/security/why-does-drupal-filter-on-output
and https://www.drupal.org/docs/develop/security/handle-user-input-with-care

### G12. Use the right sanitizer for the context
**Rule:**
- Plain text output → `Html::escape()`
- Text that may contain a limited set of HTML tags → `Xss::filter()`
- Text authored by trusted admin users that may contain most HTML → `Xss::filterAdmin()`
- URLs → `UrlHelper::stripDangerousProtocols()`
- Translatable strings with dynamic values → `t()` / `formatPlural()` with placeholders

**Why (verbatim):**
> "Use Html::escape() for plain text."
> "Use Xss::filter() for text that should allow some HTML tags. Do not use it for HTML
> elements or attributes inside of a tag."
> "Use Xss::filterAdmin() for text entered by admin users that should allow most HTML."
> "Strings sanitized by `t()`, `Html::escape()`, `Xss::filter()`, or `Xss::filterAdmin()`
> are automatically marked safe when rendered."

And on the API class itself: `Xss::filterAdmin()` — "Applies a very permissive XSS/HTML
filter for admin-only use." … "Use only for fields where it is impractical to use the whole
filter system".
**Sources:** https://www.drupal.org/docs/security-in-drupal/sanitizing-output and
https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Component%21Utility%21Xss.php/class/Xss/11

### G13. Translation placeholders: `@`, `%`, `:` — and never pass a variable as the translatable string
**Rule:** Use `t('Hello @name', ['@name' => $name])`. Choose `@variable` for plain values,
`%variable` for values wrapped in `<em>`, `:variable` for a URL used in an `href`
attribute. Never `t($some_variable)`.
**Why (verbatim):**
> `@variable`: "When the placeholder replacement value is a string or a MarkupInterface object"
> `%variable`: "When the placeholder replacement value is to be wrapped in em tags."
> `:variable`: "When the placeholder replacement value is a URL to be used in the "href" attribute"
> "only strings should be passed to translation functions, **not the values of variables**."
> … "Passing a variable directly to translation is unsafe and should not be used."

**Sources:** https://www.drupal.org/docs/security-in-drupal/sanitizing-output and
https://www.drupal.org/docs/develop/theming-drupal/twig-in-drupal/filters-modifying-variables-in-twig-templates

### G14. Use Form API for all forms and user input — it carries the CSRF protection
**Rule:** Never hand-roll an HTML `<form>` + `$_POST` handler. Use Form API. For non-form
routes that perform an action (e.g. a "delete" link), add `_csrf_token: 'TRUE'` to the
route requirements.
**Why (verbatim):**
> "Module authors should use the Form API for all forms and user-input processing." — https://www.drupal.org/docs/drupal-apis/form-api
> "CSRF protection is now integrated into the routing access system" — https://www.drupal.org/docs/administering-a-drupal-site/security-in-drupal/writing-secure-code-for-drupal
> "_csrf_token: Set to `'TRUE'` for operations not using a form." — https://www.drupal.org/docs/drupal-apis/routing-system/structure-of-routes

**Honesty note:** the Form API landing page states the rule but, as fetched, does **not**
spell out the CSRF rationale; the CSRF statement comes from the security docs and the
routing docs. So the *rule* is sourced, the *causal link* is assembled from two official
pages.

### G15. Every route must declare an access requirement
**Rule:** Each route's `requirements:` must carry `_permission`, `_role`,
`_entity_access`, or a custom `_custom_access` checker. `_access: 'TRUE'` is only for
genuinely public routes and must be a deliberate choice.
**Why (documented behaviour):**
> "_permission: A permission string. Use `,` for **AND** logic and `+` for **OR** logic."
> "_access: Set to `'TRUE'` (string/uppercase) to grant access to everyone."
> "All conditions must be met to grant access."

**Source:** https://www.drupal.org/docs/drupal-apis/routing-system/structure-of-routes
**Honesty note:** the routing docs list these under "Requirements: Access & validation
(**optional**)" and do **not** contain a sentence saying "you must always define access".
"Always declare access explicitly" is therefore a **defensible hardening rule, not a
verbatim documented mandate.** Justify it by G16 (90% of holes are in custom code) rather
than by claiming the routing doc requires it.

### G16. Security posture: assume the bug will be in *our* code
**Rule:** Treat custom modules/themes as the highest-risk surface in the codebase, and keep
core/contrib updated the moment a security advisory lands.
**Why (verbatim):**
> "the vast majority of security holes (90% or more) are present in the custom theme or
> modules written by that site's developers" — because they "did not get the same public
> scrutiny that all code on drupal.org receives."
> "your most important protection is keeping Drupal up to date whenever a security advisory
> is issued for Drupal core or the contributed code you are using."

**Source:** https://www.drupal.org/docs/administering-a-drupal-site/security-in-drupal/is-drupal-secure

### G17. Stay on a security-covered version
**Rule:** The site must run a Drupal core minor that is inside its security-coverage
window. Track the official schedule.
**Why (documented):** Minor releases follow a fixed cadence and each has a defined coverage
window; security releases land in announced windows ("usually … the third Wednesday of the
month"), and the schedule page tells you to "**be ready to update your Drupal sites**". As
of this research the page states Drupal 11 receives security updates through December 2026,
Drupal 10 reaches end of life on December 9, 2026, and Drupal 7 support ended January 5,
2025.
**Source:** https://www.drupal.org/about/core/policies/core-release-cycles/schedule

### G18. Only depend on public API; avoid `@internal`
**Rule:** Custom code may rely on `@api`-tagged interfaces and on documented public APIs.
Do not call anything marked `@internal`, and do not rely on public properties of core
classes.
**Why (verbatim, BC policy):**
> "Contributed and custom code can depend on these APIs. We will try as hard as possible to
> not change this, except in critical cases such as a security issue."
> "Contributed and custom code should avoid calling internal APIs because they might change
> in minor releases."
> On `@api` interfaces: "No methods will be added, removed or changed in a breaking way."
> "Public properties on a class are always considered @internal."
> "Necessary security hardening will take priority over API stability."

Also documented as public API and therefore safe to script against: configuration elements
("must remain compatible from one release of Drupal to another") and CLI tool parameters
("The parameters and options accepted by these tools are themselves considered to be a
public API").
**Source:** https://www.drupal.org/about/core/policies/core-change-policies/bc-policy

### G19. Documentation is mandatory, not optional
**Rule:** Every file, function, constant, class, interface and class member — including
private members — gets a docblock. Summary line: under 80 chars, capitalised, ends with a
period, starts with a third-person-singular present-tense verb ("Calculates…",
"Provides…", "Represents…"). `@param`/`@return` with data types are required. Hook
implementations use exactly `Implements hook_help().` with params/return omitted.
**Why (verbatim):**
> "Every function, constant, class, interface, class member (function, property, constant),
> and file must be documented, even private class members."
> "All summaries (first lines of docblocks) must be under 80 characters, start with a
> capital letter, and end with a period (.)."
> "Data types are required to be included as of Drupal 8.x."
> "Each `@see` reference is on its own line, with no additional text beyond the item being
> referenced."
> `@deprecated in project:version and is removed from project:version.`

**Sources (current):** https://project.pages.drupalcode.org/coding_standards/php/documentation/
— (legacy copy, flagged obsolete: https://www.drupal.org/docs/develop/standards/php/api-documentation-and-comment-standards)

### G20. PHP formatting baseline
**Rule:** 2-space indent, no tabs; Unix line endings; no trailing whitespace; file ends with
a single newline; ~80-char soft limit (function signatures and readable conditions may
exceed); short array syntax with a trailing comma on multi-line arrays; single quotes by
default; spaces around `.` concatenation; space after a cast (`(int) $x`); always use
braces; `elseif` not `else if`; `!=` never `<>`; `??` over `isset()` ternaries; visibility
declared on all methods and properties; **no underscore prefix on private/protected
members**.
**Naming:** functions & procedural variables `lower_snake_case` and **functions must be
prefixed with the module machine name**; constants `ALL_UPPER_SNAKE`; classes/interfaces
`UpperCamelCase` with interfaces suffixed `Interface`; methods/properties `lowerCamelCase`.
**Types:** "Beginning with Drupal 9, parameter and return type hints should be used wherever
possible" for new code — but adding hints to *existing* signatures is a BC break.
**Why:** these are the coding standards, "version-independent and 'always-current'", and
"all new code should follow the current standards".
**Sources (current):** https://project.pages.drupalcode.org/coding_standards/php/coding/ —
(legacy copy: https://www.drupal.org/docs/develop/standards/php/php-coding-standards)

> **Correction — Drupal is NOT PSR-12.** Drupal's PHP standard is its own (2-space indent,
> for instance, is *not* PSR-12, which mandates 4). Drupal's standards are historically
> PSR-2-derived and diverge deliberately. Never state "Drupal follows PSR-12" — state
> "Drupal has its own standard, enforced by the `Drupal` and `DrupalPractice` phpcs rulesets
> in Coder" (https://www.drupal.org/project/coder).

### G21. YAML config file standards
**Rule:** 2-space indent. A config file's name equals its unique config name plus `.yml`.
Simple config names must start with the owning extension's machine name
(`my_module.settings`). Config entities follow `(extension).(config_prefix).(suffix)` —
e.g. `node.type.article`, `views.view.frontpage`, `image.style.thumbnail`.
**Why (verbatim):**
> "Use two spaces to indent in config files. In YAML, the white space has semantic meaning
> to represent nested structures."
> "For simple configuration, the unique configuration name must start with the extension
> name (the machine name of the module, theme, or install profile that owns the
> configuration)."

Length limits: total config name ≤ 250 chars; extension ≤ 50; config prefix ≤ 32; suffix
≤ 150.
**Source:** https://project.pages.drupalcode.org/coding_standards/yaml/configuration-files/

---

## 2. Content architecture — choosing between Content Types, Paragraphs, Blocks and Media

> **IMPORTANT HONESTY FLAG for this whole section.** There is **no single authoritative
> drupal.org decision matrix** for choosing between Content Types, Paragraphs, Blocks and
> Media. The `new.drupal.org` Drupal-CMS pages "What is content modeling?" and "Content
> modeling for layouts" surface in search but **404 on direct fetch** (both `www.` and
> `new.` hosts) — they are not cited here. The most-linked community discussions on this
> exact question are drupal.org **forum threads and module issue queues**, which are not
> official documentation. What follows is built from each tool's **own official
> project/module documentation** — those are real, quotable sources for *what each thing
> is* — and the comparative "when to use which" is explicitly marked as synthesis. Say so
> when you apply it, rather than presenting it as a Drupal rule.

### C1. Content Types — the primary, addressable, listable content unit
**What's documented:** Content types are bundles of the `node` entity; nodes are the
canonical addressable content object with their own URL, revisions, workflow, and
Views/search integration. Entities split into content entities ("user data") and
configuration entities ("site settings").
**Source:** https://www.drupal.org/docs/drupal-apis/entity-api
**Synthesis (not verbatim):** use a Content Type when the thing has its own URL, needs to
appear in listings/feeds/search, or has an editorial lifecycle of its own (a service page,
a blog post, a case study).

### C2. Paragraphs — structured, ordered, **non-reusable** body composition
**What's documented, verbatim from the project page:**
> "Instead of putting all their content in one WYSIWYG body field including images and
> videos, end-users can now choose on-the-fly between pre-defined Paragraph Types."
> "You can also add custom option fields and do conditional coding in your CSS, JS and
> preprocess functions so that end-users can have more control over the look and feel of
> each item. This is way much cleaner and stable than adding inline CSS."
> "Paragraphs module does not come with any default Paragraph Types but since they are
> basic Drupal Entities you can have complete control over what fields they should be
> composed of."

Typical uses named by the project: "Accordions, Tabs, Slideshows, Masonry galleries,
Parallax backgrounds".
**Reusability:** Paragraph *items* are not reusable across content; the *types* are
reusable templates. (This is the key discriminator vs. blocks.)
**Source:** https://www.drupal.org/project/paragraphs
**Also documented in the community discussion (issue queue, NOT official docs — treat as
weak):** Paragraphs use entity reference revisions, so "it works with revisions, you can
revert and so on as a single unit" — https://www.drupal.org/project/paragraphs/issues/3114087

### C3. Content Blocks — **reusable**, placeable in regions, not owned by one node
**What's documented:** A block is "a single piece of content you can move around the page,
to any region." Content block types are fieldable bundles, so blocks can be structured
content, not just a text blob.
**Synthesis (not verbatim):** choose a Content Block when the same content must appear in
more than one place, or must be placed by region/visibility rules rather than sit inside
one node's body.
**Source (weak — forum thread, not official docs):**
https://www.drupal.org/forum/support/theme-development/2013-12-07/when-to-use-blocks-vs-content-types
**[NO STRONG OFFICIAL SOURCE FOUND]** for a normative "blocks vs. paragraphs" rule.

### C4. Media entities — reusable assets, referenced, deliberately not addressable
**What's documented, verbatim:**
> "Media entities are standard Drupal content entities. And are grouped by Media type bundles."
> "Media items are typically images, documents, slideshows, YouTube videos, tweets,
> Instagram photos, etc."
> The Media Library provides "a user interface to allow content creators to easily find and
> use existing media items."
> Visiting a media entity's canonical URL returns HTTP 404 **by design**: "Usually you don't
> want someone to navigate to the Media entity out of context."
> Media Library "is specifically useful in sites with large amounts of content, where the
> media assets can be reused."

**Synthesis:** use Media (not a bare image/file field) when the asset needs metadata (alt,
credit, licence, caption), needs to be reusable across content, or when editors need to
browse/search existing assets. Use a plain image field only for a truly one-off,
metadata-free image.
**Sources:** https://www.drupal.org/docs/8/core/modules/media/overview and
https://www.drupal.org/docs/core-modules-and-themes/core-modules/media-library-module

### C5. The one *hard*, sourced discriminator
**Rule you can state with confidence, because it follows from each tool's own
documentation:**
- Needs its own URL / listing / editorial lifecycle → **Content Type (node)**
- Ordered, structured composition **inside one piece of content**, not shared → **Paragraphs**
- Same content shown in multiple places / placed by region → **Content Block**
- A file asset with metadata that editors reuse → **Media**

**Everything beyond this** (e.g. "prefer Paragraphs over Layout Builder", nesting depth
limits, performance thresholds) is **community opinion, not documented policy** — say so
rather than presenting it as a Drupal rule.

---

## 3. Known gaps and cautions

1. **Drupal is not PSR-12.** Cite "the Drupal coding standard, enforced by Coder's
   `Drupal`/`DrupalPractice` phpcs rulesets", not PSR-12. —
   https://project.pages.drupalcode.org/coding_standards/php/coding/
2. **There is no "Twig security sandbox" rule in Drupal.** Drupal's documented
   template-security mechanism is **autoescaping** (everything printed with `{{ }}` is
   escaped; `|raw` disables it). Twig's `SandboxExtension` is an upstream Twig feature and
   is not presented anywhere on drupal.org as a Drupal theming rule.
3. **"Never modify a released `hook_update_N`" is documented as "never *renumber*."** The
   no-edit rule follows mechanically (executed numbers are recorded in the DB) but is not a
   verbatim quote. —
   https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Extension%21module.api.php/function/hook_update_N/11
4. **"Never edit contrib in place" is implied, not stated** (see G4).
5. **"Every route must declare access" is a hardening rule, not a documented mandate** (see
   G15).
6. **Batch vs. Queue has no official decision rule.** Both APIs are documented; the "when to
   use which" split is synthesis.
7. **Content architecture has no official decision matrix** (see section 2's honesty flag).
8. **Coder no longer lints CSS/JS.** Any instruction to check CSS/JS via `phpcs` is wrong —
   use **Stylelint** and **ESLint** with the official Drupal configuration files. —
   https://www.drupal.org/project/coder
9. **The JS "best practices" page is partly dated** (it still recommends jQuery over
   `document.createElement()`), and it does **not** document `Drupal.behaviors` / `once()` /
   `drupalSettings`. Don't cite it for those.
10. **Many drupal.org URLs now 302 to `new.drupal.org`,** and some `new.drupal.org` paths
    404. Fetching Drupal docs at runtime needs a redirect-and-fallback strategy.
11. **The old `drupal.org/docs/develop/standards/*` tree is self-labelled `[Obsolete]`.**
    Use `https://project.pages.drupalcode.org/coding_standards/`.
