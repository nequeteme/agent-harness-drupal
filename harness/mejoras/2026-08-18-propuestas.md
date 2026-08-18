# Propuestas de mejora del harness — 2026-08-18

Foco de este informe (pedido puntual del usuario): "¿cada agente necesita
skills y funcionarían mejor con sus propias skills?" — investigación dirigida
a esa pregunta específica, con conclusión explícita, no solo un mapeo de
opciones.

> **Actualización 1 (misma fecha, segunda pasada):** el usuario pidió ir más
> al fondo antes de quedarse con las propuestas puntuales de la primera
> pasada: ¿esta arquitectura concreta *necesita* skills, de qué forma exacta
> funcionan (no solo en teoría), y eso es una mejora de "código" o de
> "funcionamiento"? La respuesta completa está en la
> **[Sección 0](#0-segunda-pasada-restricción-técnica-código-vs-funcionamiento-y-recomendación-única)**.
> Corrige/precisa (no invalida) las 4 propuestas de las secciones 1-5.
>
> **Actualización 2 (misma fecha, tercera pasada):** el usuario planteó una
> visión más amplia — catálogo de skills por dominio, quién las instala,
> escalamiento "agente atascado → pide skill", catálogo propio de patrones
> que crece con el tiempo, y métricas de éxito/rollback — como un ciclo de
> mejora y especialización continua. Diseño completo en la nueva
> **[Sección 6](#6-ciclo-de-mejora-continua-y-especialización-tercer-pedido-del-usuario)**.
>
> **Actualización 3 (misma fecha, cuarta pasada):** antes de comitear
> `harness/` + `.claude/` como repo aparte ("versión alpha" de esta
> arquitectura), el usuario pidió una auditoría en dos partes: (A) si lo que
> el harness describe sobre cómo trabajar con Drupal es correcto/coherente
> (agentes `desarrollo-drupal`, `frontend`, `tester`, `seo`), usando las 5
> skills de Drupal recién instaladas como fuente de verificación cruzada; y
> (B) qué está hardcodeado a este proyecto puntual (danemarparceros) y
> bloquearía reusar el harness en otro proyecto Drupal. Incluye un hallazgo
> importante: una inconsistencia factual real en la propia memoria del
> harness sobre si el tema usa SDC de verdad. Ver la nueva
> **[Sección 7](#7-auditoría-de-compatibilidad-drupal-y-portabilidad-previa-a-comitear-como-repo-aparte)**.

## Conclusión en una frase

**No, no cada uno de los 13 agentes necesita una Skill propia**, y además —
esto es la corrección importante de la segunda pasada— **ningún subagente de
este harness puede invocar el tool `Skill` en tiempo de ejecución hoy** (está
verificado, no es una suposición): los 13 archivos de `.claude/agents/*.md`
declaran una lista explícita de `tools` que nunca incluye `Skill` ni `Agent`.
Esto significa que "darle una skill a un agente" en esta arquitectura **no
puede** significar "el subagente decide en tiempo real invocar `/nombre`" —
la única forma que funciona de verdad es la **precarga estática** (campo
`skills:` en el frontmatter del subagente, que inyecta el contenido completo
de la Skill en su contexto al arrancar, sin que el subagente "la invoque").
Eso convierte a "agregar una skill" en algo que es **mayormente código/config**
(mover texto duplicado a un archivo compartido y referenciarlo desde 2+
subagentes) con un beneficio de **funcionamiento real pero acotado**: una
sola fuente de verdad para conocimiento compartido por más de un agente, y la
posibilidad de que el usuario/orquestador invoquen ese mismo archivo como
comando suelto (`/nombre`) sin levantar todo el subagente. No cambia cómo
razona el subagente, no reduce su consumo de contexto (la precarga inyecta
todo igual, sin *progressive disclosure*), y no le da ninguna capacidad
nueva. Ver sección 0 para el desarrollo completo, y sección 6 para el diseño
del ciclo de mejora/especialización continua con catálogos de skills reales
por dominio.

---

## 0. Segunda pasada: restricción técnica, código vs. funcionamiento, y recomendación única

### 0.1 Restricción técnica confirmada: ningún subagente de este harness puede invocar `Skill`

`harness/agentes/agente-orquestador.md` ya documenta que "en Claude Code, un
subagente no puede invocar a otros subagentes — solo la sesión principal
tiene esa capacidad", y por eso el orquestador vive como Skill de la sesión
principal (`.claude/skills/dp.md`), no como subagente anidado. La pregunta
pendiente era si esa misma restricción (o una parecida) aplica al tool
`Skill` cuando quien lo invocaría es un subagente. Respuesta, con evidencia
en dos niveles:

**Nivel 1 — qué dice la documentación oficial sobre el default.** Según
[code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents):
si el campo `tools` se omite en el frontmatter de un subagente, hereda todas
las herramientas disponibles para subagentes, "narrowed by two filters". El
primer filtro aplica siempre (en foreground y background) y es el que excluye
`Agent` para subagentes anidados salvo casos puntuales ("Apart from `Agent`
and `ExitPlanMode`, which follow the first filter's conditions wherever the
subagent runs"). El segundo filtro, que solo aplica a subagentes corriendo en
**background**, reduce el set aún más a una lista fija que sí incluye
`Skill`: *"a background subagent keeps every MCP tool but only these
built-in tools: Read, Grep, Glob, Bash, PowerShell, Edit, Write,
NotebookEdit, WebFetch, WebSearch, TodoWrite, **Skill**, ToolSearch,
EnterWorktree, ExitWorktree, Monitor, TaskStop, SendMessage, and Artifact"*.
Es decir: en teoría, un subagente en background con `tools` omitido *podría*
tener `Skill` disponible por default.

**Nivel 2 — qué pasa en este repo, en la práctica (lo que realmente importa
para la decisión).** Los 13 subagentes de este harness **no** omiten `tools`
— cada uno declara una lista explícita y acotada en su frontmatter (revisado
directamente en `.claude/agents/*.md`, 13 de 13 archivos confirmados con el
mismo patrón):

| Subagente | `tools` declarado (línea 4 de su `.claude/agents/*.md`) |
|---|---|
| `tester` | `Bash, Read, Grep, Glob` |
| `revisor-pr` | `Bash, Read, Grep, Glob` |
| `estilo` | `Bash, Read, Grep, Glob` |
| `seo` | `Bash, Read, Grep, Glob, WebFetch` |
| `desarrollo-drupal` | `Bash, Read, Edit, Write, Grep, Glob` |
| `frontend` | `Bash, Read, Edit, Write, Grep, Glob` |
| `planificador` | `Read, Grep, Glob, Bash` |
| `documentador` | `Read, Write, Edit, Grep, Glob, WebSearch, WebFetch` |
| `project_writer` | `Bash, Read, Write, Grep, Glob` |
| `creador-contenidos` | `Bash, Read, Grep, Glob` |
| `investigador-contenidos` | `WebSearch, WebFetch, Read` |
| `investigador-noticias` | `WebSearch, WebFetch, Read` |
| `historiador` | `Read, Write, Grep, Glob` |
| `investigador-mejoras-harness` (este agente) | `WebSearch, WebFetch, Read, Write` — **sin `Bash`** (ver sección 6.2) |

Ninguna de las 13 listas incluye `Skill` (ni `Agent`, consistente con lo que
`agente-orquestador.md` ya documentaba). Como `tools` explícito es un
allowlist estricto — según la misma documentación, *"To prevent a subagent
from invoking skills entirely, omit `Skill` from the `tools` list"* —, esto
no es ambigüedad de defaults: **está decidido y en vigor hoy** que ningún
subagente de este harness puede llamar al tool `Skill`. Fuente:
[code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents).

**Confirmación adicional de por qué esto es lo correcto, no solo un
accidente de configuración.** Hay un caveat documentado (no oficialmente
confirmado por Anthropic, issue cerrado como "not planned", pero con
reproducción técnica concreta) de que cuando una Skill invocada *desde
dentro* de un subagente necesita a su vez despachar más llamadas a `Agent`
(p. ej. una skill que orquesta sub-verificaciones), esas llamadas anidadas
fallan silenciosamente: en vez de spawnear un subagente independiente, el
propio modelo completa el rol inline, produciendo una falsa sensación de
verificación cruzada sin independencia real de contexto. Fuente:
[GitHub — anthropics/claude-code issue #59968](https://github.com/anthropics/claude-code/issues/59968).
Ninguna de las skills propuestas en este informe necesita despachar agentes
desde adentro (son contenido de referencia/checklist, no orquestación), así
que esto no es un bloqueante para las propuestas 1-4 ni para el catálogo de
la sección 6 — pero sí es una razón más, alineada con el mismo principio que
ya aplica `agente-orquestador.md`, para **no** habilitar `Skill` en el
`tools` de ningún subagente de ejecución de este harness: la arquitectura ya
está diseñada around la idea de que solo la sesión principal "alcanza" hacia
afuera (a otros subagentes, y por extensión a skills invocables
dinámicamente); los subagentes reciben todo lo que necesitan de antemano y
trabajan de forma autocontenida.

### 0.2 Qué mecanismo sí funciona: precarga (`skills:`), no invocación (`tools: Skill`)

La documentación distingue explícitamente estos dos mecanismos, y son
técnicamente independientes entre sí:

| | Campo | Qué hace | Requiere `Skill` en `tools` |
|---|---|---|---|
| **Precarga** | `skills:` en el frontmatter del subagente | Inyecta el **contenido completo** de cada Skill listada en el contexto del subagente **al arrancar** — el subagente nunca "decide" cargarla, ya está ahí. | **No** — es un mecanismo separado, deliberadamente distinto: *"To preload Skills into context, use the `skills` field rather than listing `Skill` here"*. |
| **Invocación en runtime** | `tools: [..., Skill]` | El subagente puede, durante su ejecución, decidir invocar `/nombre` como cualquier otra herramienta (descubrimiento dinámico). | Sí, imprescindible. |

Fuente: [code.claude.com/docs/en/sub-agents — "Preload skills into
subagents"](https://code.claude.com/docs/en/sub-agents).

Dado 0.1, la única vía viable en este harness para "un subagente + una
skill" es la **precarga** — que, técnicamente, es casi equivalente a escribir
ese mismo texto directamente en el cuerpo del `.claude/agents/<nombre>.md`
del subagente: la tabla comparativa de la documentación oficial dice
textualmente que con `skills:` "the full skill content is injected at
startup", igual costo de contexto que si estuviera inline. **Esto corrige
una implicancia de la Sección 2 del informe original**, que mencionaba
"progressive disclosure" (carga solo cuando se dispara, ahorro de contexto)
como un beneficio general de extraer una Skill — ese beneficio es real
**solo para quien invoca la Skill por comando** (`/nombre`, típicamente desde
la sesión principal/orquestador), **no** para el contenido que un subagente
precarga, que se paga completo en cada invocación del subagente igual que
si estuviera en su propio prompt.

### 0.3 Código vs. funcionamiento — respuesta explícita a lo que pidió el usuario

**Es, en lo esencial, un cambio de código, con un beneficio de funcionamiento
real pero acotado a dos efectos concretos — no una mejora general de cómo
razonan o colaboran los agentes.**

- **Lo que es "código"** (mecánico, de configuración, sin cambiar
  comportamiento): crear `.claude/skills/<nombre>/SKILL.md`, mover ahí texto
  que hoy vive dentro de uno o más `.claude/agents/*.md`, y agregar el campo
  `skills: [<nombre>]` al frontmatter de cada subagente que lo necesite. No
  toca `tools:` (no hace falta, y no correspondería agregar `Skill` ahí, ver
  0.1). El subagente termina viendo, en tiempo de ejecución, exactamente el
  mismo texto que ya veía — solo que ahora ese texto vive en un archivo
  aparte en vez de estar pegado en su propio prompt.
- **Lo que sí es "funcionamiento" real** (y por eso vale la pena en los
  casos puntuales de la sección 4, no en general):
  1. **Una sola fuente de verdad cuando 2+ agentes necesitan el mismo
     conocimiento.** Hoy el checklist de phpcs/stylelint está escrito tres
     veces (`revisor-pr`, `desarrollo-drupal`, `frontend` — ver sección 4,
     Propuesta 2) con instrucciones ligeramente distintas cada vez. Eso *ya*
     es una duplicación real, con riesgo real de que diverjan si se
     actualiza una copia y no las otras. Consolidar en un archivo y
     precargarlo en los tres es una mejora de funcionamiento genuina
     (consistencia a lo largo del tiempo), no solo prolijidad.
  2. **Invocación directa desde la sesión principal sin levantar el
     subagente completo.** El mismo archivo que un subagente precarga puede,
     además, quedar invocable con `/nombre` por el usuario o por el
     orquestador (que sí corre en la sesión principal y sí tiene el tool
     `Skill` disponible sin restricción) — eso sí es una capacidad nueva que
     hoy no existe: hoy, para chequear "¿esto cumple el checklist de SEO?"
     hay que invocar todo el subagente `seo`; con la skill extraída, se
     puede correr `/seo-checklist-drupal` directo en la sesión principal.
- **Lo que NO cambia** (para ser honesto sobre el límite real de esta
  mejora): el subagente no razona distinto, no gana ninguna herramienta
  nueva, no consume menos contexto (0.2), y no puede decidir por sí mismo
  cargar una skill adicional a mitad de tarea — todo lo que usa está fijado
  al momento en que se lo invoca, igual que hoy.

### 0.4 Patrón de arquitectura que corresponde a este harness (no genérico — específico a esta combinación orquestador-en-sesión-principal + subagentes acotados)

De la documentación oficial y de guías de terceros consultadas (ninguna
describe un patrón productivo de "subagente descubre y decide invocar skills
en runtime" para workers aislados con tools restringidas — el patrón
documentado y usado en ejemplos reales es siempre uno de estos dos, y ambos
ya encajan con el diseño existente de este harness):

1. **Skills a nivel de sesión principal** (`context` normal, sin `fork`,
   o con `context: fork` cuando conviene aislar la ejecución): invocadas por
   el usuario o por quien gobierna la sesión principal — en este harness,
   eso ya es exactamente el rol de `/dp`/`/dp-cont`/`/dp-dev`. Cualquier
   skill nueva pensada para invocarse dinámicamente (no solo precargarse)
   pertenece a este nivel, nunca a uno de los 13 subagentes de ejecución.
2. **Skills precargadas en subagentes** (`skills:` en el frontmatter):
   contenido de referencia fijo, sin necesidad de descubrimiento en runtime
   — el único patrón que aplica a los 13 subagentes de ejecución de este
   harness.

Esto es, en el fondo, **la misma regla que ya justifica que el orquestador
sea una skill de sesión principal y no un subagente anidado** (`agente-orquestador.md`,
"Nota técnica importante"), extendida por simetría de `Agent` a `Skill`: en
esta arquitectura, **solo la sesión principal "alcanza" activamente hacia
afuera** (a otros subagentes, o a skills invocadas dinámicamente); los 13
subagentes de ejecución reciben todo lo que necesitan de antemano
(instrucciones propias + tools acotadas + skills precargadas) y trabajan de
forma autocontenida, devolviendo un resultado. No hay que inventar un patrón
nuevo — hay que aplicar el que el propio harness ya adoptó para el problema
gemelo (`Agent`), y del que la restricción de `Skill` (0.1) es, en la
práctica, un caso más.

### 0.5 Recomendación única

**Sí, tiene sentido el patrón "agente + skill dedicada" en esta arquitectura,
pero en una forma exacta y acotada, no como default para los 13 agentes:**
extraer a `.claude/skills/<nombre>/SKILL.md` únicamente el conocimiento que
hoy está (o quedaría, si no se actúa) duplicado entre 2 o más de los 13
`.claude/agents/*.md`, referenciarlo con el campo `skills:` (nunca
`tools: Skill`) en cada subagente que lo necesite, y dejarlo también
invocable con `/nombre` desde la sesión principal para uso directo del
usuario/orquestador. Es, ante todo, una mejora de **código/config** (mover
texto duplicado a una fuente única); el beneficio de **funcionamiento** que
la justifica es puntual y real —evitar drift entre archivos que ya divergen
hoy, más un atajo de invocación directa sin pasar por el subagente
completo— no una mejora general de cómo razonan o colaboran los 13 agentes.
Fuera de esos casos puntuales (sección 4), no hay beneficio real en darle
una skill a cada agente, y el propio mecanismo técnico de este harness
(subagentes sin `Skill` en su `tools`, por diseño, igual que no tienen
`Agent`) ya impide que funcione como invocación dinámica aunque se quisiera.

---

## 1. Qué es una Skill y cómo se relaciona con un subagente (no son lo mismo, son complementarios)

Según la documentación oficial de Anthropic:

- **Skill** = un directorio en el filesystem con un `SKILL.md` (instrucciones
  + metadata YAML) y, opcionalmente, scripts/archivos de referencia. Se
  carga con **"progressive disclosure"** en tres niveles: (1) metadata
  (`name`+`description`, ~100 tokens) siempre presente en el system prompt;
  (2) el cuerpo de `SKILL.md` (<5k tokens) solo cuando la Skill se dispara,
  sea porque el usuario escribe `/nombre` o porque Claude decide que la
  `description` matchea el pedido; (3) archivos/scripts adicionales, leídos
  solo si se referencian. Vive en `.claude/skills/<nombre>/SKILL.md`
  (proyecto) o `~/.claude/skills/` (personal). **Nota (sección 0.2): esta
  carga progresiva en 3 niveles aplica a quien invoca la skill como comando
  — no al mecanismo de precarga en un subagente, que inyecta todo de una
  sola vez.**
- **Subagente** (`.claude/agents/*.md`) = un worker con **ventana de
  contexto propia**, su propio set de herramientas y su propio modelo,
  invocado por el agente principal (o por otro agente) vía la herramienta
  `Agent`/`Task`. El que lo invoca solo recibe el resumen final — el
  propósito central es **aislamiento**, no reutilización de conocimiento.

Estos dos mecanismos se combinan explícitamente en Claude Code, de dos
formas documentadas:

1. **Un subagente puede "precargar" Skills** con el campo `skills:` en su
   frontmatter — el contenido completo de cada Skill listada se inyecta en
   el contexto del subagente al arrancar (no solo la descripción), **sin
   que el subagente necesite el tool `Skill` habilitado** (ver sección 0.1
   y 0.2 — esta es la corrección clave de la segunda pasada de este
   informe). Fuente:
   [code.claude.com/docs/en/sub-agents — "Preload skills into subagents"](https://code.claude.com/docs/en/sub-agents).
2. **Una Skill puede correr como subagente** con `context: fork` en su
   frontmatter — el cuerpo de la Skill se convierte en la tarea que recibe
   un subagente aislado (built-in como `Explore`/`Plan`, o uno custom de
   `.claude/agents/`). Este camino corre a nivel de sesión principal (quien
   invoca la skill), no es algo que uno de los 13 subagentes de ejecución
   pueda disparar por sí mismo (ver 0.1). Fuente:
   [code.claude.com/docs/en/skills — "Run skills in a subagent"](https://code.claude.com/docs/en/skills).

Cita textual de la comparación oficial (`skills-explained`, blog de
Anthropic): *"Skills say 'here's how to do things' and provide capabilities
that work everywhere — any conversation, any project"*, mientras que un
subagente es *"a worker Claude spawns"* con su propio contexto — la
distinción no es "qué agente debería tener una Skill" sino **"qué
conocimiento vale la pena sacar del prompt de un agente y convertir en un
archivo aparte, reusable e invocable independientemente"**.

Este harness ya tiene esto bien resuelto a nivel de capa: los 13 roles son
subagentes (decisión correcta — cada uno necesita aislamiento de contexto y
permisos de herramientas distintos, ver `harness/opciones-implementacion.md`
Opción A/D) y las 3 Skills existentes (`dp`, `dp-cont`, `dp-dev`) son
correctamente los **flujos de orquestación**, a nivel de sesión principal —
exactamente el único nivel donde, según la sección 0, una skill invocable
dinámicamente puede funcionar en esta arquitectura.

## 2. Cuándo empaquetar el conocimiento de un rol como Skill (criterio, no caso por caso todavía)

De la documentación oficial (`platform.claude.com/docs/en/agents-and-tools/agent-skills/overview`,
`code.claude.com/docs/en/skills`, `code.claude.com/docs/en/sub-agents`) y del
blog `skills-explained`, el criterio consistente es:

**Extraer a una Skill dedicada cuando:**
- El conocimiento lo necesita **más de un agente/rol** (evita duplicar la
  misma guía en varios `.claude/agents/*.md`, con el riesgo de que diverjan).
- Tiene sentido que el **usuario la invoque directamente** con `/nombre`,
  sin pasar por todo el flujo del subagente que la usa (recordar: eso solo
  funciona bien desde la sesión principal, sección 0.1).
- Es lo bastante grande/estable como para beneficiarse de **progressive
  disclosure** *cuando se invoca como comando desde la sesión principal*
  (archivos de referencia que no cuestan contexto hasta que se leen) o de
  **scripts ejecutables** (el código no entra al contexto, solo su output) —
  este beneficio no aplica al contenido que un subagente solo precarga
  (sección 0.2).
- Es **portable** — útil fuera de este subagente puntual, incluso fuera de
  este proyecto.

**Dejarlo en el prompt del subagente (`.claude/agents/<nombre>.md`) cuando:**
- Es conocimiento **único de ese rol**, sin un segundo consumidor real hoy.
- Es breve y ya está bien delimitado — convertirlo en Skill agrega
  indirección (un archivo más que mantener sincronizado) sin ganar nada,
  porque de todas formas se carga completo apenas se invoca el subagente
  (no hay ahorro de contexto real: la tabla oficial de comparación dice que
  con `skills:` precargadas "the full skill content is injected at
  startup" — igual que si estuviera ya en el cuerpo del subagente).

Aplicado a los casos concretos que pidió el usuario:

- **`revisor-pr`** (checklist fijo de phpcs/stylelint/buenas prácticas
  Drupal): el checklist en sí **sí** es candidato — no porque `revisor-pr`
  lo necesite como Skill aparte (para él solo, da igual tenerlo en su propio
  prompt), sino porque **el mismo checklist ya está duplicado hoy** en
  `agente-desarrollo-drupal.md` y `agente-frontend.md` ("Sigue los estándares
  de phpcs..." / "Sigue los estándares de stylelint..." — instrucciones
  redundantes con el checklist de `revisor-pr`, ver sección 4).
- **`estilo`** (reglas de tono ES/EN/PT): candidato una vez que exista
  `guia-tono.md` (tarea que el propio `agente-estilo.md` ya se auto-asignó
  como primera tarea, pero **aún no se hizo**) — porque `creador-contenidos`
  también necesita esas reglas, no solo `estilo`.
- **`seo`** (checklist metatag/schema_metatag/simple_sitemap): candidato de
  prioridad media — hoy no está duplicado en ningún otro agente, así que el
  argumento de "una sola fuente de verdad" no aplica todavía; el beneficio
  real es portabilidad a otros proyectos Drupal de Danemar (ver sección 4).
- **`tester`** (protocolo Drush+Playwright): este es el caso más fuerte y
  más barato de resolver — ver sección 4, propuesta 1 — porque Claude Code
  **ya trae una Skill bundled hecha exactamente para esto** (`/verify` +
  `/run-skill-generator`), no hay que escribirla a mano.

## 3. Evidencia externa: ¿hay consenso sobre "cada agente con su Skill"?

No. Ninguna fuente consultada recomienda una correspondencia 1:1
agente↔Skill como default. El consenso es selectivo, con el mismo criterio
en todas las fuentes:

- *"If multiple agents or conversations need the same expertise — like
  security review procedures or data analysis methods — create a Skill
  rather than building that knowledge in each one."* — [Claude — Skills
  explained](https://claude.com/blog/skills-explained)
- *"The decision rule: Skill teaches the how, Hook enforces the rule,
  Subagent isolates the work. Use all three together for production agent
  setups."* — síntesis recurrente en fuentes de terceros (duotach, totalum,
  joseparreogarcia), consistente con la guía oficial de Claude sobre
  cuándo usar `CLAUDE.md` vs Skills vs Hooks vs Subagentes:
  [Claude — Steering Claude Code](https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more)
- *"Code examples and style guides go in skills, not in the subagent... Output
  format stays in the Subagent prompt (the agent's own work spec); code
  style/test patterns go in Skills (general knowledge, shared across
  agents)."* — patrón documentado en guías de mejores prácticas de terceros
  (ver [github.com/dianyike/claude-code-insights](https://github.com/dianyike/claude-code-insights/blob/main/subagent-best-practices.md)),
  coincide exactamente con el criterio de la sección 2.

Ninguna fuente, incluida la documentación oficial, sugiere que dar Skill
propia a cada subagente mejore el resultado por sí solo — el beneficio
siempre se enmarca en reutilización/invocación directa/portabilidad, nunca
en "más Skills = mejor". Y ninguna de las fuentes de terceros consultadas
describe el patrón "subagente aislado descubre e invoca skills en runtime"
como práctica recomendada — todas coinciden, implícita o explícitamente, en
que el contenido reusable se entrega precargado al subagente, no
descubierto por él en vivo (sección 0.4).

## 4. Propuestas concretas

> **Nota de mecanismo (agregada en la segunda pasada, aplica a las 4
> propuestas de abajo):** en todos los casos, "el subagente X usa la Skill
> Y" significa **precarga** — agregar `skills: [Y]` al frontmatter de
> `.claude/agents/X.md` — nunca agregar `Skill` a su lista de `tools`. Es un
> cambio de código/config (sección 0.3), con el beneficio de funcionamiento
> específico que se detalla en cada propuesta (nunca "el subagente decide
> dinámicamente invocar la skill").

### Propuesta 1 — Grabar la receta de verificación del `tester` como Skill de proyecto, con la herramienta que Claude Code ya trae para esto

**Problema**: `agente-tester.md` describe el *método* de verificación
(Drush, Playwright contra `site-dev.danemarparceros.net`, revisión de logs,
caso feliz + caso borde) pero no fija los comandos exactos — cada sesión
puede re-derivar ligeramente distinto cómo levantar/verificar el entorno
(DDEV, rutas, credenciales de prueba), lo que es exactamente el tipo de
"re-descubrimiento" que las Skills existen para evitar.

**Cambio propuesto**: correr una vez el comando bundled `/run-skill-generator`
(incluido de fábrica en Claude Code desde v2.1.145) **desde la sesión
principal** (el generador corre a ese nivel, no dentro del subagente
`tester` — coherente con la sección 0.4) contra este proyecto. Levanta el
entorno desde cero, registra qué funcionó (comandos DDEV/Drush, variables de
entorno, script de arranque de Playwright), y lo graba como Skill de
proyecto en `.claude/skills/verify/SKILL.md`. A partir de ahí, dos usos
distintos del mismo archivo (doble beneficio, sección 0.3):
- El subagente `tester` la **precarga** agregando `skills: [verify]` a su
  frontmatter (`.claude/agents/tester.md`), en vez de re-derivar comandos
  cada vez — esto no le da ninguna herramienta nueva, solo fija el contenido
  exacto que ya vería igual si estuviera escrito inline.
- Cualquiera puede tipear `/verify` directo **desde la sesión principal**,
  sin pasar por todo el flujo del subagente, para una verificación rápida —
  esto sí es una capacidad nueva, y solo funciona a ese nivel (0.1).

Fuente: [code.claude.com/docs/en/skills — "Run and verify your app"](https://code.claude.com/docs/en/skills).

**Esfuerzo**: bajo — el trabajo pesado (recorrer el entorno y grabar la
receta) lo hace el generador, no hay que escribir la Skill a mano.

**Riesgo**: bajo. Si la receta de arranque cambia (p. ej. cambia la config de
DDEV), hay que volver a correr el generador — riesgo de que la Skill quede
desactualizada si nadie la revisita, mitigable simplemente recordándolo en
`harness/memoria/largo-plazo/desarrollo.md` cuando pase.

---

### Propuesta 2 — Skill compartida de revisión (phpcs/stylelint/buenas prácticas Drupal) entre `revisor-pr`, `desarrollo-drupal` y `frontend`

**Problema**: el checklist real (comando `phpcs` exacto + ruleset
`phpcs.xml`, comando `stylelint` + config, más los ítems de revisión
razonada: convención de nombres de hooks/servicios, credenciales
hardcodeadas, accesibilidad, consistencia Forge/Dossier) vive hoy
**duplicado en tres archivos**: completo en `agente-revisor-pr.md`
(y su `.claude/agents/revisor-pr.md`), y repetido parcialmente como
instrucción de auto-chequeo en `agente-desarrollo-drupal.md`/
`.claude/agents/desarrollo-drupal.md` ("Antes de entregar, correr
`vendor/bin/phpcs`...") y `agente-frontend.md`/`.claude/agents/frontend.md`
("Antes de entregar, correr `npm run lint:css`..."). Esto ya es una
duplicación confirmada (no hipotética — verificada leyendo los tres
archivos), con riesgo real de que diverjan sin que nadie lo note si el
ruleset o la config cambian.

**Cambio propuesto**: crear `.claude/skills/revision-pr-drupal/SKILL.md` con
los comandos exactos y el checklist de revisión razonada. Agregar
`skills: [revision-pr-drupal]` al frontmatter de los tres:
`.claude/agents/revisor-pr.md` (su fuente de autoridad), y
`.claude/agents/desarrollo-drupal.md`/`.claude/agents/frontend.md` (como
auto-chequeo previo a abrir el PR, reduce vueltas del loop tester↔revisor-pr).
Beneficio adicional: es portable a otros proyectos Drupal que Danemar
Parceros atienda como cliente (alineado con la "reutilización en otros
proyectos" que `opciones-implementacion.md` ya identifica como un eje de
valor del harness).

**Esfuerzo**: bajo-medio — es extraer texto ya escrito a un archivo nuevo y
agregar una línea de frontmatter (`skills:`) a tres subagentes; no hay
lógica nueva ni cambio de `tools:`.

**Riesgo**: bajo. Único cuidado: al extraer, no perder el matiz de "el
revisor no corrige, solo comenta" — eso queda en el prompt propio de
`revisor-pr`, no en la Skill compartida (la Skill es el *qué revisar*, no
el *qué hacer con el hallazgo*, que sigue siendo distinto por rol).

---

### Propuesta 3 — Empaquetar `guia-tono.md` como Skill compartida entre `estilo` y `creador-contenidos`

**Problema**: `agente-estilo.md` ya identifica que hoy no existe una guía de
tono explícita y se auto-asigna como primera tarea proponer una
`guia-tono.md` — pero como documento suelto, solo `estilo` la consultaría
(en revisión posterior), no `creador-contenidos` (en el momento de
redactar). Eso significa que las inconsistencias de tono se detectarían
tarde (después de escribir), no se previenen al escribir.

**Cambio propuesto**: cuando se apruebe y ejecute esa primera tarea de
`estilo`, escribir el resultado directamente como
`.claude/skills/guia-tono-danemar/SKILL.md` en vez de un `.md` suelto en
`harness/`, y agregar `skills: [guia-tono-danemar]` al frontmatter de
**ambos** `.claude/agents/estilo.md` y `.claude/agents/creador-contenidos.md`.
Beneficios concretos sobre la alternativa de documento plano:
`creador-contenidos` la precarga y aplica el tono al redactar (no solo lo
corrige después `estilo`), y el usuario puede invocarla directo con
`/guia-tono-danemar` **desde la sesión principal** para chequear una frase
puntual sin levantar todo el flujo de `estilo`.

**Esfuerzo**: bajo — mismo contenido que ya estaba planeado escribir, cambia
solo el formato/ubicación del archivo. No es trabajo adicional sobre lo ya
previsto.

**Riesgo**: bajo.

---

### Propuesta 4 (prioridad media) — Skill de checklist SEO/GEO/AEO Drupal (`metatag`/`schema_metatag`/`simple_sitemap`)

**Problema**: el checklist de auditoría (huecos de `Service` JSON-LD por
`service_card`, `og:url` canónico, sitemap/`robots.txt`/`llms.txt`, patrón de
`drush config:export`) vive como prosa dentro de `agente-seo.md`/
`.claude/agents/seo.md`. A diferencia de la Propuesta 2, hoy **no está
duplicado** en ningún otro agente del harness, así que el argumento de "una
sola fuente de verdad" no aplica todavía — el beneficio real es que quede
como procedimiento directamente ejecutable/repetible desde la sesión
principal (`/seo-checklist-drupal`), y portable a otros sitios Drupal que
Danemar atienda (mismo argumento de reutilización que la Propuesta 2).

**Cambio propuesto**: crear `.claude/skills/seo-checklist-drupal/SKILL.md`
con el checklist de auditoría y los comandos de config export, referenciada
con `skills: [seo-checklist-drupal]` en `.claude/agents/seo.md`.

**Esfuerzo**: bajo.

**Riesgo**: bajo. Como es prioridad media (no resuelve una duplicación
activa), puede posponerse sin costo si hay otras tareas más urgentes.

---

## 5. Qué NO aplica y por qué (para no proponer trabajo sin beneficio real)

No se propone Skill dedicada para los siguientes 8 roles — en cada caso el
motivo es la ausencia de un segundo consumidor y/o de contenido lo bastante
grande/estable como para justificar un archivo aparte (y, tras la segunda
pasada, con el motivo adicional de que ninguno de estos casos tiene sentido
como skill invocable en runtime por el propio subagente — solo tendría
sentido si hubiera un segundo consumidor real que precargarla, que no
existe hoy en estos ocho):

- **`orquestador`**: ya es una Skill (`/dp`), es el propio punto de entrada
  del meta-flujo, corriendo exactamente en el único nivel (sesión principal)
  donde una skill invocable dinámicamente tiene sentido en esta arquitectura
  (sección 0.4) — no hay nada más que extraer.
- **`planificador`**: su "cómo" es razonamiento general sobre el estado del
  repo (`harness/agentes/*`, `harness/memoria/`) para producir un plan
  distinto cada vez — no hay un procedimiento fijo reusable que empaquetar.
- **`documentador`**: las convenciones de `harness/memoria/` son específicas
  de este repo, sin un segundo consumidor que las necesite precargadas ni
  motivo para que el usuario las invoque como comando suelto.
- **`project_writer`**: sus comandos de `gh project`/GraphQL están atados al
  esquema de un solo tablero (`site danemar`) — extraerlos a Skill no los
  hace más reusables hasta que exista un segundo tablero real que los
  necesite; sería trabajo especulativo hoy. (Hallazgo aparte, no relacionado
  con Skills: GitHub ya expone sub-issues nativos vía GraphQL — esto resuelve
  el "no confirmado" que el propio `agente-project-writer.md` deja anotado
  en su nota honesta de alcance; vale una investigación de seguimiento
  específica sobre eso, pero es una mejora de funcionalidad, no de
  empaquetado en Skill.)
- **`historiador`**: estilo narrativo específico de `harness/historia/`, sin
  reuso fuera de ese rol.
- **`investigador-mejoras-harness`** (este agente): el procedimiento es
  "leer estado actual + buscar en la web + escribir informe" — el contenido
  sustantivo cambia en cada corrida, no hay checklist fijo que valga la pena
  fijar en un archivo aparte.
- **`investigador-contenidos`**, **`investigador-noticias`**: el valor de
  estos roles es el hallazgo en sí, no un procedimiento repetible —no hay
  "cómo" fijo que extraer.
- El resto de `creador-contenidos`/`desarrollo-drupal`/`frontend` fuera de lo
  ya cubierto en las Propuestas 2 y 3: su conocimiento (estructura de
  Paragraphs, patrón de Drush scripting, sistema de tokens/SDC del tema) es
  demasiado contextual por tarea como para fijarlo en una Skill sin quedar
  obsoleto rápido — mejor mantenerlo vivo en `harness/memoria/largo-plazo/`,
  que ya es responsabilidad del `documentador` (ver sección 6.4, que retoma
  exactamente este punto con más detalle).

---

## 6. Ciclo de mejora continua y especialización (tercer pedido del usuario)

Cita del pedido: catálogo de skills por dominio de agente, quién las
instala, un patrón de "agente atascado → pide skill al agente de mejora",
un catálogo propio de estructuras/patrones que crece con el tiempo, y
métricas de éxito/agilidad con posibilidad de rollback. Se diseñan los 5
puntos en el mismo orden, y se cierra con una recomendación única de qué
implementar ya y qué es prematuro.

### 6.1 Catálogo real de skills por dominio (no una cantidad inventada — verificado contra catálogos reales)

Se investigó el ecosistema real de Agent Skills instalable (`npx skills
add`, del ecosistema `vercel-labs/skills`/`skills.sh` y marketplaces
asociados) buscando específicamente qué existe para el trabajo real de
`desarrollo-drupal` y `frontend` en este proyecto — no una cifra estimada.

**Para `desarrollo-drupal`** — el catálogo más relevante encontrado es
[`grasmash/drupal-claude-skills`](https://github.com/grasmash/drupal-claude-skills)
(Matthew Grasmick, mantenedor conocido del ecosistema Drupal/Acquia — autor
de BLT), **12 skills en total**, instalable con `npx skills add
grasmash/drupal-claude-skills` (o el `install.sh` del repo). De esas 12, las
que calzan con el trabajo real de `desarrollo-drupal` en este proyecto (Drush
scripting, content types/campos/Paragraph types, config export, sin
PHPUnit/OAuth/Search API todavía) son, en efecto, **4** — coincide con la
estimación del usuario:

| Skill | Qué resuelve | Ya instalada localmente |
|---|---|---|
| `drupal-ddev` | DDEV, comandos Drush, import/export/snapshot de base de datos, Xdebug, performance | **Sí** — confirmado leyendo `~/.claude/skills/drupal-ddev/SKILL.md` en esta sesión |
| `drupal-config-mgmt` | Config export/import seguro, config splits — exactamente el patrón `drush config:export` que ya usa `desarrollo-drupal`/`seo` | No |
| `drupal-at-your-fingertips` | 50+ temas de APIs de Drupal (hooks, entidades, forms, caching) — cubre directamente la regla ya escrita en `agente-desarrollo-drupal.md`: "no inventes hooks/APIs no confirmados" | No |
| `drupal-contrib-mgmt` | Gestión de módulos contrib vía Composer, compatibilidad Drupal 11 | No |

Una quinta candidata de prioridad más baja hoy: `drupal-config-reconcile`
(resuelve drift de config entre ambientes) — este proyecto todavía tiene un
solo ambiente real (`site-dev.danemarparceros.net`), así que no hay drift
multi-ambiente que reconciliar todavía; queda documentada para cuando exista
un ambiente de producción separado. El resto del paquete de 12
(`drupal-testing` con PHPUnit, `drupal-simple-oauth`, `drupal-search-api`,
`drupal-canvas`/`drupal-canvas-sdc`, `ivangrynenko-cursorrules-drupal`,
`skill-developer`) **no aplica hoy**, con razón puntual: el proyecto no usa
PHPUnit (usa el patrón Drush+Playwright ya documentado en `agente-tester.md`),
no expone API (`simple_oauth` es un camino de evolución de la Opción B, no
implementado — ver `opciones-implementacion.md`), no tiene `search_api`
instalado, y `drupal-canvas`/`drupal-canvas-sdc` están acoplados a
integración con "Acquia Source Site Builder", que este proyecto no usa (el
proyecto usa SDC core de Drupal, no el producto Canvas). `skill-developer`
(meta-skill para crear skills nuevas) sí es interesante, pero para
`investigador-mejoras-harness` mismo si en el futuro redacta skills a mano,
no para `desarrollo-drupal`.

**Para `frontend`** — la investigación confirma un hallazgo honesto
importante: **la skill `bootstrap-components`, ya disponible globalmente en
esta sesión (`~/.claude/skills/bootstrap-components/SKILL.md`, verificada
directamente), no aplica a este proyecto.** Es exclusivamente sobre
componentes de Twitter Bootstrap 5.3 (`.btn`, `.modal`, `data-bs-*`,
`bootstrap.bundle.js`) — y `danemar_theme.libraries.yml` (revisado
directamente) no tiene ninguna dependencia de Bootstrap: el tema se
construye sobre `tokens.css`/`base.css`/`sections.css` propios + CSS de
componentes de Drupal core, con SDC para los componentes del tema. Su sola
presencia en la sesión no significa que aplique — es un hallazgo de "qué NO
usar", no una propuesta.

Lo que sí calza, verificado:

| Skill | Qué resuelve | Fuente |
|---|---|---|
| `drupal-sdc` | Estructura de directorios, `component.yml`+JSON Schema de props, patrones de slots en Twig — exactamente el modelo de componentes SDC que ya usa `danemar_theme` (Drupal 10.3+ core, compatible con Drupal 11.4 de este sitio) | [AJV009/drupal-devkit](https://skillsmp.com/skills/ajv009-drupal-devkit-plugins-drupal-core-skills-drupal-sdc-skill-md), `npx skills add https://github.com/AJV009/drupal-devkit --skill drupal-sdc` |
| `drupal-coding-standards-rt` | Revisión de calidad/estándares de código (sibling de `drupal-sdc`, mismo repo) — solapa con la Propuesta 2 (checklist de revisión), evaluar cuál se prefiere en vez de duplicar | mismo repo que `drupal-sdc` |
| `web-quality-audit`, `performance`, `core-web-vitals`, `accessibility`, `seo`, `best-practices` (6 skills en un solo repo) | Auditoría Lighthouse/Core Web Vitals (LCP/INP/CLS), accesibilidad WCAG, SEO técnico, buenas prácticas — agnóstico de framework, aplica igual a Twig/CSS plano que a React | [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) (Addy Osmani, ingeniero de performance web reconocido, Chrome/Google), `npx skills add addyosmani/web-quality-skills` |

Sumando `drupal-sdc` + las 6 de `web-quality-skills` da **7** — otra vez
coincide con la estimación del usuario ("6 o 7 skills" para frontend), sin
haber partido de esa cifra: es el resultado de investigar el catálogo real.
`drupal-coding-standards-rt` queda como candidata de prioridad menor (se
solapa con la Propuesta 2 de este informe — mejor decidir cuál de las dos
fuentes de verdad se adopta, no ambas, para no reintroducir el mismo riesgo
de duplicación que motiva la Propuesta 2).

### 6.2 Quién instala y da acceso — mecanismo real (no el que el usuario imagina si no es viable)

Verificado directamente en `.claude/agents/investigador-mejoras-harness.md`
(este mismo agente): `tools: WebSearch, WebFetch, Read, Write` — **sin
`Bash`**. Esto significa que **hoy no puedo correr `npx skills add` yo
mismo** — no tengo la herramienta para ejecutar procesos del sistema.

Esto no es un accidente a corregir agregando `Bash` a mi propia lista de
`tools` — sería la decisión equivocada, por dos razones concretas:

1. **Instalar software es exactamente el tipo de acción que este harness ya
   trata como gate 0.3**, y el propio `agente-documentador.md` (el agente
   más parecido a este en cuanto a investigar mejoras) ya lo resuelve así:
   *"Instalar una herramienta/programa... siempre como propuesta, nunca
   instalado sin aprobación explícita"*. Ampliar mis `tools` para poder
   instalar solo estaría duplicando, con más riesgo, algo que ya está bien
   resuelto para `documentador`.
2. Es coherente con el principio ya establecido en la sección 0.4: en esta
   arquitectura, solo la sesión principal (donde vive el orquestador)
   "alcanza hacia afuera" — instalar un paquete nuevo es, con más razón
   todavía que invocar una Skill, una acción que debe quedar centralizada
   ahí, no distribuida a un subagente de solo-lectura.

**Mecanismo real propuesto**: `investigador-mejoras-harness` investiga y
entrega al orquestador una propuesta con el comando exacto (p. ej. `npx
skills add grasmash/drupal-claude-skills --skill drupal-config-mgmt --skill
drupal-at-your-fingertips --skill drupal-contrib-mgmt`), el orquestador la
presenta al usuario, y **si el usuario aprueba, el orquestador la ejecuta
él mismo** (la sesión principal sí tiene `Bash` disponible de forma nativa,
sin necesitar ningún cambio de configuración) o delega la ejecución al
`desarrollo-drupal`/`documentador` según corresponda (instalar un paquete
npm de skills es más cercano a una tarea de `documentador`, que ya escribe
en `.claude/agents/*.md`, que a una de desarrollo del sitio). **Si en el
futuro se decide que `investigador-mejoras-harness` instale directamente**,
el cambio concreto sería agregar `Bash` a la línea 4 de
`.claude/agents/investigador-mejoras-harness.md` — documentado aquí
explícitamente para que quede claro qué habría que tocar, pero **no se
recomienda hacerlo** por las dos razones de arriba.

### 6.3 Escalamiento "agente atascado → pide skill al agente de mejora"

Confirmado (mismo hallazgo que la sección 0.1): ningún subagente, incluido
este, tiene `SendMessage`, `Agent` ni `Skill` en su `tools` — no hay
escalamiento en tiempo real posible dentro de una misma ejecución. El patrón
más cercano que sí es viable con las herramientas reales de este entorno es
exactamente el que ya sugirió el usuario en su cita, aterrizado a lo que
`agente-orquestador.md` ya documenta: *"Recoge los reportes de cada paso,
incluyendo fallos"* — el reporte de vuelta al orquestador (que ya es como
funciona hoy, todos los agentes reportan hacia arriba, nunca hablan directo
entre sí ni con el usuario) es el único punto de enganche real.

**Diagnóstico**: esto **no está implícito hoy de forma explícita** — reviso
los 13 `.claude/agents/*.md` y ninguno tiene una convención formal para
señalar "esto me faltó por falta de una capacidad/skill concreta" como
distinto de un fallo normal de tarea (un `fail` del `tester` hoy se ve igual
que "no sé cómo hacer esto sin una skill que no tengo"). Hace falta
agregarla explícitamente.

**Cambio propuesto** (código concreto, mecánico, bajo riesgo): agregar una
convención de una línea a la sección "Entregable"/"Salidas" de los 13
`harness/agentes/*.md` (y reflejarla en el `.claude/agents/*.md`
correspondiente, siguiendo el mismo patrón que ya usa `documentador` para
mantenerlos sincronizados):

> *"Si durante la tarea identificás que te falta una capacidad concreta
> (no solo un permiso — conocimiento de un dominio específico que no tenés y
> que una skill podría cubrir, p. ej. 'no tengo patrones de Drupal Migrate
> API para esta migración'), señalalo explícitamente en tu reporte como
> **'bloqueado por falta de capacidad: <descripción concreta>'** — no lo
> disfraces de fallo genérico ni lo silencies (regla 0.2). No busques ni
> instales nada vos mismo."*

Y una línea equivalente en `agente-orquestador.md`, en su sección "Cómo
opera" (paso 5, "Recoge los reportes"): cuando el orquestador recibe un
reporte con esa marca, en vez de (o además de) devolver la tarea al agente
original, dispatcha a `investigador-mejoras-harness` con el hueco concreto
para que busque una skill candidata (sección 6.1 es, en los hechos, el
resultado de correr este mismo proceso una vez).

**Esfuerzo**: bajo — 14 archivos (13 `harness/agentes/*.md` + `agente-orquestador.md`,
más sus 13 espejos en `.claude/agents/`), una línea de convención cada uno,
sin lógica nueva.

**Riesgo**: bajo. Cuidado real: no confundir "bloqueado por falta de
capacidad" con "bloqueado por falta de aprobación humana" (regla 0.3) — son
señales distintas y no deberían compartir la misma etiqueta, para que el
orquestador sepa a quién dispatchar en cada caso.

### 6.4 Catálogo propio de estructuras/patrones que crece con el tiempo

**Hallazgo importante antes de proponer dónde vive esto**: el usuario pide
"un listado de estructuras, patrones de desarrollo, patrones de contenido...
para que cada vez el sistema sea mucho más especializado" — pero **este
harness ya tiene, diseñado y funcionando, exactamente ese concepto**:
`harness/memoria/largo-plazo/*.md`, responsabilidad de `documentador`,
descrito textualmente en `agente-documentador.md` como *"Conocimiento
consolidado por tema (`contenido.md`, `seo.md`, `desarrollo.md`,
`decisiones-arquitectura.md`, etc.): qué funcionó, qué no, por qué se
decidió algo, **patrones que deben repetirse**"*. Crear una ubicación nueva
(`harness/mejoras/patrones/`) duplicaría esa responsabilidad y crearía dos
fuentes de verdad compitiendo por el mismo tipo de contenido — exactamente
el mismo riesgo de drift que ya se señaló para los checklists duplicados
(sección 4, Propuesta 2). **Recomendación: no crear una ubicación nueva —
extender la que ya existe.**

**Qué contendría, en concreto (ejemplos reales de este proyecto, verificados
al escribir este informe, no genéricos)**:
- En `memoria/largo-plazo/desarrollo.md`: el patrón de contenido estructurado
  como Paragraphs anidados (14 tipos: `hero`, `service_card`,
  `process_step`, `faq_item`, `client_item`, `legal_section`, etc.,
  referenciados desde `field_sections` de `landing_page`/`legal_page`) como
  **modelo de datos**, junto con — capa distinta, no sustituta — el uso de
  **SDC** en el tema como capa de **renderizado/Twig+CSS** de esos mismos
  Paragraphs (aclaración honesta: no es "Paragraphs en vez de SDC", son dos
  capas que coexisten — el patrón vale la pena documentarlo así de preciso
  para que un agente nuevo no las confunda).
- En `memoria/largo-plazo/frontend.md` (hoy no existe explícitamente en el
  listado de `agente-documentador.md` — habría que crearlo, con el mismo
  criterio que los demás): el patrón de **custom properties CSS para el
  toggle Forge/Dossier** (`danemar_theme.settings:dp_direction` controlando
  variables CSS en `tokens.css` para las dos direcciones visuales, en vez de
  temas/CSS duplicados por dirección).
- En `memoria/largo-plazo/contenido.md`: el patrón de tono de marca ES/EN/PT
  (una vez exista `guia-tono-danemar`, sección 4 Propuesta 3 — el patrón en
  sí, más allá del contenido puntual de la guía, es "redactar primero en ES,
  coordinar traducciones después vía `content_translation`, nunca reescribir
  desde cero", ya mencionado en `agente-creador-contenidos.md`).

**Cómo lo consumirían los agentes** — mismo criterio que la sección 0/2 de
este informe, aplicado de forma consistente en vez de inventar un mecanismo
paralelo:
- Mientras un patrón está en `memoria/largo-plazo/*.md` y lo usa un solo
  agente puntualmente, el `planificador`/`orquestador` se lo pasa como
  contexto al dispatchar la tarea (ya es lo que hacen hoy — `planificador`
  ya lee `harness/memoria/largo-plazo/` antes de planificar, según su propio
  `.claude/agents/planificador.md`).
- Cuando un patrón demuestra ser **estable y usado por 2+ agentes de forma
  repetida** (el mismo umbral que ya motiva las Propuestas 2-4 de la sección
  4), se "gradúa" — el mismo verbo que `agente-documentador.md` ya usa para
  pasar de corto a largo plazo — a `.claude/skills/patrones-<dominio>/SKILL.md`,
  precargada vía `skills:` en los subagentes correspondientes. Es decir: la
  sección 6.4 no es un sistema nuevo, es la extensión natural del ciclo
  corto-plazo → largo-plazo → Skill que este harness ya tiene diseñado,
  solo que hoy termina en largo-plazo y no sigue el último paso.

**Quién escribe qué** (para evitar que dos agentes escriban el mismo
archivo con criterios distintos): `documentador` sigue siendo el único
que escribe en `memoria/largo-plazo/` (como ya lo es hoy);
`investigador-mejoras-harness` **no escribe ahí directamente** — cuando su
investigación externa (sección 6.1, o futuras) encuentra un patrón que
aplicaría, lo entrega como **propuesta** en su propio informe
(`harness/mejoras/YYYY-MM-DD-propuestas.md`, este mismo tipo de archivo), y
es el orquestador quien, si el usuario aprueba, le pide a `documentador` que
lo incorpore — mismo flujo de aprobación que ya rige todo lo demás en este
informe.

### 6.5 Métricas de éxito/agilidad + rollback

**Qué es realista medir hoy, con las herramientas reales de este entorno**
(nada de telemetría que no existe):

| Métrica | ¿Medible hoy? | Cómo |
|---|---|---|
| Pass/fail del `tester` por tipo de tarea, y cuántos intentos hasta pass | Sí, pero requiere estructurarlo — hoy vive como prosa libre en `memoria/corto-plazo.md`, no como dato agregable | `documentador` empieza a registrar cada cierre de tarea con un formato mínimo estructurado (tabla: fecha, tarea, agente, resultado, intentos) |
| Rondas `tester`↔`revisor-pr` antes de un PR limpio | Sí, en principio (son comentarios/commits reales en el PR de GitHub) — pero **no la puedo recolectar yo mismo**, no tengo `Bash`/`gh` (mismo límite de la sección 6.2) | El orquestador o `project_writer` (que sí usa `gh`) la registra al cerrar el flujo; yo la leo después desde `memoria/` |
| Tareas re-planificadas por mal dimensionamiento (regla del 60% de contexto, ya documentada en `agente-planificador.md`) | Sí — es un evento que el propio `planificador`/orquestador ya puede señalar cuando ocurre | `documentador` lo anota cuando el orquestador se lo reporta |
| Tiempo/turnos por tarea | **No, no con las herramientas actuales** — ningún agente de este harness tiene acceso a telemetría de sesión/duración; requeriría un hook nuevo (fuera del alcance de un subagente, y de este informe) | Queda fuera de alcance por ahora — no inventar el dato |

**Dónde se registran**: un archivo nuevo,
`harness/memoria/largo-plazo/metricas.md`, mismo dueño (`documentador`),
mismo criterio de "no reescribir historial, agregar al final" que ya aplica
a `corto-plazo.md`.

**Quién las recolecta y analiza**: `investigador-mejoras-harness` (este
agente) — con las herramientas que sí tiene (`Read` sobre `harness/memoria/`)
— en cada corrida periódica (la misma cadencia de ~2 días ya documentada en
`agente-investigador-mejoras-harness.md`), lee `metricas.md` y
`corto-plazo.md`, y agrega a su informe una sección de "salud del sistema":
tendencias (¿bajó la tasa de pass del tester en tareas de un tipo
particular?, ¿aumentaron las rondas revisor-pr después de precargar una
skill nueva — señal de que la skill no está ayudando o está mal escrita?).
Esto responde también la parte del pedido del usuario de "estar
constantemente verificando el uso" de las skills: no es una verificación en
tiempo real (no hay herramienta para eso), es una revisión periódica de las
mismas métricas, mirando específicamente si después de precargar una skill
nueva las métricas del agente que la usa mejoraron o no.

**Rollback**: confirmado que alcanza con lo simple — todo lo que cambia en
este diseño (`.claude/skills/*/SKILL.md` nuevos, el campo `skills:` en el
frontmatter de un `.claude/agents/*.md`, entradas de
`memoria/largo-plazo/*.md`) son archivos de texto versionados en este mismo
repo git. Si una skill o un cambio de agente resulta contraproducente (las
métricas de 6.5 empeoran en vez de mejorar), el mecanismo es un `git
revert`/`git checkout <ruta>` puntual sobre ese archivo — el mismo mecanismo
que ya usa cualquier otro cambio de este repo, sin infraestructura nueva.
No hace falta versionado especial, feature flags, ni un sistema de
rollback dedicado: el tamaño y la naturaleza de estos cambios (archivos de
configuración/texto, no infraestructura corriendo) no lo justifican.

### 6.6 Recomendación única: qué implementar ya y qué es prematuro

**Implementar ya (bajo esfuerzo, beneficio claro, sin infraestructura nueva)**:
1. Precargar (vía `skills:`, sección 0.2) las **4 skills concretas y ya
   identificadas** de `desarrollo-drupal` (`drupal-ddev` ya instalada;
   `drupal-config-mgmt`, `drupal-at-your-fingertips`, `drupal-contrib-mgmt`
   a instalar) y **1 de `frontend`** para empezar (`drupal-sdc` — la más
   directamente acoplada a la arquitectura real del tema; evaluar las 6 de
   `web-quality-skills` después de ver cómo funciona esta primera, no
   todas de una vez).
2. La convención "bloqueado por falta de capacidad" en los 13 agentes +
   orquestador (sección 6.3) — mecánica, barata, y es la que hace que el
   resto del ciclo tenga sentido (sin la señal, no hay forma de saber cuándo
   dispatchar a `investigador-mejoras-harness` a buscar una skill nueva).
3. Extender `memoria/largo-plazo/` con `frontend.md` (falta hoy) y las 3
   entradas de patrón ya conocidas (sección 6.4) — no crear una carpeta
   nueva.
4. El esqueleto de `memoria/largo-plazo/metricas.md` con las 3 métricas
   realmente medibles hoy (sección 6.5) — aunque arranque vacío, es barato
   y sin él no hay forma de verificar después si algo de esto funcionó.

**Prematuro para el tamaño actual del proyecto (documentar como camino
futuro, no implementar todavía)**:
- Instalar el resto de los catálogos completos (los 12 de `grasmash` y las
  6 de `addyosmani`) de una — la mayoría no tiene un caso de uso real hoy
  (sin PHPUnit, sin API expuesta, sin multi-ambiente); instalar todo de
  entrada es exactamente lo opuesto al modelo "just-in-time" que el propio
  usuario describió (agente atascado → se busca la skill puntual que falta),
  y generaría mantenimiento/ruido sin necesidad real.
- La métrica de tiempo/turnos por tarea — no medible hoy sin construir un
  hook nuevo; no vale la pena esa inversión todavía dado el volumen de
  tareas actual de este proyecto (sitio de una landing, no un pipeline de
  alto volumen).
- Cualquier infraestructura de rollback más allá de `git revert` — el
  tamaño y naturaleza de los cambios (archivos de texto versionados) no lo
  justifica; se documenta así de simple porque alcanza, no por conservador
  de más.
- Agregar `Bash` a `investigador-mejoras-harness` para que instale skills
  por su cuenta (sección 6.2) — no aporta nada que el orquestador no pueda
  hacer ya, y rompe el principio de que solo la sesión principal ejecuta
  acciones con huella real en el entorno del usuario.

---

---

## 7. Auditoría de compatibilidad Drupal y portabilidad, previa a comitear como repo aparte

Pedido puntual (2026-08-18, cuarta pasada): antes de comitear `harness/` +
`.claude/` como repositorio aparte (versión alpha de esta arquitectura de
agentes, separada de `site/`), verificar (A) si el harness describe
correctamente cómo trabajar con Drupal real, y (B) qué está hardcodeado a
este proyecto puntual y bloquearía reusarlo en otro proyecto Drupal.
Confirmación de interpretación: son dos preguntas separadas, tratadas por
separado abajo (7.A y 7.B), cerrando con una recomendación única (7.C).

### 7.A ¿El harness describe correctamente cómo trabajar con Drupal real?

#### 7.A.1 `agente-desarrollo-drupal.md` — "Drush scripting + config export"

**Veredicto: es una forma real y sensata de trabajar en Drupal 11, con un
gap de completitud, no de corrección.** El patrón (`drush php:script`/
`php:eval` para crear content types/campos/Paragraph types vía la API
nativa, `drush config:export` para dejar todo en `config/sync`) es un
patrón legítimo y ampliamente usado — coincide exactamente con lo que
documentan `drupal-ddev` y `drupal-config-mgmt` (las dos skills instaladas
más relevantes para este agente, leídas directamente en esta pasada:
`.agents/skills/drupal-ddev/SKILL.md`, `.agents/skills/drupal-config-mgmt/SKILL.md`).
No hay error de terminología ni de comandos.

**Lo que falta mencionar (gap, no error)**: Drupal 10.3+/11 tiene un
mecanismo más moderno para *empaquetar* exactamente este tipo de cambio —
**Recipes** (`recipe.yml` + **Config Actions API**, `drush recipe`) —
pensado para instalar/configurar módulos y campos de forma declarativa y
reaplicable, con acciones como `createIfNotExists`, `grantPermissions`,
`simpleConfigUpdate`. El proyecto ya tiene un directorio `site/recipes/`
(scaffolding por defecto de Drupal core) pero **vacío** — solo el
`README.txt` que trae el core, ninguna recipe propia creada. No es un error
que `agente-desarrollo-drupal.md` no lo use hoy (el patrón Drush scripting
actual funciona y ya está probado en 13 planes), pero para un harness que
se va a reusar en otros proyectos Drupal, vale la pena que el agente sepa
que existe esta alternativa — especialmente útil para *"reprovisionar un
entorno nuevo desde cero"* o para el propio caso de portabilidad de la
sección 7.B (una Recipe es, literalmente, el mecanismo nativo de Drupal para
"config de arranque reusable entre proyectos", el mismo problema que motiva
extraer datos hardcodeados de este harness a config).

**Propuesta** (baja prioridad, no bloqueante para comitear):
- **Problema**: `agente-desarrollo-drupal.md` no menciona Recipes/Config
  Actions como alternativa/complemento al patrón Drush scripting actual.
- **Cambio propuesto**: agregar un párrafo breve en "Cómo operar" que
  mencione Recipes como opción a evaluar para cambios que tengan sentido
  como "paquete reaplicable" (p. ej. si en el futuro Danemar Parceros
  reusa este harness en un segundo proyecto Drupal y quiere portar el mismo
  módulo de moderación de contenido/SEO base) — sin obligar a migrar nada
  existente.
- **Esfuerzo**: bajo (una nota, no un cambio de flujo).
- **Riesgo**: ninguno — es aditivo, no reemplaza el patrón ya probado.

Fuentes: [Recipe Author Guide — Drupal Recipes Initiative](https://project.pages.drupalcode.org/distributions_recipes/recipe_author_guide.html),
[Use Config Actions in a Recipe — Drupalize.me](https://drupalize.me/tutorial/use-config-actions-recipe),
[Use Drupal's Config Actions API to Spice Up Your Recipes — The Droptimes](https://www.thedroptimes.com/54771/use-drupals-config-actions-api-spice-your-recipes).

#### 7.A.2 `agente-frontend.md` — Paragraphs + Twig clásico + "SDC" + SASS

**Veredicto: hay una inconsistencia factual real, no solo una imprecisión
menor — y esta misma investigación (sección 6.4 de este informe, escrita
horas antes en la misma fecha) la reprodujo sin verificarla.** Se verificó
directamente contra el código real del tema (`site/web/themes/custom/danemar_theme/`),
no contra la documentación del harness:

- **No existe ningún componente SDC en el tema real.** `find` sobre todo
  `danemar_theme/` no encuentra ningún archivo `*.component.yml` ni ningún
  directorio `components/`. Confirmado también por
  `harness/memoria/largo-plazo/decisiones-arquitectura.md` (escrito en una
  pasada *anterior* del mismo día): *"no hay componentes SDC propios
  implementados (`*.component.yml` no existe en el tema)"*.
- **El renderizado real es Twig clásico por bundle de Paragraph**
  (`templates/paragraph/paragraph--hero.html.twig`,
  `paragraph--service-card.html.twig`, etc. — 14 archivos, uno por tipo),
  no SDC. El propio comentario de cabecera de `paragraph--hero.html.twig`
  lo dice explícitamente: *"Traducido de
  `diseño/handoff/components/dp-hero/dp-hero.twig` a template de Paragraph
  clásico"* — es decir, alguien tradujo **a mano** el mockup en formato SDC
  del paquete de diseño a un template Twig clásico de Drupal. No hay
  ninguna invocación real de un componente SDC (`{{ component(...) }}` o
  equivalente) desde esos templates.
- **Los archivos `.component.yml` sí existen — pero viven en
  `diseño/handoff/components/`, el paquete de diseño**, no en el tema. Son
  el formato de entrega del diseño (fuente de verdad *visual*, correcto
  como lo describe `agente-frontend.md` línea 6), no una integración viva
  de Drupal SDC.

**Por qué esto importa (regla 0.2, honestidad)**: `harness/memoria/largo-plazo/desarrollo.md`
y la sección 6.4 de este mismo informe (escritas *después* de
`decisiones-arquitectura.md`, mismo día) afirman textualmente que "SDC... es
la capa de **renderizado**" del tema y que "un `service_card` es, a la vez,
un Paragraph type... y un componente SDC" — eso es una afirmación
verificable que no es cierta hoy contra el código real. Es exactamente el
tipo de deriva que la propia regla 0.2 de `AGENTS.md` pide corregir ("si
algo hecho en una sesión anterior resulta estar roto o mal documentado,
decirlo y corregir el registro"). La combinación real, correcta, es:
**Paragraphs (modelo de datos) + Twig clásico por bundle (renderizado real
en producción) + SDC-format mockups en `diseño/handoff/` (especificación de
diseño, traducida a mano, no integrada vía Drupal SDC) + SASS (pipeline de
compilación del CSS propio, correctamente documentado)**. El patrón SASS en
sí (`scss/` → `css/` vía Dart Sass CLI, custom properties `--dp-*` para el
toggle Forge/Dossier preservadas como CSS real y no `$variables` de Sass) es
coherente con prácticas reales de temas Drupal 10.3+/11 y está bien
documentado, sin objeciones.

**Propuesta**:
- **Problema**: `harness/memoria/largo-plazo/desarrollo.md` (sección
  "Aclaración importante") y la sección 6.4 de este mismo informe afirman
  que el tema usa SDC como capa de renderizado; el código real no lo
  confirma — es Twig clásico por bundle, con SDC solo en el paquete de
  diseño.
- **Cambio propuesto**: corregir `harness/memoria/largo-plazo/desarrollo.md`
  (y agregar una nota de corrección a la sección 6.4 de este archivo) para
  reflejar la cadena real: Paragraphs (datos) → mockup SDC en
  `diseño/handoff/` (especificación visual) → traducción manual a Twig
  clásico por bundle (renderizado real). Opcionalmente, evaluar como tarea
  aparte (no urgente) si conviene migrar el tema a SDC real — el `find`
  confirma que Drupal 11.4/`^11` del proyecto lo soporta nativamente y ya
  hay una skill instalada (`drupal-sdc`) que documenta cómo hacerlo — pero
  eso es una decisión de arquitectura del sitio, no una corrección de
  harness, y está fuera del alcance de "listo para comitear".
- **Esfuerzo**: bajo (corrección de un párrafo en memoria + una nota en este
  informe).
- **Riesgo**: bajo — es documentación, no código. El riesgo real es *no*
  corregirlo: un agente futuro (`desarrollo-drupal`/`frontend`) podría
  intentar "editar el componente SDC de `service_card`" y no encontrarlo,
  perdiendo tiempo, o peor, un agente nuevo podría inventar que existe un
  archivo `.component.yml` en el tema que en realidad no está ahí.

#### 7.A.3 `agente-tester.md` — protocolo Drush+Playwright (sin PHPUnit)

**Veredicto: elección razonable y bien justificada para el estado actual del
proyecto, no un gap real.** El proyecto no tiene ningún módulo custom propio
(`site/web/modules/custom/` no existe todavía — toda la lógica vive en
`src/Hook/` del tema o en configuración), así que no hay superficie de
código con lógica de negocio aislable que un test unitario/kernel de
PHPUnit tradicional necesitaría cubrir. Verificación end-to-end contra el
sitio real (Drush para datos/config + Playwright para flujo de usuario) es
proporcional al riesgo real hoy.

**Lo que sí vale la pena anotar como opción futura (no urgente)**: si el
proyecto llega a tener un módulo custom propio, existe **Drupal Test
Traits (DTT)** — confirmado en la skill `drupal-at-your-fingertips`
(`references/dtt.md`, leída directamente en esta pasada) — que es PHPUnit
por debajo pero corre contra el **sitio existente** (`ExistingSiteBase`),
sin recrear la base de datos, mucho más rápido que el PHPUnit tradicional de
Drupal (`KernelTestBase`/`BrowserTestBase`) y compatible en espíritu con el
patrón "verificar contra el entorno real" que ya sigue este harness. No es
un reemplazo del patrón actual — es el camino natural de evolución el día
que exista lógica custom que amerite tests automatizados versionados (hoy
la verificación la hace el agente `tester` en cada tarea, no hay suite de
regresión persistente).

**Propuesta**:
- **Problema**: ninguno urgente — el patrón actual es correcto para el
  estado real del proyecto.
- **Cambio propuesto**: agregar una nota breve en `agente-tester.md` (o en
  `harness/memoria/largo-plazo/desarrollo.md`) documentando DTT como camino
  de evolución si aparece un módulo custom, para que quien lo evalúe en el
  futuro no tenga que redescubrirlo.
- **Esfuerzo**: bajo (una nota).
- **Riesgo**: ninguno.

#### 7.A.4 `agente-seo.md` — `metatag`/`schema_metatag`/`simple_sitemap`

**Veredicto: sigue siendo el stack estándar/recomendado del ecosistema
Drupal hoy — confirmado tanto por búsqueda web dirigida como porque es el
mismo stack que documentan las skills de Drupal instaladas.** `metatag` y
`schema_metatag` (JSON-LD) siguen siendo, en 2026, los módulos de referencia
para meta tags y datos estructurados en Drupal 10.3/11; `simple_sitemap`
sigue siendo el módulo estándar para sitemaps XML, sin que haya aparecido un
sucesor que lo desplace. No hay corrección que hacer al stack en sí.

**Hallazgo menor, no crítico**: existe un módulo nuevo,
**`schema_metatag_ai`**, que agrega generación asistida por IA de campos de
Schema.org (botón "Generate Schema Metatag" en el formulario de contenido)
— relevante puntualmente para el pendiente ya documentado del propio agente
("falta `Service` JSON-LD por cada `service_card`"). No es indispensable
(el patrón manual de config YAML que ya sigue el agente sigue siendo
correcto), pero vale mencionarlo como alternativa a evaluar si se quiere
automatizar ese pendiente puntual.

**Propuesta**:
- **Problema**: ninguno bloqueante.
- **Cambio propuesto**: agregar una línea en `agente-seo.md` mencionando
  `schema_metatag_ai` como opción a evaluar (no adoptar de entrada) para el
  pendiente de `Service` JSON-LD por `service_card`.
- **Esfuerzo**: bajo.
- **Riesgo**: ninguno — es solo una nota informativa, no cambia el flujo
  actual del agente.

Fuentes: [Schema.org Metatag — drupal.org](https://www.drupal.org/project/schema_metatag),
[Schema.org Metatag AI — drupal.org](https://www.drupal.org/project/schema_metatag_ai),
[Metatag — drupal.org](https://www.drupal.org/project/metatag).

### 7.B Qué está hardcodeado a este proyecto puntual (danemarparceros)

Barrido sistemático (`grep -rl` sobre `harness/` + `.claude/`, listado
completo de archivos con coincidencias verificado en esta pasada, no
estimado) confirma que **la mayoría de los 14 agentes + sus 3 flujos + las
3 skills de sesión tienen algún grado de acoplamiento a este proyecto
puntual** — es la norma, no la excepción, en este harness tal como está
escrito hoy. Se agrupan por categoría, con la clasificación pedida
(generalizar vs. correcto que quede específico) en cada una.

#### 7.B.1 Hallazgo más importante: `AGENTS.md` — ubicación + secreto real

**Este es el hallazgo de mayor severidad de toda la auditoría.**
`AGENTS.md` vive en la raíz del proyecto (`/home/sendor/danemar_site/AGENTS.md`),
**fuera** de `harness/`, `.claude/` y `site/` — es decir, fuera de lo que se
va a comitear como el nuevo repo del harness *y* fuera del repo `site/`
(la nota propia del archivo dice explícitamente: *"Este archivo vive fuera
del repo git (`site/`), por lo que no se sube a ningún remoto"*). Sin
embargo, **todos** los agentes y flujos referencian constantemente
"regla 0.1/0.2/0.3/0.4/0.5 de `AGENTS.md`" como si fuera parte integral del
harness — sin `AGENTS.md`, esas referencias apuntan a un archivo que no
existiría en el nuevo repo.

Peor: la sección 0.4 de `AGENTS.md` contiene **un token real de Cloudflare
Tunnel en texto plano** (`cloudflared tunnel run --token eyJ...`), con una
advertencia propia de que es "una credencial real... no pegarlo en lugares
públicos/compartidos". Si `AGENTS.md` se copia al nuevo repo tal cual (para
que las referencias de los agentes sigan siendo válidas), **ese token
quedaría comiteado a un repositorio nuevo** — exactamente lo que la propia
nota de seguridad del archivo pide evitar. Esto no es hipotético: es el
escenario más probable si alguien simplemente copia `AGENTS.md` al armar el
nuevo repo sin revisar su contenido primero.

**Clasificación**: mixta. Las reglas 0.1 (idioma), 0.2 (honestidad), 0.3
(aprobación antes de implementar) y 0.5 (commits tras implementar) son
**genéricas y deberían generalizarse** — no tienen nada específico de
Danemar Parceros, cualquier proyecto con este harness las querría. La regla
0.4 (URL de pruebas + token de túnel) es **correctamente específica de este
proyecto**, pero **no debería viajar al repo del harness en absoluto** — ni
siquiera en versión genérica, porque el problema no es que esté hardcodeada
(algo así siempre lo estará, hay que apuntar a *algún* entorno), sino que
**contiene un secreto real**.

**Propuesta**:
- **Problema**: `AGENTS.md` no está incluido en el commit plan del harness
  (vive fuera de `harness/`/`.claude/`), pero el harness depende de él por
  referencia constante; si se copia sin editar, se filtra un secreto real.
- **Cambio propuesto** (bajo esfuerzo, alta prioridad — el único hallazgo de
  esta auditoría que se recomienda resolver *antes* de comitear, no después):
  1. Crear un `AGENTS.md` **genérico** dentro del nuevo repo del harness,
     con solo las reglas 0.1/0.2/0.3/0.5 (sin secretos, sin URL de proyecto
     específico) — esto es lo que de verdad hace portable al harness, más
     que cualquier otro cambio de esta sección.
  2. Dejar la regla 0.4 (URL de entorno + comando de túnel) en un archivo
     separado, **no comiteado** al repo del harness (p. ej. seguir viviendo
     donde está hoy, fuera de git, o en un `.env`/`AGENTS.local.md`
     explícitamente en `.gitignore`), y que los agentes la referencien como
     "configuración local del proyecto", no como parte fija del harness.
  3. Antes de cualquier `git init`/primer commit del nuevo repo, correr un
     `grep -r` de palabras clave de secretos (`token`, `eyJ`, `key`,
     `secret`, `password`) sobre todo `harness/`/`.claude/` para confirmar
     que no quedó nada más filtrado (este informe ya lo hizo sobre
     `AGENTS.md`, pero vale repetirlo sobre el árbol completo justo antes
     del commit, como último chequeo).
- **Esfuerzo**: bajo — es dividir un archivo existente en dos y ajustar
  referencias, no reescribir contenido.
- **Riesgo si no se hace**: alto y concreto (filtración de una credencial
  real de infraestructura), no teórico.

#### 7.B.2 Rutas e infraestructura de este proyecto puntual — generalizar

Confirmado por grep: `danemar_theme`, `site-dev.danemarparceros.net`,
`nequeteme/danemarparceros-site`, y el ID del proyecto GitHub
(`PVT_kwHOAAPQv84BguB5`) aparecen hardcodeados en (lista completa, no
parcial): `agente-desarrollo-drupal.md`, `agente-frontend.md` (6 menciones),
`agente-seo.md`, `agente-tester.md`, `agente-revisor-pr.md`,
`agente-project-writer.md`, `flujo-desarrollo-testing.md`,
`estado-actual.md`, `MANUAL-DE-USO.md`, `mapas-arquitectura.md`, más sus
espejos completos en `.claude/agents/*.md` y la skill `dp-dev.md`
(remoto git hardcodeado en el paso 5: `git@github.com:nequeteme/danemarparceros-site.git`).
`project_writer` es, con diferencia, el más acoplado: el ID del proyecto
GitHub, el nombre del repo, y el patrón de columnas/labels están escritos
como literales en el cuerpo del agente, no como parámetros.

**Clasificación: debería generalizarse/parametrizarse**, con un mecanismo
de bajo esfuerzo (no un refactor grande): un único archivo de configuración
al principio del repo del harness, algo como
`harness/config-proyecto.md` (o `.env`/`project.yml`, formato a elección),
con placeholders del tipo:

```
theme_name: danemar_theme
site_dev_url: https://site-dev.danemarparceros.net/
repo_remote: git@github.com:nequeteme/danemarparceros-site.git
github_project: nequeteme/2 (site danemar, PVT_kwHOAAPQv84BguB5)
site_repo_path: site/
```

y que los 14 agentes + 3 flujos + 3 skills lo referencien ("ver
`harness/config-proyecto.md`") en vez de repetir el literal — mismo
principio que ya aplica el harness a `harness/memoria/` para evitar
duplicación (sección 4, Propuesta 2 de este mismo informe).

**Propuesta**:
- **Problema**: ~15 archivos repiten los mismos 4-5 valores de
  infraestructura de este proyecto puntual, literal, sin una fuente única.
- **Cambio propuesto**: crear `harness/config-proyecto.md` con esos valores,
  y reemplazar los literales por una referencia a ese archivo en los puntos
  donde hoy están hardcodeados. **No es necesario hacerlo antes de comitear
  como alpha** (a diferencia de 7.B.1) — es la mejora que haría al harness
  reusable de verdad en un segundo proyecto, pero el propio README ya puede
  documentar honestamente que hoy es un caso de uso concreto, no una
  plantilla (ver 7.C).
- **Esfuerzo**: medio — no es complejo conceptualmente, pero toca ~15
  archivos existentes (búsqueda y reemplazo dirigido, no lógica nueva).
- **Riesgo**: bajo, con un cuidado: no hacerlo a medias (algunos archivos
  referenciando el config nuevo y otros con el literal viejo sería peor que
  no hacerlo, porque generaría dos fuentes de verdad).

#### 7.B.3 Contenido de marca/dominio — correctamente específico, debe quedar separado si se comparte como plantilla

Reglas de tono ES/EN/PT, la estructura real de 14 Paragraph types (`hero`,
`service_card`, `process_step`, `faq_item`, `client_item`, `legal_section`,
etc.), el sector de negocio de Danemar Parceros (desarrollo Drupal
institucional, UE/LatAm) mencionado en `agente-investigador-contenidos.md`/
`agente-investigador-noticias.md`, y las dos direcciones visuales
Forge/Dossier — todo esto **es correcto que quede específico de este
proyecto**. No tiene sentido "genericarlo": son la configuración real de
*este* cliente/sitio, no del harness en sí. El harness como concepto
(orquestador → planificador → flujos → documentador/historiador) es
agnóstico de esto; el contenido de estos agentes de ejecución (`estilo`,
`creador-contenidos`, `investigador-contenidos`, `investigador-noticias`,
`desarrollo-drupal`, `frontend`) es, en efecto, la capa de "config de
cliente" sobre el harness genérico.

**Clasificación: correcto que quede específico — pero con una nota honesta
en el README si el repo se va a compartir/mostrar como ejemplo**: dejar
explícito que estos 6 agentes de ejecución (de los 14 totales) están escritos
*para* Danemar Parceros, no como plantilla vacía — alguien que quiera adoptar
este harness para otro cliente necesitaría reescribir el contenido de esos
6 archivos (no su estructura/formato, que sí es reusable), no solo cambiar
un config. No se propone ningún cambio de archivo aquí — solo un párrafo de
honestidad en el README (ver 7.C).

#### 7.B.4 Decisiones de arquitectura de este sitio, no universales a Drupal

`decisiones-arquitectura.md` documenta correctamente varias decisiones que
son de *este* sitio, no de Drupal en general (idioma inglés sin prefijo de
URL + es/pt con prefijo, Paragraphs sin integración SDC real todavía — ver
7.A.2 — workflow "Editorial" de 3 estados en vez del default de core). Esto
también **es correcto que quede específico** — son decisiones de producto
de este sitio puntual, documentadas exactamente donde corresponde
(`memoria/largo-plazo/`, no en la especificación de los agentes). No
requiere ningún cambio: es, de hecho, un ejemplo de que el harness ya separa
bien "diseño del harness" (`harness/agentes/`) de "estado/decisiones de este
proyecto" (`harness/memoria/`) — la separación correcta ya existe a nivel de
carpeta, el problema de 7.B.2 es que algunos agentes rompen esa separación
escribiendo el literal directo en su propio prompt en vez de referenciar
`memoria/`.

#### 7.B.5 Los 4 agentes de coordinación/meta — ya bien generalizados

Verificado leyendo los cuatro completos en esta pasada:
`agente-orquestador.md`, `agente-planificador.md`, `agente-documentador.md`,
`agente-historiador.md` **no tienen hardcoding de infraestructura** más allá
de mencionar el nombre del tablero ("site danemar") en un par de líneas de
`orquestador`/`planificador` — su lógica (cómo delega, cómo planifica, cómo
documenta, cómo decide qué es una historia) es genérica por diseño. Esto es
un punto a favor real del estado actual del harness: la capa de
coordinación, que es la parte más "core" de la arquitectura, ya está limpia
sin necesidad de ningún cambio.

### 7.C Recomendación final

**El harness está listo para comitear como "alpha" documentando
explícitamente que es un caso de uso concreto (danemarparceros), no una
plantilla genérica todavía — con una única excepción de bajo esfuerzo que
sí conviene resolver antes, no después: 7.B.1 (`AGENTS.md`/secreto).** No se
recomienda emprender la generalización completa (7.B.2) antes de comitear —
sería abrir un proyecto de refactor que el usuario explícitamente no pidió
todavía ("no quiere abrir un proyecto nuevo de generalización"), y no bloquea
la utilidad del commit como registro honesto de esta arquitectura.

**Antes de comitear (recomendado, bajo esfuerzo, ~1-2 horas):**
1. **7.B.1** — dividir `AGENTS.md` en una parte genérica (0.1/0.2/0.3/0.5,
   sí viaja al nuevo repo) y una parte específica con el secreto (0.4, no
   viaja) — es el único hallazgo con riesgo real de filtración.
2. **7.A.2** — corregir la afirmación de que el tema usa SDC como capa de
   renderizado en `harness/memoria/largo-plazo/desarrollo.md` (y la nota
   equivalente en la sección 6.4 de este mismo informe) — es una
   inconsistencia factual verificable, no una opinión, y viola la regla 0.2
   si queda sin corregir sabiendo ya que es incorrecta.
3. Un párrafo en `harness/README.md` (o al principio de este informe)
   dejando explícito, para cualquiera que abra el repo nuevo: *"este harness
   está escrito para un proyecto Drupal concreto (danemarparceros); los 6
   agentes de contenido/dominio (`estilo`, `creador-contenidos`,
   `investigador-contenidos`, `investigador-noticias`, `desarrollo-drupal`,
   `frontend`) tienen configuración/contenido específico de ese cliente que
   habría que adaptar para reusarlo en otro proyecto; los 4 agentes de
   coordinación (`orquestador`, `planificador`, `documentador`,
   `historiador`) y el patrón de flujos son agnósticos y reusables tal
   cual"* — honestidad explícita en vez de vender el repo como algo que no
   es todavía.

**Después de comitear, si en algún momento se reusa el harness en un
segundo proyecto (no urgente, no bloqueante hoy):**
4. **7.B.2** — extraer los valores de infraestructura repetidos
   (`danemar_theme`, URL de entorno, remoto git, ID de proyecto GitHub) a
   `harness/config-proyecto.md` y referenciarlo desde los ~15 archivos que
   hoy los repiten literal.
5. **7.A.1**, **7.A.3**, **7.A.4** — las tres notas menores de Drupal
   (Recipes/Config Actions como alternativa a evaluar, DTT como camino de
   evolución de testing si aparece un módulo custom, `schema_metatag_ai`
   como opción para el pendiente de `Service` JSON-LD) — ninguna es
   bloqueante, todas son de esfuerzo bajo y sin riesgo.

No se identificó ningún **"bloqueado por falta de capacidad"** durante esta
investigación — toda la verificación (código real del tema, contenido de
las 5 skills instaladas, estado real de `AGENTS.md`, ecosistema Drupal
2026) fue posible con las herramientas ya disponibles para este agente.


## Fuentes

- [platform.claude.com — Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [code.claude.com/docs/en/skills — Extend Claude with skills](https://code.claude.com/docs/en/skills)
- [code.claude.com/docs/en/sub-agents — Subagents (tools por defecto, filtros foreground/background, "Preload skills into subagents")](https://code.claude.com/docs/en/sub-agents)
- [Claude — Skills explained: How Skills compares to prompts, Projects, MCP, and subagents](https://claude.com/blog/skills-explained)
- [Claude — Steering Claude Code: when to use CLAUDE.md, skills, hooks, and subagents](https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more)
- [Anthropic Engineering — Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [GitHub — anthropics/claude-code issue #59968 (limitación de `Agent` anidado dentro de una Skill invocada por un subagente)](https://github.com/anthropics/claude-code/issues/59968)
- [github.com/dianyike/claude-code-insights — subagent-best-practices.md](https://github.com/dianyike/claude-code-insights/blob/main/subagent-best-practices.md)
- [duotach.com — Claude Code Subagents and Skills: Complete Guide](https://duotach.com/en/blog/subagentes-claude-code)
- [totalum.app — Claude Code Skills in 2026: vs Hooks, vs Subagents, vs MCP](https://www.totalum.app/blog/claude-code-skills-totalum)
- [GitHub Docs — Issues GraphQL reference (sub-issues)](https://docs.github.com/en/graphql/reference/issues)
- [GitHub Community — Sub-issues Public Preview](https://github.com/orgs/community/discussions/148714)
- [GitHub — grasmash/drupal-claude-skills (12 skills: ddev, config-mgmt, config-reconcile, contrib-mgmt, at-your-fingertips, testing, simple-oauth, search-api, canvas, canvas-sdc, cursorrules-drupal, skill-developer)](https://github.com/grasmash/drupal-claude-skills)
- [GitHub — addyosmani/web-quality-skills (web-quality-audit, performance, core-web-vitals, accessibility, seo, best-practices)](https://github.com/addyosmani/web-quality-skills)
- [skillsmp.com — drupal-sdc skill (AJV009/drupal-devkit)](https://skillsmp.com/skills/ajv009-drupal-devkit-plugins-drupal-core-skills-drupal-sdc-skill-md)
- [GitHub — vercel-labs/skills (herramienta `npx skills`/`find-skills`)](https://github.com/vercel-labs/skills)
- [drupal.org — Agentic Skills (`ai_skills`, obsoleto/sin soporte — 3 skills, descartado como fuente activa)](https://www.drupal.org/project/ai_skills)
- [Recipe Author Guide — Drupal Recipes Initiative Documentation](https://project.pages.drupalcode.org/distributions_recipes/recipe_author_guide.html) — sección 7.A.1.
- [Use Config Actions in a Recipe — Drupalize.me](https://drupalize.me/tutorial/use-config-actions-recipe) — sección 7.A.1.
- [Use Drupal's Config Actions API to Spice Up Your Recipes — The Droptimes](https://www.thedroptimes.com/54771/use-drupals-config-actions-api-spice-your-recipes) — sección 7.A.1.
- [Getting Started: Configuring Drupal 11 to Apply Recipes](https://project.pages.drupalcode.org/distributions_recipes/getting_started.html) — sección 7.A.1.
- [Schema.org Metatag — drupal.org](https://www.drupal.org/project/schema_metatag) — sección 7.A.4.
- [Schema.org Metatag AI — drupal.org](https://www.drupal.org/project/schema_metatag_ai) — sección 7.A.4.
- [Metatag — drupal.org](https://www.drupal.org/project/metatag) — sección 7.A.4.

## Referencia interna (línea base, no repetida desde cero)

- [harness/README.md](../README.md)
- [harness/opciones-implementacion.md](../opciones-implementacion.md)
- [harness/investigacion/harness-engineering.md](../investigacion/harness-engineering.md)
- [harness/agentes/agente-orquestador.md](../agentes/agente-orquestador.md) — fuente de la restricción `Agent` que la sección 0 extiende por simetría a `Skill`.
- [harness/agentes/agente-documentador.md](../agentes/agente-documentador.md) — fuente del sistema `memoria/largo-plazo/` que la sección 6.4 reutiliza en vez de duplicar.
- [harness/agentes/agente-planificador.md](../agentes/agente-planificador.md) — fuente de la heurística del 60% de contexto usada en la métrica de re-planificación (6.5).
- `.claude/skills/dp.md`, `.claude/skills/dp-cont.md`, `.claude/skills/dp-dev.md`
- `~/.claude/skills/bootstrap-components/SKILL.md`, `~/.claude/skills/drupal-ddev/SKILL.md` — leídos directamente para confirmar contenido real (secciones 6.1).
- `site/web/themes/custom/danemar_theme/danemar_theme.libraries.yml`, `danemar_theme.info.yml`, `package.json` — leídos directamente para confirmar que el tema no usa Bootstrap y que sí usa SASS (build vía `sass`/`scss/` → `css/`, hallazgo honesto: esto contradice la nota "el proyecto no usa SASS" que hoy repiten `agente-frontend.md`/`agente-revisor-pr.md` citando `docs/plans/plan-09-estilos-sass.md` — parece haber quedado desactualizada tras una migración a SASS posterior a ese plan; vale una verificación puntual de `documentador` para corregir esa nota si se confirma, fuera del alcance de este informe pero señalado aquí por honestidad, regla 0.2).
- `harness/agentes/agente-{revisor-pr,estilo,seo,tester,desarrollo-drupal,frontend,documentador,project-writer,investigador-mejoras-harness}.md`
- `.claude/agents/*.md` (13 archivos, frontmatter `tools:` revisado directamente para la sección 0.1)
- `.agents/skills/drupal-ddev/SKILL.md`, `.agents/skills/drupal-config-mgmt/SKILL.md`, `.agents/skills/drupal-at-your-fingertips/SKILL.md` (+ `references/dtt.md`), `.agents/skills/drupal-contrib-mgmt/SKILL.md`, `.agents/skills/drupal-sdc/SKILL.md` — las 5 skills de Drupal instaladas, leídas directamente en esta pasada como fuente de verificación cruzada (sección 7.A).
- `AGENTS.md` (raíz del proyecto) — leído completo para confirmar el hallazgo de la sección 7.B.1 (ubicación fuera de `harness/`/`.claude/`/`site/`, y el token de Cloudflare Tunnel en texto plano en la regla 0.4).
- `site/web/themes/custom/danemar_theme/templates/paragraph/paragraph--hero.html.twig`, `danemar_theme.libraries.yml`, `danemar_theme.info.yml`, y `find` sobre todo `danemar_theme/` (sin resultados para `*.component.yml`) — verificación directa del código real que sustenta el hallazgo de la sección 7.A.2 (no hay SDC implementado en el tema, solo en `diseño/handoff/components/`).
- `harness/agentes/agente-{estilo,creador-contenidos,revisor-pr,project-writer,orquestador,planificador,documentador,historiador,investigador-contenidos,investigador-noticias}.md`, `harness/flujos/*.md`, `harness/memoria/largo-plazo/{decisiones-arquitectura,desarrollo,frontend}.md`, `harness/MANUAL-DE-USO.md` — leídos completos para el barrido de hardcoding de la sección 7.B.
- `site/recipes/` (solo `README.txt` de scaffolding, sin recipes propias) — confirma que Recipes no se usa hoy en el proyecto (sección 7.A.1).
