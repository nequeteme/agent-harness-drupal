# Manual de uso del harness (Opción D — implementada)

Este documento explica cómo usar, en la práctica, todo lo que se implementó
de la Opción D (ver [opciones-implementacion.md](opciones-implementacion.md)
y [mapas-arquitectura.md](mapas-arquitectura.md)). Es la guía operativa; los
documentos de diseño (`agentes/`, `flujos/`, `investigacion/`) siguen siendo
la referencia de fondo.

## 1. Qué se implementó exactamente

| Pieza | Dónde vive | Qué hace |
|---|---|---|
| Workflow "Editorial" | `site/config/sync/workflows.workflow.editorial.yml` | Estados `draft` (Borrador) → `review` (En revisión) → `published` (Publicado), aplicado a `landing_page` y `legal_page`. |
| Módulos `content_moderation` + `workflows` | Core de Drupal, habilitados | Motor nativo que sostiene el workflow anterior. |
| 8 subagentes de Claude Code | `.claude/agents/*.md` | Uno por cada rol pedido (estilo, seo, creador-contenidos, investigador-contenidos, investigador-noticias, desarrollo-drupal, frontend, tester). Ver §1.3 por el noveno, `revisor-pr`. |
| 2 skills de orquestación | `.claude/skills/dp-cont.md` (`/dp-cont`), `.claude/skills/dp-dev.md` (`/dp-dev`) | Encadenan los subagentes en el orden correcto, con los gates humanos en el lugar correcto. |
| Contenido existente | Nodos 7, 9, 10, 11 (los 4 nodos del sitio) | Backfileados a `moderation_state = published` para que el workflow nuevo no rompa nada de lo ya publicado (verificado: el sitio responde 200 en `/`, `/es`, `/pt`, `/privacidad`, `/cookies`, `/terminos` tras el cambio). |

Todo esto quedó commiteado en `develop` (commit `339691a`) — ver `git log` en
`site/`.

## 1.1 Actualización: orquestador, planificador, documentador e investigador de mejoras

Se agregaron 4 agentes de coordinación sobre lo anterior — no reemplazan
nada, se suman:

| Agente | Tipo real en Claude Code | Qué hace |
|---|---|---|
| `orquestador` | Skill (`.claude/skills/dp.md`, comando `/dp`) | Punto de entrada recomendado. Habla con el usuario, delega planificación, dispatcha a los flujos/agentes, consolida reportes. |
| `planificador` | Subagente (`.claude/agents/planificador.md`) | Convierte un objetivo en un plan de tareas por agente. Invocado por el orquestador. |
| `documentador` | Subagente (`.claude/agents/documentador.md`) | Registra en `harness/memoria/` (corto y largo plazo) qué se hizo en cada tarea/flujo. |
| `investigador-mejoras-harness` | Subagente (`.claude/agents/investigador-mejoras-harness.md`) | Investiga cómo mejorar el harness mismo; informe cada ~2 días en `harness/mejoras/`. |

**Por qué el orquestador es una skill y no un subagente**: en Claude Code un
subagente no puede invocar a otros subagentes — solo la sesión principal
puede. El orquestador necesita dispatchar a varios agentes, así que tiene que
gobernar la sesión principal directamente (igual que ya hacían `/dp-cont` y
`/dp-dev`). Ver `harness/agentes/agente-orquestador.md`.

**Cómo se usa en la práctica**: en vez de acordarte de qué skill o agente
invocar, ahora puedes simplemente hablar con Claude Code de forma normal
("necesito una FAQ nueva sobre X", "¿cómo va el pendiente de SEO?") y dejar
que actúe como orquestador — o invocarlo explícito con `/dp` si quieres
forzar ese modo. Él decide si hace falta planificar
(`planificador`), qué flujo/agente ejecutar, y te da un resumen consolidado
al final, no una descarga de reportes crudos de cada agente.

## 1.2 Reglas de calidad: tamaño de tareas y Definición de Done

Dos reglas más, a pedido explícito del usuario, ya reflejadas en el diseño y
en el runtime:

