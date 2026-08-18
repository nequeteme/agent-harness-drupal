# Decisiones de arquitectura del harness

Conocimiento consolidado — el porqué de las decisiones, no el qué (eso está
en [harness/opciones-implementacion.md](../../opciones-implementacion.md) y
[harness/mapas-arquitectura.md](../../mapas-arquitectura.md)).

- **Se eligió la Opción D (híbrida)** sobre nativa-Drupal (B) o harness
  externo (C) porque el sitio es una landing de una página con volumen de
  contenido bajo — no justifica instalar/mantener `ai`, `ai_agents`,
  `mcp_server` todavía. Ver camino de evolución D→B documentado en
  `opciones-implementacion.md`.
- **El orquestador es una skill, no un subagente anidado**: Claude Code no
  soporta que un subagente invoque a otros subagentes. Ver nota técnica en
  `harness/agentes/agente-orquestador.md`.
- **El workflow "Editorial" tiene 3 estados explícitos** (`draft`/`review`/
  `published`), en vez de reusar el default de Drupal core (`draft`/
  `published`/`archived`) — porque el flujo de contenido pedido explícitamente
  necesita una etapa de revisión separada del borrador inicial.
- **La memoria del proyecto vive en `harness/memoria/`, no en la memoria
  propia de Claude Code**: la memoria de Claude Code es del usuario (cross-
  proyecto); esta es del proyecto/plataforma y debe ser legible por
  cualquier agente o sesión futura sin depender de qué usuario la abre.
- **El planificador dimensiona cada tarea a ≤60% del contexto del agente que
  la ejecuta**, dividiendo en sub-tareas cuando el alcance es mayor. Pedido
  explícito del usuario (2026-08-18), para evitar tareas sobredimensionadas
  que degraden la calidad de ejecución o fuercen compactar contexto a medio
  camino. Ver `harness/agentes/agente-planificador.md`.
- **"Done" para el tester es 100% de los criterios de aceptación, verificado
  con más de un método** (Drush/SQL, Playwright, revisión de logs/
  regresiones, caso feliz + caso borde) — nunca "debería funcionar" ni un
  pass parcial. Pedido explícito del usuario (2026-08-18). Ver
  `harness/agentes/agente-tester.md`.
- **El orquestador es el único agente que se comunica con el usuario, en
  todos los flujos.** Ningún flujo ni agente de ejecución le presenta un
  borrador, hallazgo o pedido de aprobación al usuario por su cuenta — todo
  se reporta hacia arriba y el orquestador decide cómo comunicarlo. Puede
  investigar/razonar antes de escalar una pregunta (revisando el pedido
  original y la memoria), pero eso no reemplaza el gate humano real de la
  regla 0.3 de AGENTS.md para publicar contenido o comitear código — el
  orquestador es el canal de esa aprobación, no quien la reemplaza. Aplica
  incluso si un flujo se invoca directo sin pasar por `/dp`.
  Pedido explícito del usuario (2026-08-18). Ver
  `harness/agentes/agente-orquestador.md`.
- **El flujo de desarrollo termina en Pull Request revisado, nunca en commit
  directo a `develop`.** A pedido explícito del usuario (2026-08-18): (1)
  ninguna tarea de desarrollo termina sin commit — en cuanto el tester da
  pass, siempre se commitea, pushea y abre PR; (2) nuevo agente `revisor-pr`
  corre `phpcs` (Drupal/DrupalPractice) y `stylelint`, más revisión
  razonada, y comenta hallazgos en el PR — si hay bloqueantes, vuelve a
  desarrollo; (3) **el merge lo hace siempre el usuario, ningún agente
  ejecuta `gh pr merge` bajo ninguna circunstancia**. Ver
  `harness/agentes/agente-revisor-pr.md` y
  `harness/flujos/flujo-desarrollo-testing.md`.
- **Infraestructura conectada para lo anterior** (2026-08-18): remoto
  `git@github.com:nequeteme/danemarparceros-site.git` como `origin` de
  `site/` (con `develop`/`main`/`release_0.1` ya espejados); `gh` CLI
  instalado en `~/.local/bin/gh` (sin sudo, binario oficial — en sesión
  nueva ya está en `PATH` vía `~/.profile`); `phpcs` vía `drupal/coder`
  (composer dev dependency, config `site/phpcs.xml`); `stylelint` vía npm en
  `site/web/themes/custom/danemar_theme/` (config `.stylelintrc.json`,
  apunta solo a `css/*.css` — el sistema de diseño propio, no a
  `css/components/`, que son estilos base copiados de Drupal core/Classy).
  El proyecto no usaba SASS (`docs/plans/plan-09-estilos-sass.md`) al
  momento de instalar `stylelint` — ver más abajo la reversión de esa
  decisión. **No hay branch protection ni CI configurados en GitHub** — el
  control de que nadie mergee sin revisión es de proceso (regla del
  harness), no de plataforma, por ahora.
- **El tablero de GitHub Projects "site danemar" (#2) es la fuente de
  verdad visual del progreso, sincronizada por `project_writer`.** Cada
  fase/tarea del planificador se materializa como Issue+tarjeta (`Backlog`→
  `Ready`→`In progress`→`In review`→`Done`, labels `develop`/`contenido`).
  El tablero ya tenía la estructura de columnas heredada del template que
  usa el usuario en su otro proyecto (`layer kanban`, #1) — solo hizo falta
  crear las labels. Además funciona como entrada inversa: una tarjeta que el
  usuario cree a mano en el tablero es detectada por `project_writer` y
  entregada al orquestador como pedido de trabajo. `gh` tiene el scope
  `project` (lectura y escritura), autorizado 2026-08-18. Pedido explícito
  del usuario. Ver `harness/agentes/agente-project-writer.md`.
- **`harness/historia/` es narrativa, no memoria operativa.** El
  `documentador` registra estado (siempre); el nuevo `historiador` registra
  historias reales (bugs interesantes, decisiones revertidas, aprendizajes)
  solo cuando amerita — pensado explícitamente como insumo para un futuro
  blog de "cómo construimos esto" vía `creador-contenidos`/
  `investigador-contenidos`. Pedido explícito del usuario (2026-08-18). Ver
  `harness/agentes/agente-historiador.md`.
- **Reversión de plan-09-estilos-sass.md: el proyecto SÍ va a usar SASS.**
  A pedido explícito del usuario (2026-08-18), se decidió una migración
  completa a SASS para los 4 archivos propios del sistema de diseño
  (`tokens`, `fonts`, `base`, `sections`) — no para `css/components/`, que
  son estilos vendorizados de Drupal core/Classy, no autoría del equipo. El
  motivo original del rechazo (plan-09 v2.0: SASS rompe la auto-carga de
  SDC) no aplica tal cual a como está construido el tema real hoy: no hay
  componentes SDC propios implementados (`*.component.yml` no existe en el
  tema), los 4 archivos se declaran directo en
  `danemar_theme.libraries.yml` por ruta fija, así que un pipeline que
  compile `.scss` al mismo `.css` de siempre no pierde nada. Las custom
  properties `--dp-*` (el mecanismo real de las dos direcciones visuales
  Forge/Dossier) deben seguir siendo CSS custom properties en tiempo de
  ejecución, no convertirse en variables `$sass` (se resolverían en tiempo
  de compilación y romperían el cambio de tema en vivo). **Pendiente de
  ejecutar** — quedó interrumpida por otras tareas en la misma sesión, ver
  `memoria/corto-plazo.md`.
