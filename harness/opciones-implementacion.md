# Opciones de implementación del harness

Cuatro formas concretas de construir esto en este proyecto, de menor a mayor
inversión de infraestructura. Ninguna requiere módulos nuevos para
*empezar* — todas pueden arrancar con lo que ya existe (Drush, Playwright,
`AGENTS.md`) y crecer hacia lo demás.

---

## Opción A — Subagentes y skills de Claude Code, orquestación manual

**Qué es**: cada agente del harness (estilo, SEO, creador de contenidos,
investigador de contenidos, investigador de noticias, desarrollo Drupal,
frontend, tester) se define como un subagente de Claude Code
(`.claude/agents/*.md`, igual que `Explore`/`Plan` ya listados en este
entorno) y los flujos como skills (`.claude/skills/dp-cont.md`,
`dp-dev.md`, comandos `/dp-cont`/`/dp-dev`) que invocan a los subagentes en
secuencia. Las
"manos" son Drush scripting (ya probado) + Playwright (ya instalado). El gate
de aprobación humano es, literalmente, el usuario aprobando en la sesión de
Claude Code (regla 0.3 ya funciona así).

| | |
|---|---|
| **Alcance** | Todo lo pedido: 8 agentes + 2 flujos, operando sobre este mismo repo. |
| **Funcionalidades que habilita** | Generación/revisión de contenido asistida, auditoría SEO, desarrollo+testing con loop automático, todo disparable desde una sesión de Claude Code. |
| **Ventajas** | Cero instalación en Drupal. Reutiliza patrones ya validados (Drush, Playwright, `AGENTS.md`). Más rápido de tener funcionando (días, no semanas). Iteración y ajuste de prompts es trivial (son archivos Markdown en el repo). |
| **Desventajas** | No corre "solo" — necesita que alguien abra una sesión de Claude Code para disparar cada paso (aunque `ScheduleWakeup`/`/loop` permiten pacing autónomo dentro de una sesión activa). No hay UI para que un editor no técnico use los agentes sin pasar por Claude Code. Memoria/estado limitados a archivos del repo, no a un sistema de moderación nativo de Drupal. |
| **Esfuerzo estimado** | Bajo. Escribir 8 archivos de subagente + 2 skills, y (si se quiere estado borrador real) habilitar `content_moderation`+`workflows` (core, sin composer). |

---

## Opción B — Nativo en Drupal: módulos `ai` + `ai_agents` + `mcp_server`

**Qué es**: instalar el ecosistema de IA de Drupal directamente en el sitio
(`ai`, `ai_agents`/`ai_agents_ossa`, `mcp_server`, `simple_oauth`), de forma
que los agentes vivan **dentro** de Drupal, disparados por
`content_moderation` transitions o por ECA (Event-Condition-Action), y
accesibles desde el propio panel de administración. Ver
[drupal-ia-mcp.md](investigacion/drupal-ia-mcp.md).

| | |
|---|---|
| **Alcance** | Los agentes de desarrollo/config (Field Type Agent, Content Type Agent, Taxonomy Agent de `ai_agents`) llegan "de fábrica"; el resto (estilo, SEO, contenido, noticias) habría que construirlos como agentes custom sobre `ai_agents_ossa` + ECA. |
| **Funcionalidades que habilita** | Cualquier editor de contenido (sin acceso a Claude Code) puede disparar un agente desde la UI de Drupal. Automatización real vía cron/ECA sin depender de una sesión externa abierta. Sienta base para el roadmap de IA 2026 de Drupal core. |
| **Ventajas** | Integrado al producto: sobrevive a este proyecto, disponible para cualquier futuro editor humano. Usa la gobernanza nativa de Drupal (permisos, content_moderation) como guardrail, alineado con el patrón "Outside AI" de 5 etapas. |
| **Desventajas** | `ai_agents_ossa`, `mcp_server` y el meta-módulo experimental están en madurez temprana — sin versión/compatibilidad confirmada con Drupal 11.4 de este sitio (habría que probarlo). Requiere gestionar API keys de proveedor de modelo dentro de Drupal (superficie de seguridad nueva). Curva de aprendizaje de ECA + `ai_agents` para el equipo. Más piezas que mantener/actualizar a futuro. |
| **Esfuerzo estimado** | Alto. Instalación, configuración de OAuth, pruebas de compatibilidad, y aún así hay que construir 5 de los 8 agentes a medida. |

---

## Opción C — Harness externo tipo `deepseek-harness`, con MCP como puente

**Qué es**: construir un runtime de agentes independiente (fuera de Drupal,
p. ej. como servicio Node/Python en este mismo repo o en uno nuevo), con
arquitectura de plugins (inspirada en el patrón "everything is a plugin" de
DeepSeek — ver [harness-engineering.md](investigacion/harness-engineering.md)
§3), que se conecta a Drupal vía el módulo `mcp_server` (Drupal expone
nodos/Paragraphs como tools MCP) en vez de Drush directo.