- **El planificador no dimensiona ninguna tarea por encima del ~60% del
  contexto** del agente que la va a ejecutar. Si un objetivo es grande, lo
  vas a ver dividido en varias tareas secuenciales más chicas en vez de una
  sola tarea gigante — es deliberado, no un límite técnico que se esté
  midiendo en vivo. Detalle: `harness/agentes/agente-planificador.md`.
- **Definición de "Done" = 100%, verificado por el `tester` con más de un
  método** (Drush/SQL, Playwright, revisión de logs/regresiones, caso feliz +
  caso borde). No existe un "pass parcial": si un solo criterio de
  aceptación no se cumple al 100%, es fail completo y vuelve a
  implementación, sin excepción. Vas a ver esto reflejado en los reportes
  del flujo de desarrollo — el `tester` cita evidencia de cada método que
  usó, no solo "debería funcionar". Detalle: `harness/agentes/agente-tester.md`.

## 1.3 Actualización (2026-08-18): branches + PR revisado, nunca commit directo

Tres cambios más, a pedido explícito del usuario:

1. **Ninguna tarea de desarrollo termina sin commit.** En cuanto el `tester`
   da pass, el agente que implementó siempre commitea, pushea y abre un PR —
   nunca se queda una rama verificada sin subir.
2. **El flujo de desarrollo ya no comitea directo a `develop`.** Ahora
   siempre es: rama de trabajo → `tester` → commit + push + `gh pr create`
   contra `develop` → **`revisor-pr`** (noveno subagente, `.claude/agents/revisor-pr.md`)
   → si hay hallazgos, vuelve a desarrollo; si no, el orquestador te avisa
   que el PR está listo.
3. **El merge lo hacés siempre vos, nunca un agente.** Ni el `revisor-pr` ni
   el orquestador ejecutan `gh pr merge` bajo ninguna circunstancia, aunque
   el PR esté perfecto.

**Infraestructura conectada para esto** (todo real, no simulado):
- Remoto: `git@github.com:nequeteme/danemarparceros-site.git`, configurado
  como `origin` en `site/`. `develop`, `main` y `release_0.1` ya están
  espejados ahí.
- `gh` CLI instalado en `~/.local/bin/gh` (no requirió `sudo` — se bajó el
  binario oficial). En una sesión nueva de terminal ya está en el `PATH` vía
  `~/.profile`; si hace falta usarlo manual en esta misma sesión, con
  `export PATH="$HOME/.local/bin:$PATH"` alcanza.
- `phpcs` (con estándares `Drupal`+`DrupalPractice`, vía `drupal/coder`)
  instalado como dependencia de desarrollo de Composer, config en
  `site/phpcs.xml` (apunta a `web/modules/custom` y `web/themes/custom`).
- `stylelint` instalado como dependencia npm del tema, config en
  `site/web/themes/custom/danemar_theme/.stylelintrc.json` (`npm run
  lint:css`, sobre `css/*.css` — el sistema de diseño propio, no sobre
  `css/components/`, que son estilos base copiados de Drupal core/Classy).
  El proyecto **no usa SASS** (decisión ya documentada en
  `docs/plans/plan-09-estilos-sass.md`), así que no hay nada de `.scss` que
  lintear.

Detalle completo del flujo: `harness/flujos/flujo-desarrollo-testing.md` y
`harness/agentes/agente-revisor-pr.md`.

## 1.4 Actualización (2026-08-18): tablero de GitHub Projects + historia

Dos agentes más:

