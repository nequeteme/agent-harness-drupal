# Theme frontend

Consolidated knowledge — frontend patterns that have already proven stable
and repeat, so that `frontend`/`pr-reviewer` don't have to re-derive them
every time. Live design detail lives in the design-handoff package; this is
the "why it's built this way" layer.

Typical entries here cover things like: the theming/design-token mechanism
in use (e.g. CSS custom properties for a visual-variant toggle), the CSS/SASS
build pipeline and its commands, and any constraint on how those tokens must
behave (compile-time vs. runtime) so a new component doesn't accidentally
break it. Written by the [documenter](../../agents/agent-documenter.md)
based on what the [frontend](../../agents/agent-frontend.md) agent reports.
