# Memoria de corto plazo

Log rotativo. El [documentador](../agentes/agente-documentador.md) agrega una
entrada al final de cada tarea/flujo, y de tanto en tanto "gradúa" lo estable
hacia [largo-plazo/](largo-plazo/) y recorta lo que ya no es útil aquí.

---

## 2026-08-18 — Implementación de la Opción D del harness

- Se habilitaron `content_moderation` + `workflows` (core de Drupal), con
  workflow "Editorial" (`draft` → `review` → `published`) aplicado a
  `landing_page` y `legal_page`. Commit `339691a` en `develop`
  (`site/config/sync/workflows.workflow.editorial.yml`).
- Se crearon los 8 subagentes de ejecución (`.claude/agents/`) y 2 skills de
  flujo (`.claude/skills/`).
- Se verificó extremo a extremo: revisión de prueba en `draft` no afecta la
  revisión publicada en vivo; todas las rutas del sitio responden 200 tras
  el cambio.
- **Pendiente detectado, no resuelto**: error preexistente (no causado por
  este cambio) en `paragraph--hero.html.twig` línea 27 — `count()` sobre un
  `FieldItemList` bloqueado por el sandbox de Twig, visible al pedir
  `/node/3`. Queda como tarea suelta para `desarrollo-drupal`/`frontend`.
  **Actualización (mismo día, más tarde)**: investigado a fondo por
  `desarrollo-drupal` — no reproduce en el código actual (probadas 7
  variantes: 0/1/3 valores en `field_features`, ruta publicada, "Latest
  version" con `content_moderation`, `/node/3` original). La línea 27 hoy
  usa el filtro `|length` de Twig, que no pasa por el sandbox de Drupal, así
  que no puede disparar ese error tal como está escrita; las entradas de
  `watchdog` con el error real eran de una iteración local previa al commit
  de la versión actual del archivo. Detalle completo en el comentario del
  [Issue #5](https://github.com/nequeteme/danemarparceros-site/issues/5),
  ya movido a `Done` y cerrado.
- Se agregaron orquestador, planificador, documentador (este sistema de
  memoria) e investigador de mejoras del harness — mismo día, misma sesión.
- **Decisión del usuario**: el informe del `investigador-mejoras-harness`
  queda en disparo **manual** por ahora (no `/loop`, no rutina de cron vía
  `schedule`). Revisar con el usuario si esto cambia más adelante antes de
  asumir que se puede automatizar.
- **Reglas nuevas del usuario, ya reflejadas en diseño y en los subagentes
  runtime**: (1) el planificador no debe dimensionar ninguna tarea por
  encima del 60% del contexto del agente que la ejecuta; (2) el tester
  certifica "done" solo al 100% de los criterios de aceptación, verificado
  con más de un método (nunca un pass parcial). Graduado directo a
  `largo-plazo/decisiones-arquitectura.md` por ser reglas permanentes de
  funcionamiento, no un evento puntual.
- **Comunicación centralizada en el orquestador**: a pedido del usuario,
  ningún flujo ni agente le habla al usuario directamente — todo pasa por el
  orquestador, incluso si un flujo se invocó directo. Se actualizaron
  `agente-orquestador.md`, `agente-tester.md`, ambas skills de flujo, y su
  runtime en `.claude/`. Graduado directo a
  `largo-plazo/decisiones-arquitectura.md`.
- **Renombre de comandos** (a pedido del usuario, 2026-08-18):
  `/flujo-contenido` → `/dp-cont`, `/flujo-desarrollo-testing` → `/dp-dev`,
  `/orquestador` → `/dp`. Se renombraron los archivos en `.claude/skills/`
  (`dp-cont.md`, `dp-dev.md`, `dp.md`, con `name:` actualizado) y se
  corrigieron todas las referencias cruzadas en `harness/` y `.claude/`. Los
  nombres de los documentos de diseño (`harness/flujos/flujo-contenido.md`,
  `flujo-desarrollo-testing.md`, `harness/agentes/agente-orquestador.md`) NO
  cambiaron — solo los comandos/skills invocables.
- **Verificación real de que `/dp` (y en general las skills/subagentes nuevos
  del día) no estaban disponibles en la sesión que ya estaba corriendo** —
  tanto el tool `Skill` como el tool `Agent` con `subagent_type` custom
  fallaron hasta que una notificación posterior del sistema los listó como
  disponibles. Conclusión para futuras sesiones: los comandos/agentes nuevos
  creados en `.claude/` durante una sesión activa no quedan utilizables de
  inmediato en esa misma sesión con certeza — puede hacer falta esperar a
  que el harness los recargue, o abrir una sesión nueva.
- **Primer caso de uso real end-to-end de la Opción D**: bug reportado por
  el usuario (banner de cookies con fondo casi transparente en tema Forge,
  causando que el contenido de atrás se leyera mezclado con el texto del
  banner; también ocupaba ~45% del viewport en mobile). Diagnosticado por
  `frontend` con Playwright real (no solo lectura de CSS), aprobado por el
  usuario, implementado (`var(--dp-surface)` → `var(--dp-bg)` +
  `@media(max-width:640px)` más compacto) en la rama
  `fix/cookie-banner-visibility`, verificado por `tester` con 7/7 criterios
  pass (Playwright, config real alternando `dp_direction`, logs, caso
  borde). Único archivo tocado:
  `site/web/themes/custom/danemar_theme/css/sections.css`. **Pendiente de
  cerrar como PR** (ver regla de branches+PR de abajo) — no se comiteó
  directo porque, a mitad de esta misma tarea, el usuario pidió el cambio de
  proceso a PRs; el cierre de este caso concreto se retomó bajo el nuevo
  flujo.
- **Regla nueva: commit + PR siempre, nunca commit directo a `develop`** —
  ver detalle completo, ya graduado, en
  `largo-plazo/decisiones-arquitectura.md` (incluye la infraestructura
  conectada: remoto GitHub, `gh` CLI, `phpcs`, `stylelint`). Se agregó el
  noveno subagente de ejecución, `revisor-pr`.
- **PR #1 (tooling) revisado por `revisor-pr`**: encontró un hallazgo real
  (`phpcs.xml` sin excluir `node_modules/`, 52 falsos positivos sobre una
  dependencia vendorizada de stylelint) — corregido en el momento (commit
  `1e94512`), verificado, comentado en el PR. **PR #2 (fix del banner de
  cookies)**: sin hallazgos bloqueantes, corrió `stylelint` de verdad
  (trayendo temporalmente config del PR #1 sin commitearla). Ambos PRs
  quedaron **pendientes de merge por el usuario** —
  https://github.com/nequeteme/danemarparceros-site/pull/1 y `/pull/2`.
- **Se agregaron `project_writer` e `historiador`** (a pedido explícito del
  usuario) — ver detalle graduado en `largo-plazo/decisiones-arquitectura.md`.
  `gh` ahora tiene scope `project` (lectura+escritura de GitHub Projects,
  autorizado por el usuario vía device flow, dos veces: una para
  `read:project`, reemplazada por una segunda con `project` completo cuando
  el usuario pidió también poder escribir). Confirmado acceso real a los dos
  proyectos de la cuenta: `site danemar` (#2, el que se usa, 0 tarjetas
  todavía) y `layer kanban` (#1, el que sirvió de referencia de estructura).
  Se creó una primera entrada de historia real:
  `harness/historia/2026-08-18-el-banner-de-cookies-que-se-transparentaba.md`.
- **Pendiente real, no ejecutado todavía**: la migración completa a SASS
  que pidió el usuario (ver decisión graduada en
  `largo-plazo/decisiones-arquitectura.md`). Se investigó la estructura del
  tema (confirmado: no hay SDC propio, los 4 archivos van directo por
  `libraries.yml`) pero no se llegó a implementar — la sesión se desvió
  hacia el pedido de GitHub Projects. **Retomar esto en la próxima tarea de
  desarrollo**, idealmente ya usando el flujo completo (planificador →
  project_writer crea la tarjeta → dp-dev → revisor-pr → PR).
  **Actualización (mismo día, más tarde): implementada.** Pipeline
  `scss/` → `css/` vía Dart Sass CLI, commits `83e5cf4`/`572209c` en rama
  `feature/sass-migration` de `site/` — **todavía sin mergear, PR
  pendiente**. Detalle del pipeline en
  `harness/memoria/largo-plazo/frontend.md` y en el propio
  `site/web/themes/custom/danemar_theme/README.md`. Con esto, la nota "el
  proyecto no usa SASS" (citando `docs/plans/plan-09-estilos-sass.md`) que
  repetían `agente-frontend.md`/`agente-revisor-pr.md` y sus espejos en
  `.claude/agents/` quedó desactualizada — corregida en los 4 archivos.

## 2026-08-18 (misma fecha, sesión posterior) — Convención de capacidad + memoria de largo plazo extendida

Implementación de la sección 6.6 ("Implementar ya") de
`harness/mejoras/2026-08-18-propuestas.md`, en paralelo con otro agente que
instaló las skills de las secciones 6.1/6.2 (ver notas de `skills:` ya
reflejadas arriba en `agente-desarrollo-drupal.md`/`agente-frontend.md` y
sus espejos, hechas por ese otro agente, no por este).

- **Convención "bloqueado por falta de capacidad"** (sección 6.3) agregada
  a la sección "Salidas"/"Entregable" (o equivalente más cercano, p. ej.
  "Si pasa" en `tester`) de los 14 `harness/agentes/agente-*.md` que no son
  el orquestador, y sus 14 espejos en `.claude/agents/*.md` — nota: el
  informe original decía "13" en la sección 6.3/6.6, pero el conteo real de
  agentes de ejecución (tabla de la sección 0.1 del mismo informe, que
  incluye a `investigador-mejoras-harness`) es 14; se aplicó a los 14
  reales, no al número textual del informe. Se agregó también la
  contraparte en `agente-orquestador.md` (paso 5 de "Cómo opera"): al
  recibir esa marca en un reporte, dispatcha a
  `investigador-mejoras-harness` con el hueco concreto.
- **Memoria de largo plazo extendida** (sección 6.4): creados
  `largo-plazo/frontend.md` (toggle Forge/Dossier vía custom properties +
  pipeline SASS), `largo-plazo/desarrollo.md` (Paragraphs anidados como
  modelo de datos, aclarando que es capa distinta y complementaria a SDC —
  no reemplazo) y `largo-plazo/contenido.md` (tono ES/EN/PT: redactar
  primero en español, traducir después vía `content_translation`, nunca
  reescribir desde cero — ya confirmado como práctica en
  `agente-creador-contenidos.md`).
- **Esqueleto de métricas** (sección 6.5): creado
  `largo-plazo/metricas.md` con las 3 tablas vacías (pass/fail del tester
  por tipo + intentos, rondas tester↔revisor-pr por PR, tareas
  re-planificadas por la regla del 60%), documentando quién la llena
  (documentador, al cierre de cada tarea/flujo) y quién la lee
  (investigador-mejoras-harness, cada ~2 días).
- **Corrección de la nota desactualizada sobre SASS** (sección 6.6 punto D):
  ver la actualización de la entrada anterior en este mismo archivo — 4
  archivos corregidos (`agente-frontend.md`, `agente-revisor-pr.md` y sus
  espejos en `.claude/agents/`).
- **Pendiente**: nada de lo anterior requiere aprobación humana (es
  documentación/config, regla 0.3 libre), pero la migración SASS en sí
  sigue con **PR sin mergear** — eso sí necesita acción del usuario cuando
  quiera revisarlo/mergearlo.

## 2026-08-18 (misma fecha, pasada posterior) — Corrección: el tema NO usa SDC de verdad

`investigador-mejoras-harness` detectó (sección 7.A.2 de
`harness/mejoras/2026-08-18-propuestas.md`) que la entrada anterior de esta
misma sesión (arriba, "Memoria de largo plazo extendida") escribió en
`largo-plazo/desarrollo.md` una afirmación falsa: que SDC es la capa de
renderizado del tema y "coexiste" con los Paragraphs. Verificado contra el
filesystem (`find site/web/themes/custom/danemar_theme -name
"*.component.yml"` → 0 resultados): el tema usa **Twig clásico**
(`templates/paragraph/*.html.twig`), traducido a mano desde mockups SDC de
`diseño/handoff/` (paquete de diseño de referencia, no código que corre).
Esto ya estaba correctamente documentado en
`docs/plans/plan-08-componentes-theme.md` (decisión: clásicos, sin SDC) y en
`harness/memoria/largo-plazo/decisiones-arquitectura.md` (escrito *antes* en
el mismo día) — la entrada nueva de `desarrollo.md` lo contradijo por no
verificar el estado real del filesystem antes de escribir.

- **Corregido `harness/memoria/largo-plazo/desarrollo.md`**: reemplazada la
  afirmación de "Paragraphs + SDC como capas que coexisten" por la cadena
  real (Paragraphs = modelo de datos; Twig clásico = renderizado real;
  mockups SDC en `diseño/handoff/` = solo referencia visual, ya traducida a
  mano), dejando explícito que es una decisión ya tomada
  (`plan-08-componentes-theme.md`), no un hueco.
- **Corregido `harness/agentes/agente-frontend.md`** (línea de "Criterios de
  aceptación" que decía "reutiliza tokens y componentes SDC existentes",
  ambigua porque el tema no tiene componentes SDC propios que reutilizar) —
  aclarado que se reutilizan tokens CSS y templates de Paragraph clásicos,
  con nota explícita de que no hay SDC implementado en el tema.
- **Revisado y sin cambios necesarios**: `.claude/agents/frontend.md` (su
  mención de SDC ya estaba correctamente acotada a `diseño/handoff/`, el
  paquete de diseño, no al tema),
  `harness/memoria/largo-plazo/decisiones-arquitectura.md` (ya decía
  correctamente "no hay componentes SDC propios implementados") y
  `harness/memoria/largo-plazo/roadmap.md` (ya decía "sin SDC propio").
- Esta misma entrada de corto plazo (línea "aclarando que es capa distinta y
  complementaria a SDC — no reemplazo", más arriba en este archivo) queda
  **sin reescribir** — es historial de lo que se hizo/creyó en su momento,
  no se reescribe (convención de este archivo) — pero queda invalidada por
  esta entrada posterior.