- **`project_writer`** (`.claude/agents/project_writer.md`): crea una
  tarjeta en el tablero **[site danemar](https://github.com/users/nequeteme/projects/2)**
  por cada fase/tarea del planificador (etiqueta `develop` o `contenido`,
  `Status` empieza en `Backlog`) y la va moviendo (`Ready` → `In progress` →
  `In review` → `Done`) a medida que el flujo avanza — así podés ver el
  progreso real **desde GitHub**, sin preguntarle a Claude Code. Además,
  funciona al revés: si vos creás una tarjeta directo en el tablero, la
  detecta y se la entrega al orquestador como un pedido de trabajo nuevo —
  el tablero es una segunda puerta de entrada al harness. El tablero ya
  tenía la estructura (`Backlog`/`Ready`/`In progress`/`In review`/`Done`,
  `Priority`, `Size`) heredada del mismo template que usás en tu proyecto
  `layer kanban`; solo hizo falta crear las labels `develop`/`contenido` en
  el repo.
- **`historiador`** (`.claude/agents/historiador.md`): registra en
  `harness/historia/` — no en `harness/memoria/` — las historias reales
  (bugs con causa interesante, decisiones revertidas, aprendizajes), para
  que sirvan de insumo a un blog futuro de `creador-contenidos`. No registra
  todo, solo lo que amerita (ver criterio en `agente-historiador.md`). Ya
  hay una primera entrada real:
  `harness/historia/2026-08-18-el-banner-de-cookies-que-se-transparentaba.md`.

**Infraestructura conectada**: `gh` ya tiene el scope `project` (lectura y
escritura de GitHub Projects), autorizado por vos vía device flow. Repos
confirmados con acceso real: los dos proyectos de tu cuenta
(`site danemar` #2, `layer kanban` #1).

Detalle completo: `harness/agentes/agente-project-writer.md`,
`harness/agentes/agente-historiador.md`,
`harness/flujos/flujo-orquestacion.md`.

## 2. Cómo disparar un flujo completo

Los flujos son **skills** — se invocan como slash command, escritos en
lenguaje natural describiendo la tarea:

```
/dp-cont escribe una nueva pregunta de FAQ sobre accesibilidad
en portales institucionales, basada en investigación reciente
```

```
/dp-dev agrega el campo Service (JSON-LD) faltante
a service_card, es uno de los pendientes de plan-13
```

También puedes simplemente describir la tarea sin el slash y Claude Code
elegirá el flujo si la descripción calza (por ejemplo, "quiero contenido
nuevo para la sección de servicios, basado en lo que está pasando en el
sector" activará `/dp-cont` solo).

**Qué vas a ver mientras corre**: cada paso del flujo invoca a un subagente
distinto (verás el nombre en la salida — p. ej. "investigador-contenidos",
luego "creador-contenidos", luego "estilo", luego "seo"). Al final del flujo
de contenido, se te va a **pedir aprobación explícita** antes de transicionar
algo a `published` — sin un "sí, publícalo" tuyo, se queda en `draft`/
`review`. Al final del flujo de desarrollo, vas a ver el reporte del
`tester`, después el del `revisor-pr` (con el PR ya abierto y comentado), y
se te va a avisar cuando esté listo para que **vos** hagas el merge — ningún
agente comitea a `develop` ni mergea el PR (ver §1.3).

**Quién te hace esa pregunta**: siempre el orquestador, nunca el flujo en
bruto — aunque hayas invocado `/dp-cont` o `/dp-dev` directo sin pasar por
`/dp` primero. Es una regla explícita del proyecto: toda la comunicación con vos está
centralizada en el orquestador, para que sea siempre el mismo criterio el
que decide cómo presentarte algo y cuándo pedirte una aprobación (ver
"Regla de comunicación centralizada" en
`harness/agentes/agente-orquestador.md`).

## 3. Cómo invocar un solo agente (sin correr el flujo completo)

Útil cuando no necesitas el pipeline entero — por ejemplo, solo una
auditoría SEO puntual:

```
usa el agente seo para auditar los metadatos de la portada
```

```
que el agente tester verifique el cambio que acabo de pedir en el menú móvil
```

Claude Code elige el subagente correcto por su `description` (ver cada
archivo en `.claude/agents/`), pero nombrarlo explícitamente ("usa el agente
X") es la forma más confiable de forzar cuál se usa.

## 4. Cómo funciona el estado de moderación (`draft` → `review` → `published`)

Desde ahora, cualquier nodo `landing_page`/`legal_page` tiene un campo
**Moderation state** en su formulario de edición (`/node/{id}/edit` en el
admin de Drupal). Los agentes de contenido dejan sus borradores en `draft` o
`review` — nunca publican directo.

**Como humano, para revisar y publicar un borrador tienes dos caminos:**

1. **Desde la UI de Drupal**: entra a `/admin/content` (o
   `/admin/content/moderated` para ver solo lo pendiente de revisión),
   abre el nodo, cambia el desplegable de estado de moderación a
   `Publicado` y guarda.
2. **Pidiéndoselo a Claude Code**: "aprueba y publica el borrador del nodo
   X" — pero solo hazlo cuando de verdad lo revisaste; el agente no te va a
   forzar a mirarlo primero.

Las transiciones válidas son: `draft → review`, `review → draft` (pedir
cambios), `review → published`, `draft → published` (vía "crear nuevo
borrador" seguido de publicar directo, para casos urgentes), y volver a
`draft` desde cualquier estado para retomar edición.

## 4.1 Memoria del proyecto (`harness/memoria/`)

El `documentador` mantiene dos niveles:

- `harness/memoria/corto-plazo.md` — qué se hizo en las últimas tareas, qué
  quedó pendiente. Se actualiza después de cada flujo/tarea.
- `harness/memoria/largo-plazo/*.md` — conocimiento consolidado por tema
  (empezó con `decisiones-arquitectura.md`; el documentador va a ir creando
  más archivos ahí según haga falta: `contenido.md`, `seo.md`, etc.).

No necesitas pedir explícitamente que se actualice — el orquestador invoca al
documentador al cierre de cada flujo. Si quieres consultar el estado sin
disparar una tarea nueva, puedes pedir directo "¿qué dice la memoria de
corto plazo?" o leer los archivos vos mismo, son Markdown plano.

## 4.2 Informe de mejoras del harness (cada ~2 días)

**Decisión actual: disparo manual, no programado.** El
`investigador-mejoras-harness` está diseñado para una cadencia de ~2 días,
pero por ahora corre solo cuando se lo pedís explícitamente — no hay `/loop`
ni rutina de cron (`schedule`) configurada. Invocalo con algo como "que el
investigador de mejoras del harness genere su informe ahora".

Si más adelante lo querés automático, las dos formas siguen disponibles y
documentadas en `harness/agentes/agente-investigador-mejoras-harness.md`
(`/loop`, atado a una sesión activa; o una rutina de cron real vía la skill
`schedule`, que sí corre sin sesión abierta pero es una automatización
recurrente con consumo continuo — pedímelo cuando quieras dar ese paso).

## 5. Recomendaciones para usarlo a máximo potencial

- **No corras `/dp-cont` a ciegas por rutina.** El sitio es una
  landing de una página, no un blog — genera contenido cuando haya una razón
  real (un gap de SEO detectado, una noticia relevante, un pedido concreto),
  no para "tener actividad". El propio agente `investigador-noticias` está
  instruido para filtrar ruido, pero la decisión final de generar contenido
  es tuya.
- **Usa `/loop` para vigilancia periódica de noticias** si quieres que
  `investigador-noticias` corra con cadencia regular dentro de una sesión
  activa (por ejemplo, `/loop 1d investigador-noticias: revisa novedades del
  sector`). Recuerda que esto solo corre mientras la sesión existe — no es
  autonomía 24/7 sin supervisión (esa es la diferencia con la Opción B, ver
  [opciones-implementacion.md](opciones-implementacion.md)).
- **Mantén `harness/agentes/*.md` como fuente de verdad** de cada rol. Si
  cambias cómo quieres que trabaje un agente, edita el `.md` correspondiente
  en `harness/agentes/` primero y luego refleja el cambio en
  `.claude/agents/*.md` (el subagente) — así ambos quedan sincronizados.
- **El gate humano no es negociable para publicar/mergear** — está así por
  diseño (regla 0.3 de `AGENTS.md`), no lo saltees pidiéndole a un agente que
  "publique directo" o "mergee el PR" — ningún agente lo va a hacer, ni
  aunque insistas, es una regla dura.
- **Si un reporte del `tester` dice "fail", no le pidas que lo redondee a
  pass** — es al 100% o no está done (ver §1.2). Un fail bien reportado con
  evidencia de varios métodos es más útil que un pass optimista.
- **Si una tarea se ve enorme al pedirla, esperá que el planificador la
  parta en varias** (regla del 60%, §1.2) — no es que el harness sea lento,
  es que prefiere varias tareas chicas bien verificadas a una grande a medio
  verificar.
- **Para contenido nuevo que no encaje en los 14 Paragraph types actuales**
  (p. ej. si algún día quieres un blog/sección de insights), eso es una
  tarea de `desarrollo-drupal` primero (crear el content type/Paragraph
  type), no algo que `creador-contenidos` deba improvisar.

## 6. Verificación rápida de que todo sigue sano

```bash
cd site
ddev drush php:eval "print_r(\Drupal::entityTypeManager()->getStorage('workflow')->load('editorial')->getTypeSettings());"
```

Debe mostrar los 3 estados (`draft`, `review`, `published`) y los bundles
`landing_page`/`legal_page`. Para ver el contenido pendiente de revisión:
`/admin/content/moderated` en el navegador (contra
`https://site-dev.danemarparceros.net/admin/content/moderated`, levantando el
túnel si no responde — ver `AGENTS.md` 0.4).

## 7. Qué NO cambió (límites reales de esta implementación)

- Nadie fuera de una sesión de Claude Code puede disparar un agente — un
  editor sin acceso a Claude Code puede **revisar y publicar** borradores
  (paso 4 arriba) pero no generar contenido nuevo por sí solo. Ese salto es
  la Opción B, documentada como camino de evolución.
- No hay JSON:API/REST habilitado — toda la automatización sigue siendo vía
  Drush scripting dentro de DDEV, igual que en los 13 planes previos del
  proyecto.
- No se instaló ningún módulo de IA de Drupal (`ai`, `ai_agents`,
  `mcp_server`) — los agentes viven enteramente en `.claude/`, no dentro de
  Drupal.
- El merge de PRs sigue siendo 100% manual — no hay branch protection ni CI
  configurados en GitHub todavía (el repo se conectó recién en esta sesión),
  así que hoy nada impide técnicamente mergear un PR sin que `revisor-pr`
  haya opinado; el control es de proceso (regla del harness), no de
  plataforma. Si en algún momento se quiere un control duro a nivel GitHub,
  eso es configuración de branch protection — no forma parte de esta
  implementación todavía.

## 8. Mapa de archivos de esta implementación

```
danemar_site/
├── .claude/
│   ├── agents/            ← 14 subagentes: los 8 de ejecución + revisor-pr +
│   │                          planificador, documentador,
│   │                          investigador-mejoras-harness,
│   │                          project_writer, historiador
│   └── skills/             ← 3 skills: dp-cont, dp-dev, dp
│                              (comandos /dp-cont, /dp-dev, /dp)
├── harness/
│   ├── README.md           ← índice general
│   ├── MANUAL-DE-USO.md    ← este documento
│   ├── opciones-implementacion.md
│   ├── mapas-arquitectura.md
│   ├── agentes/             ← especificación de diseño de cada agente (fuente de verdad)
│   ├── flujos/               ← especificación de diseño de cada flujo (incluye flujo-orquestacion.md)
│   ├── memoria/               ← corto-plazo.md + largo-plazo/ (project-tracker.md,
│   │                             roadmap.md, decisiones-arquitectura.md)
│   ├── historia/               ← narrativa real, mantenida por el historiador
│   ├── mejoras/                ← informes del investigador-mejoras-harness
│   ├── investigacion/        ← research de fondo
│   └── analisis-proyecto/    ← foto del proyecto al momento del diseño
└── site/                    ← repo git real, remoto en
                                git@github.com:nequeteme/danemarparceros-site.git
    ├── phpcs.xml             ← estándares Drupal/DrupalPractice (revisor-pr)
    └── web/themes/custom/danemar_theme/
        ├── package.json + .stylelintrc.json  ← lint de CSS (revisor-pr)
        └── css/sections.css                  ← fix del banner de cookies
                                                 (PR #2, ver §1.3)
```

**Tablero en vivo**: https://github.com/users/nequeteme/projects/2
("site danemar").
