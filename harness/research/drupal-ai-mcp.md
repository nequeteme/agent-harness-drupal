# Research: Drupal + AI agents

> Source: web research + `drupal.org` (August 2026). No exact version or
> compatibility with the target site's Drupal version was verified via a
> real installation — confirm at the time of installing any of these
> modules.

## 1. Drupal's official AI initiative

Dries Buytaert published a 2026 roadmap with eight AI capabilities for
Drupal core/ecosystem: background agents reacting to triggers/schedules,
integration with the design system, content creation/discovery, advanced
governance, multichannel campaigns, among others. Stated goal: "democratize"
SEO, brand, and accessibility expertise within the normal editorial flow —
not as an external plugin, but as part of Drupal itself.

Source: [Drupal's AI Roadmap for 2026](https://www.drupal.org/blog/drupals-ai-roadmap-for-2026)

## 2. Relevant contrib modules (not installed in this project today)

| Module | What it does | Relevance to the harness |
|---|---|---|
| [`ai`](https://www.drupal.org/project/ai) | Abstraction layer over model providers (48+, including Anthropic). Requires Drupal 10.5/11.2+ per its documentation. | Would be the entry point if agents were to run **inside** Drupal instead of externally. |
| [`ai_agents`](https://www.drupal.org/project/ai_agents) | "Text-to-action" agent framework over Drupal's own config: ships with Field Type Agent, Content Type Agent, Taxonomy Agent out of the box. | Useful for a *Drupal development* agent that modifies config under supervision, not for editorial content. |
| `ai_agents_ossa` ("Open Standard Agents") | Custom tokens for prompts, integration with ECA (Event-Condition-Action) for agent workflows, orchestration via MCP, Cedar policies. | The closest thing to "a harness inside Drupal": ECA as the orchestration engine. |
| `ai_agents_experimental_collection` | Meta-module that installs individual agents (`ai_agents_views`, `ai_agents_content_types`, etc.). | Still experimental — evaluate maturity before relying on it in production. |
| [`mcp_server`](https://www.drupal.org/project/mcp_server) | Turns the Drupal site into an MCP endpoint: exposes nodes/views/Drush as "tools" via OAuth 2.1. | The cleanest path for an **external** harness (Claude Code, another MCP client) to operate on content without needing Drush/SSH. |
| [`mcp_client`](https://www.drupal.org/project/mcp_client) | Connects Drupal, as a client, to external MCP servers. | Only relevant if Drupal itself should consume external tools (e.g. a news service). |
| [`mcp`](https://www.drupal.org/project/mcp) | The original project, in the process of merging into `mcp_server`. | Check which of the two is active at install time. |
| `seo_ai_generator` | Generates SEO metadata via the OpenAI API. | Tied specifically to OpenAI; evaluate whether it fits your provider or whether a custom SEO agent should replace it. |
| [`simple_oauth`](https://www.drupal.org/project/simple_oauth) | OAuth2 for JSON:API/REST. | Standard module for authenticating any external agent that needs to write content via API. |
| [`rest_api_authentication`](https://www.drupal.org/project/rest_api_authentication) | Alternative/complement authentication for REST. | — |

## 3. "Outside AI — The State of Agent Experience in Drupal"

An article on drupal.org identifying real gaps when connecting AI agents to
an already-existing Drupal site: setup/authentication friction, lack of a
machine-readable inventory of structure/permissions/actions, configuration
fragility that custom code can bypass, and lack of independent verification
of what an agent claims to have done (exactly the kind of problem a
project's own AGENTS.md "no unverified claims" rule already tries to prevent
at the human-process level).

It proposes a five-stage architecture, directly applicable to this kind of
project:

1. **Connect** with delegated, auditable identity (not a shared admin
   account).
2. **Understand** the site via machine-readable discovery (what content
   types, fields, permissions exist — today that lives in `config/sync/*.yml`
   and in the [project analysis](../project-analysis/current-state.md)).
3. **Act** through governed interfaces with typed inputs and structured
   "receipts" — "one action model, many gates" instead of direct database
   access.
4. **Verify and recover**, leveraging Drupal's native draft/review/approve
   flows (`content_moderation` + `workflows`).
5. **Rebuild/migrate** with parity auditing (not applicable today — this is
   for large sites undergoing migration).

Source: [Outside AI: The State of Agent Experience in Drupal](https://www.drupal.org/about/ai/initiatives/blog/outside-ai-the-state-of-agent-experience-in-drupal)

## 4. What Drupal exposes today in this project

Confirm in the site's own `core.extension.yml` and `composer.json` whether
`jsonapi`, `rest`, `content_moderation`, and `workflows` are enabled. Before
any of that is wired up, the most readily available automation path is
**Drush scripting** (`drush php:script -`), used to create content types and
load content through Drupal's native APIs. See the
[project analysis](../project-analysis/current-state.md) for the confirmed
detail on a given project.

For external agents (outside a session with direct DDEV/Drush access) to
interact with the site, one of the following is needed:

- Enable `jsonapi`/`rest` (core) + `simple_oauth` (contrib), or
- Install `mcp_server` as a direct bridge to an MCP client (Claude Code,
  Claude Desktop, etc.).

Both options are evaluated in [implementation-options.md](../implementation-options.md).
