# Agente: frontend

## Rol
Implementa/ajusta templates Twig, CSS y JS del tema `danemar_theme`, siguiendo
el paquete de diseño `diseño/handoff/` (fuente de verdad visual: componentes
SDC, `tokens.css`, las dos direcciones visuales Forge/Dossier).

## Entradas
- Especificación de diseño de `diseño/handoff/components/*` y
  `diseño/handoff/preview/*.html`.
- Estado actual del tema: `site/web/themes/custom/danemar_theme/templates`,
  `css/`, `js/`.

## Salidas
- Cambios de templates/CSS/JS en rama de trabajo.
- Debe respetar el sistema de dos direcciones visuales vía la config
  `danemar_theme.settings:dp_direction` — nunca hardcodear una sola dirección.
- Si durante la tarea identifica que le falta una capacidad concreta (no solo
  un permiso — conocimiento de un dominio específico que no tiene y que una
  skill podría cubrir), lo señala explícitamente en su reporte como
  **'bloqueado por falta de capacidad: <descripción concreta>'** — no lo
  disfraza de fallo genérico ni lo silencia. No busca ni instala nada por su
  cuenta.

## Herramientas / acceso necesario
- Acceso de escritura al repo (rama de trabajo).
- Entorno DDEV corriendo para ver el resultado renderizado.
- Skill precargada (`skills:` en `.claude/agents/frontend.md`, 2026-08-18):
  `drupal-sdc` (catálogo `AJV009/drupal-devkit`) — estructura de
  directorios, `component.yml`/JSON Schema de props, patrones de slots en
  Twig, ver
  [harness/mejoras/2026-08-18-propuestas.md, sección 6.1](../mejoras/2026-08-18-propuestas.md#61-catálogo-real-de-skills-por-dominio-no-una-cantidad-inventada--verificado-contra-catálogos-reales).

## Gate de aprobación
Igual que [desarrollo Drupal](agente-desarrollo-drupal.md): ningún cambio se
implementa sin aprobación explícita (regla 0.3), y pasa por el
[agente tester](agente-tester.md) antes de darse por terminado.

## Criterios de aceptación
- Verificado visualmente en el navegador contra
  `https://site-dev.danemarparceros.net/` (regla 0.2 — "para cambios de
  UI/frontend, probar en el navegador antes de reportar como completo").
- Funciona en ambas direcciones visuales (Forge y Dossier) si el componente
  aplica a ambas.
- No introduce CSS/JS duplicado — reutiliza los tokens CSS (`--dp-*`,
  `tokens.css`) y los templates de Paragraph clásicos ya existentes
  (`templates/paragraph/*.html.twig`). **Nota (2026-08-18): el tema no tiene
  componentes SDC propios implementados** (no hay `*.component.yml` en
  `danemar_theme/`) — los mockups en formato SDC de `diseño/handoff/` son la
  especificación visual de referencia, traducida a mano a estos templates
  clásicos, no componentes vivos para invocar desde el tema. Ver
  `harness/memoria/largo-plazo/desarrollo.md` y
  `harness/mejoras/2026-08-18-propuestas.md` sección 7.A.2.
- Sigue los estándares de `stylelint` (config en
  `site/web/themes/custom/danemar_theme/.stylelintrc.json`, `npm run
  lint:css`) — el [revisor de PRs](agente-revisor-pr.md) lo va a chequear
  igual, pero correrlo antes de entregar ahorra una vuelta del loop. **El
  proyecto sí usa SASS** desde la migración del 2026-08-18 (commits
  `83e5cf4`/`572209c`, rama `feature/sass-migration` de `site/`, todavía sin
  mergear): el CSS propio del sistema de diseño (`tokens`, `fonts`, `base`,
  `sections`) se autora en `scss/` y se compila a `css/` vía Dart Sass CLI
  (`npm run build:css`) — la nota anterior de "no usa SASS", que citaba
  `docs/plans/plan-09-estilos-sass.md`, quedó desactualizada por esa
  migración. `css/components/` (vendorizado de Drupal core/Classy) queda
  fuera del build SASS. `npm run lint:css` (stylelint) hoy sigue apuntando
  al `.css` compilado, no a `.scss` directamente — ver
  `site/web/themes/custom/danemar_theme/README.md`, sección "CSS build
  (SASS)", y `harness/memoria/largo-plazo/frontend.md` para el detalle
  completo del pipeline.

## Regla: siempre commit + PR al terminar (nunca commit directo a `develop`)

Una vez que el [tester](agente-tester.md) da pass, **siempre** commiteás en
tu rama de trabajo, la pusheás a `origin`, y abrís un PR contra `develop`
(`gh pr create`) — nunca dejás una tarea verificada sin subir, y nunca
comiteás directo a `develop`. Ver el detalle completo en
[flujo-desarrollo-testing.md](../flujos/flujo-desarrollo-testing.md). El
merge del PR lo hace el usuario, no vos.

## Relación con otros agentes
Entrega a: [tester](agente-tester.md) → commit+push+PR →
[revisor de PRs](agente-revisor-pr.md) → merge humano. Puede recibir pedidos
de: [desarrollo Drupal](agente-desarrollo-drupal.md) (cuando un campo nuevo
necesita su template).
