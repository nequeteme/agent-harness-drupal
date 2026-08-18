# AGENTS.md

General instructions for any agent (human or AI) working in this
repository. Must be loaded and respected automatically before doing any
task.

## 0.1 Language

- All conversation with the user (explanations, questions, summaries) goes
  in the user's preferred language — set this explicitly for your project.
- All code goes in ENGLISH: variable/function/class/method names, routes,
  table/column names, commit messages, and code comments. No identifier or
  comment in any other language, without exception (unlike projects where
  domain vocabulary is kept in the source language — here everything is in
  English unless the user says otherwise for a term with no reasonable
  translation).
- If "readable code" and "everything in English" conflict, "everything in
  English" wins.

## 0.2 Honesty and transparency (mandatory)

- Never claim something "works" or is "done" without having verified it
  (running the site, running tests, checking the actual result). If it
  wasn't verified, say so explicitly: "this hasn't been tested yet."
- Never invent Drupal modules, APIs, hooks, config fields, or behavior that
  hasn't been confirmed in the official documentation or the project's own
  code. If unsure, say so and offer to verify before continuing.
- Don't hide ambiguity: if an instruction admits more than one
  interpretation, state the ambiguity explicitly, present options with
  pros/cons, recommend one, and wait for confirmation before implementing.
- Don't swallow errors: if a command fails, a module won't install, a test
  doesn't pass, report it as-is (don't silence it with an empty try/catch or
  carry on as if nothing happened). This applies to code too: a `catch` that
  only logs and continues without really propagating or handling the error
  is forbidden.
- If something done in a previous session turns out to be broken or
  mis-documented, say so and fix the record (memory/plan) instead of leaving
  it inconsistent.
- When something is fully working, it works 100%; if it's not working
  100%, the failure needs to be fixed.

## 0.3 Approval before implementing

- Code is not implemented (custom modules, theme, exported configuration,
  migrations) without the user's explicit approval ("implement it", "do
  it", "go ahead", or equivalent).
- Analysis, design, research on modules/approaches, and documentation do
  NOT require prior approval — those can be done freely.
- Stop signals: "I want you to analyze/document/research/explain", "don't
  implement yet", "I just want to understand", "help me plan".

## 0.4 Test environment (public URL + tunnel)

Project-specific section — fill in for each concrete project that adopts
this harness. In the original project this harness was extracted from, it
documented the public staging URL and the command (with a real Cloudflare
Tunnel token) to bring the tunnel up. That command/token does **not**
travel in this template, since it would be a real credential; it's replaced
here with a placeholder:

```
cloudflared tunnel run --token <REAL_TOKEN_NOT_INCLUDED_IN_THIS_REPO>
```

When adopting this harness for a new project, fill in this section with
that project's real data (staging URL, tunnel command if applicable) — and
if this file ever contains a real secret again, keep it out of any repo
that gets shared or published.

## 0.5 Commits after implementing a plan

- After implementing the code for a plan (or a working part of a plan),
  commit to the `develop` branch — never leave uncommitted changes from a
  task that's already done and verified.
- If `develop` doesn't exist yet, create it before the first implementation
  commit.
- Never commit half-finished or unverified code (see 0.2): verify it works
  first, then commit.
- Commit messages in English (see 0.1).
