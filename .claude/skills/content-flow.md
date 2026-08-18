---
name: content-flow
description: Orchestrates the [Your Project] harness's full editorial flow (research → writing → style → SEO → human review → publication). Use it when the user asks to generate/update site content end to end, or invokes /content-flow.
---

Orchestrates the flow documented in `harness/flows/content-flow.md`.
Follow these steps, invoking each subagent with the `Agent` tool
(`subagent_type` = agent name) and explicitly passing the output of one
step as input to the next — don't let an agent "guess" what the previous
one did:

1. If the user gave a concrete topic/brief, skip to step 3. If not, invoke
   `news-researcher` (and optionally `content-researcher`) to detect
   signals/topics with real value. Don't generate content just to generate
   content.
2. If the topic warrants deeper research, invoke `content-researcher` with
   the detected topic/signal.
3. Invoke `content-writer`, passing it the research dossier (or the user's
   brief) and the concrete section/field of the site to write or update.
4. Invoke `style-reviewer` on the resulting draft.
5. Invoke `seo` on the draft already reviewed for style.
6. **Don't ask the user for approval yourself.** Return the final draft
   (content + proposed metadata) to the `orchestrator` (skill
   `/orchestrator`) — it's the orchestrator, following
   `harness/agents/agent-orchestrator.md`, that decides how to present it
   and asks for explicit approval before transitioning `moderation_state`
   to `published` (rule 0.3 of `AGENTS.md`). If this flow was invoked
   directly (without going through `/orchestrator` first), act yourself
   under that document's rules at this step — the user should only receive
   that question from the orchestrator role, never from the raw flow. If
   the user rejects it, go back to the relevant step depending on the
   reason (form → `content-writer`/`style-reviewer`; substance →
   research).
7. Invoke `documenter` to record what happened in `harness/memory/`.

This flow never talks to the user directly — it always reports back to the
orchestrator (real or assumed, see step 6).

Content state (`draft` → `review` → `published`) uses the site's editorial
workflow via `content_moderation`/`workflows` if enabled (see
`harness/project-analysis/current-state.md`). Every agent must leave
content in `draft` or `review`, never `published` directly.
