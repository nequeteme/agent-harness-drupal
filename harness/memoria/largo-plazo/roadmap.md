# Roadmap

Vista de conjunto de fases y tareas, mantenida por
[project_writer](../../agentes/agente-project-writer.md) como espejo legible
en el repo del tablero visual en GitHub (`site danemar`,
https://github.com/users/nequeteme/projects/2). Cada fase enlaza a sus
Issues.

## Épico: sitio Danemar Parceros (plan-02, contenedor)

[Issue #8](https://github.com/nequeteme/danemarparceros-site/issues/8) —
plan contenedor, no ejecutable en sí mismo, agrupa los planes 01-13 en
`docs/plans/` (fuera del repo git de `site/`, en la raíz del contenedor
`danemar_site/`). **Los 12 planes ejecutables (01, 03-13) ya están
`Completed`** según `docs/plan-memoria.md`, con pendientes menores no
bloqueantes documentados en cada uno.

| Plan | Issue | Status | Pendiente conocido |
|---|---|---|---|
| 01 — Inicialización Drupal 11 + DDEV | [#7](https://github.com/nequeteme/danemarparceros-site/issues/7) | Done | — |
| 03 — Análisis del sitio actual | [#9](https://github.com/nequeteme/danemarparceros-site/issues/9) | Done | — |
| 04 — Módulos y dependencias | [#10](https://github.com/nequeteme/danemarparceros-site/issues/10) | Done | — |
| 05 — Tema base (starterkit) | [#11](https://github.com/nequeteme/danemarparceros-site/issues/11) | Done | — |
| 06 — Arquitectura de contenido (Paragraphs) | [#12](https://github.com/nequeteme/danemarparceros-site/issues/12) | Done | — |
| 07 — Idiomas / multiidioma | [#13](https://github.com/nequeteme/danemarparceros-site/issues/13) | Done | Sin detección automática de idioma en `/` (límite de Drupal core, aceptado) |
| 08 — Componentes de theme | [#14](https://github.com/nequeteme/danemarparceros-site/issues/14) | Done | — |
| 09 — Estilos (CSS del diseño) | [#15](https://github.com/nequeteme/danemarparceros-site/issues/15) | Done | — |
| 10 — Contenidos reales | [#16](https://github.com/nequeteme/danemarparceros-site/issues/16) | Done | Logos reales de clientes pendientes (se usan placeholders) |
| 11 — Formulario de contacto | [#17](https://github.com/nequeteme/danemarparceros-site/issues/17) | Done | — |
| 12 — QA y lanzamiento | [#18](https://github.com/nequeteme/danemarparceros-site/issues/18) | Done | `og:url` de portada muestra `/node/N` en vez de la URL limpia (cosmético) |
| 13 — SEO técnico + GEO/AEO | [#19](https://github.com/nequeteme/danemarparceros-site/issues/19) | Done | `Service` por tarjeta en Schema.org pendiente; NIF/credencial Acquia sin mapear a ningún tag del submódulo |

**Conclusión**: el sitio está funcionalmente listo para revisión del
usuario antes de producción (`docs/plan-memoria.md`, sesión 2026-08-18).

## Trabajo del 2026-08-18 (post-épico, ya bajo el flujo de PRs)

Tareas surgidas después de cerrar el épico, parte del nuevo proceso de
"siempre commit + PR, nunca commit directo a `develop`".

| Tarea | Issue | Status | Nota |
|---|---|---|---|
| Fix banner de cookies transparente (tema Forge) | [#3](https://github.com/nequeteme/danemarparceros-site/issues/3) | In review | PR [#2](https://github.com/nequeteme/danemarparceros-site/pull/2), revisado por `revisor-pr` sin hallazgos bloqueantes, verificado por `tester` (7/7 criterios). Pendiente de merge por el usuario. |
| Infraestructura de PRs (phpcs + stylelint + gh CLI) | [#4](https://github.com/nequeteme/danemarparceros-site/issues/4) | In review | PR [#1](https://github.com/nequeteme/danemarparceros-site/pull/1), revisado por `revisor-pr` (corrigió `phpcs.xml` sin excluir `node_modules/`). Pendiente de merge por el usuario. |
| Bug: `count()` sobre `FieldItemList` en `paragraph--hero.html.twig` | [#5](https://github.com/nequeteme/danemarparceros-site/issues/5) | Done | Diagnosticado a fondo por `desarrollo-drupal` (7 variantes probadas: 0/1/3 valores en `field_features`, ruta publicada, "Latest version" con `content_moderation`, `/node/3` original) — **no reproduce en el código actual**. Causa raíz: la línea 27 hoy usa el filtro `\|length` de Twig, que llama a `count()` nativo de PHP sin pasar por el sandbox de Drupal, así que no puede disparar ese error tal como está escrita. Las 2 entradas de `watchdog` con el error real (`wid=100/101`) son de una iteración local del template previa al commit de la versión actual — no del código que existe hoy. Ver comentario completo en el Issue. Issue cerrado (cierre automático del proyecto al mover a Done).
| Migración completa a SASS | [#6](https://github.com/nequeteme/danemarparceros-site/issues/6) | Ready | Pedido nuevo del usuario (no relacionado con que plan-09 ya haya adoptado CSS plano). Investigado (sin SDC propio, 4 CSS vía `libraries.yml`), no implementado todavía. Movido de Backlog a Ready. |
| Investigar bug de banner de cookies en /es y /pt (posible fix no desplegado) | [#21](https://github.com/nequeteme/danemarparceros-site/issues/21) | Done | El agente `frontend` verificó con Playwright real 12 combinaciones (en/es/pt × forge/dossier × desktop/mobile): no reproduce en ningún caso. Era el entorno de pruebas local desactualizado respecto al fix del PR #2, ya sincronizado. Nota aparte no accionable: en dossier+es+mobile el banner ocupa ~36% del viewport por longitud del copy en español (vs ~26-28% en el resto) — tema de longitud de texto, no amerita Issue nuevo. |