| | |
|---|---|
| **Alcance** | Los 8 agentes como plugins independientes del proveedor de modelo, reusable en otros proyectos futuros (no solo Danemar Parceros). Soporta ejecución programada real (cron del runtime, no depende de una sesión interactiva). |
| **Funcionalidades que habilita** | Pipeline de contenido/noticias corriendo de forma desatendida 24/7. Desacoplamiento total del cliente (podría usarse desde Claude Code, Claude Desktop, o cualquier cliente MCP). Reutilizable como producto/oferta propia de Danemar Parceros hacia otros clientes Drupal (encaja con su negocio real). |
| **Desventajas** | Mayor esfuerzo de ingeniería: hay que construir y mantener el runtime del harness (no existe uno "de caja" maduro y estable — `deepseek-harness` mismo está en developer preview). Requiere habilitar `mcp_server`+OAuth en Drupal igualmente (mismo riesgo de madurez que la Opción B en esa capa). Sobre-ingeniería para el tamaño actual del sitio (una landing de una página, sin volumen de contenido que justifique un runtime propio hoy). |
| **Esfuerzo estimado** | Muy alto. Semanas, no días; solo se justifica si el objetivo real es un producto reutilizable, no solo automatizar este sitio. |

---

## Opción D — Híbrida (recomendada)

**Qué es**: empezar con la **Opción A** para todo lo que se beneficia de
supervisión humana estrecha y acceso amplio al repo (creación de contenido,
estilo, SEO, desarrollo, frontend, testing — que son 7 de los 8 agentes
pedidos), y reservar automatización nativa de Drupal (`content_moderation` +
`workflows`, ambos de core, sin instalar nada de composer) **solo** para dar
al contenido un estado borrador/revisión/publicado real. No instalar
`ai_agents`/`mcp_server` todavía — quedan documentados como camino de
evolución (Opción B) para cuando exista necesidad real de que un editor no
técnico dispare agentes sin pasar por una sesión de Claude Code, o de que el
investigador de noticias corra desatendido con una cadencia fija.

| | |
|---|---|
| **Alcance** | Los 8 agentes y los 2 flujos, operativos en días, con estado borrador real vía `content_moderation`. |
| **Ventajas** | Menor riesgo (nada nuevo que mantener en producción todavía). Reutiliza 100% de lo ya probado en los 13 planes (Drush, Playwright, `AGENTS.md`). Deja una puerta clara hacia B o C cuando el proyecto lo justifique, sin haber tirado trabajo. |
| **Desventajas** | Mismas limitaciones de la Opción A mientras no se de el salto: nadie fuera de una sesión de Claude Code puede disparar un agente; el investigador de noticias no corre 24/7 sin que alguien mantenga una sesión activa o configure `/loop`/cron externo. |
| **Esfuerzo estimado** | Bajo-medio. Igual que A, más habilitar y configurar `content_moderation`/`workflows` (core). |

---

## Resumen comparativo

| | A. Claude Code | B. Nativo Drupal | C. Harness externo | D. Híbrida |
|---|---|---|---|---|
| Instalación en Drupal | Ninguna (opcional core) | Alta (varios contrib) | Media (mcp_server) | Ninguna (opcional core) |
| Corre desatendido 24/7 | No (requiere sesión) | Sí | Sí | No (igual que A) |
| Usable por editor no técnico | No | Sí | Parcial (vía cliente MCP) | No |
| Madurez de las piezas usadas | Alta (todo probado en este proyecto) | Baja-media (módulos nuevos/experimentales) | Baja (deepseek-harness en preview) | Alta |
| Reutilizable en otros proyectos | Parcial | Parcial | Alta | Parcial |
| Tiempo a primera versión funcionando | Días | Semanas | Semanas-meses | Días |

**Recomendación**: Opción D. Es la que más rápido entrega los 8 agentes y los
2 flujos pedidos, con el menor riesgo, y no cierra la puerta a evolucionar
hacia B o C más adelante si el volumen de contenido o la necesidad de
autonomía 24/7 lo justifican.

---

## Camino de evolución: de D a B

D no es una alternativa aparte de B — está diseñada a propósito como su base.
Nada de lo que se implemente en D bloquea ni hay que deshacer para llegar a
B, pero tampoco es gratis: hay trabajo real de por medio.

**Se reutiliza sin cambios**

- `content_moderation` + `workflows` (core), ya habilitados en D, son
  exactamente el mismo gate de estado borrador → revisión → publicado que usa
  el diagrama de B (ver [mapas-arquitectura.md](mapas-arquitectura.md)). No
  hay que tocarlos.
- El contenido ya creado bajo moderación en D (borradores, historial de
  revisiones) sigue siendo válido — no hay migración de datos.
- Las especificaciones de cada agente en [agentes/](agentes/) (entradas,
  salidas, criterios de aceptación, gates) son el *diseño*, independiente del
  runtime — sirven igual para escribir un subagente de Claude Code o una
  acción ECA.

**No se reutiliza — hay que rehacerlo**

- La orquestación misma: un subagente `.md` de Claude Code y un plugin de
  `ai_agents`/acción ECA son dos runtimes distintos. Pasar de D a B implica
  reimplementar la lógica de cada agente en PHP/ECA, no solo mover un
  archivo.
- Instalar y configurar `ai`, `ai_agents`/`ai_agents_ossa`, `mcp_server`,
  `simple_oauth` (composer + config nueva), y verificar su compatibilidad
  real con Drupal 11.4 (sigue sin confirmar — ver
  [drupal-ia-mcp.md](investigacion/drupal-ia-mcp.md)).
- Gestión de API keys del proveedor de modelo *dentro* de Drupal (superficie
  de seguridad nueva que hoy no existe).
- Definir los triggers de ECA/cron que reemplacen al "humano corre una sesión
  de Claude Code".

**No es todo o nada**: la migración más probable es parcial — dejar
desarrollo/frontend/testing en Claude Code (se benefician de acceso completo
al repo) y mover solo los agentes de contenido/noticias a B cuando de verdad
se necesite que corran 24/7 sin sesión abierta.
