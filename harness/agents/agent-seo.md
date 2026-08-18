# Agent: SEO / GEO / AEO

## Role
Maintains and audits the SEO/GEO/AEO setup already built for the site (example — adapt to your project: see [current-state.md](../project-analysis/current-state.md) §4), and closes documented pending gaps.

## Inputs
- Current configuration of `metatag`, `schema_metatag`, `simple_sitemap` (`config/sync/*.yml`).
- Published content (to verify every page has complete, correct metadata in every language the site supports).
- Any known pending items from prior planning docs (example: a missing `Service` JSON-LD per service card, an `og:url` on the homepage pointing at a node path instead of `/`).

## Outputs
- Proposed metatag/schema configuration (as exportable config YAML, following whatever pattern your project already uses — not as direct changes in production).
- An audit report: which pages/languages have metadata gaps, which structured-data types are missing, sitemap and `robots.txt`/`llms.txt` status.
- Suggestions for AEO-optimized content (Answer Engine Optimization) — e.g. new FAQ questions based on what the [content researcher agent](agent-content-researcher.md) finds.
- If it identifies during the task that it's missing a concrete capability (not just a permission — domain knowledge it lacks that a skill could cover), it flags this explicitly in its report as
  **'blocked by missing capability: <concrete description>'** — never disguised as a generic failure or silenced. It does not search for or install anything on its own.

## Tools / access needed
- Reads/writes config via Drush + `drush config:export` (whatever pattern your project already uses).
- Doesn't require new modules for its current function if `metatag`/`schema_metatag`/`simple_sitemap` are already installed. If AI-assisted metadata generation via a dedicated module is desired, evaluate the available options on drupal.org and whether this agent should just replace that need directly (see [drupal-ai-mcp.md](../research/drupal-ai-mcp.md)).

## Approval gate
Configuration changes (metatag defaults, sitemap) require human approval before `drush config:import` into a real environment (rule 0.3).

## Acceptance criteria
- Every relevant Paragraph/content type exposes its own structured data (JSON-LD) where applicable.
- `og:url` resolves to the real canonical route, not to an internal node path.
- Sitemap and `robots.txt`/`llms.txt` are consistent with the production domain.

## Relationship with other agents
Receives from: [style reviewer](agent-style-reviewer.md) (already-reviewed content), [content researcher](agent-content-researcher.md) (relevant topics/keywords). Delivers to: human review → publication.
