# Análisis del proyecto Danemar Parceros (para el diseño del harness)

## 1. Qué es el sitio

Danemar Parceros es una **agencia de desarrollo Drupal** para instituciones y
empresas, con sede en Portugal, orientada a UE y LatAm, en ES/EN/PT
(`diseño/handoff/content/site.es.json`). El sitio actual es una **landing de
una página** (no un blog/portal de noticias), construida en Drupal 11 con
Paragraphs anidados.

Esto importa para el harness: los agentes de "investigador de noticias" y
"creador de contenidos" no tienen hoy un content type de tipo *artículo/post*
donde publicar — su output inicial encajaría mejor como insumos para
actualizar las secciones existentes (Servicios, Proceso, FAQ) o como base para
una futura sección de blog/insights, que **no existe todavía**.

## 2. Estado de los 13 planes

Todos marcados **Completed** en `docs/plan-memoria.md`, sitio listo para
revisión previa a producción (no lanzado). Pendientes menores documentados
(no ocultos, por regla 0.2 de AGENTS.md):

| Plan | Pendiente relevante para el harness |
|---|---|
| 06 Arquitectura de contenido | Borrado de nodos no hace cascada a Paragraphs huérfanos (riesgo conocido) |
| 07 Idiomas | Sin detección automática de idioma por navegador en `/` |
| 13 SEO/GEO/AEO | Falta `Service` (JSON-LD) por cada `service_card` — los Paragraphs no tienen campo de metatag propio |
| 12 QA | `og:url` de portada muestra `/node/7` en vez de `/` |
| 11 Contacto | Email destino sigue apuntando a dominio `.pt`, no `.com` |

Los dos primeros pendientes de la tabla (13 y 12) son candidatos directos y
concretos para el **agente de SEO** una vez exista.

## 3. Arquitectura de contenido (confirmado en `site/config/sync`)

Modelo: landing de una página, sin content types por sección — decisión
deliberada documentada en plan-06 (el paquete de diseño original asumía
content types separados Servicio/Cliente/Paso; se descartó a favor de
Paragraphs).

| Content type | Campos | Traducible |
|---|---|---|
| `landing_page` | `field_sections` (Entity Reference Revisions multivalor → Paragraphs) | Sí |
| `legal_page` | `field_sections` (idem) | Sí |

14 Paragraph types (`config/sync/paragraphs.paragraphs_type.*.yml`): `hero`,
`hero_fact`, `service_card`, `services_section`, `process_step`,
`process_section`, `client_item`, `clients_section`, `tech_stack_section`,
`faq_item`, `faq_section`, `contact_section`, `legal_section`,
`legal_document`.

Sin taxonomías custom (solo el vocabulario `tags` por defecto del perfil
`standard`, sin uso confirmado). `field_icon` es `string` simple, no
referencia a Media (decisión deliberada para no instalar el módulo Media sin
acordarlo).

## 4. SEO/GEO/AEO — lo que ya existe

- `metatag` + `metatag_open_graph` + `metatag_twitter_cards` — activos.
- `schema_metatag` + `schema_organization` + `schema_qa_page` +
  `schema_web_site` — activos → JSON-LD `ProfessionalService` (global) y
  `FAQPage` (portada, 3 idiomas).
- **Hallazgo curioso**: el FAQ de la portada (`metatag.metatag_defaults.front.yml`,
  campo `schema_qa_page_main_entity`) ya incluye una pregunta redactada
  explícitamente sobre "harness engineering" (agentes de corrección de estilo,
  validación de datos, control de alcance) — el propio sitio ya le anuncia al
  público la existencia de este proyecto que se está diseñando aquí.
- `simple_sitemap` configurado con hreflang y `base_url` de producción.
- `web/robots.txt` permite explícitamente bots de IA: `GPTBot`, `CCBot`,
  `OAI-SearchBot`, `ChatGPT-User`, `Claude-SearchBot`, `Claude-User`,
  `PerplexityBot`.
- `web/llms.txt` estático, escrito a mano (se descartó módulo generador por
  baja adopción).

## 5. Módulos instalados (`site/composer.json`)

`paragraphs`, `honeypot`, `captcha`+`image_captcha`, `pathauto`, `metatag`,
`schema_metatag`, `simple_sitemap`, `admin_toolbar`. Ningún módulo de IA, MCP,
moderación de contenido ni workflows. `jsonapi`, `rest`, `content_moderation`,
`workflows` existen en core pero **no están habilitados**.

## 6. Automatización ya probada: Drush scripting

No hay API externa habilitada, pero sí un patrón de automatización ya usado
con éxito en los planes 06 y 08: `drush php:script -` para crear content
types, campos y cargar contenido mediante las APIs nativas de Drupal
(`\Drupal::entityTypeManager()`, etc.), corrido dentro de DDEV. Es la vía más
barata y ya validada para que un agente de desarrollo/contenido actúe sobre
este sitio sin necesitar módulos nuevos.

## 7. Testing ya probado: Playwright + Chromium

Usado en plan-12 (QA/lanzamiento) para verificación real del sitio (no solo
asumir que algo "funciona" — regla 0.2). Ya instalado y disponible. Es la base
natural del **agente tester**.

## 8. Theme y frontend

`site/web/themes/custom/danemar_theme/`: `templates/paragraph/*` (uno por
Paragraph type), `css/tokens.css` + `base.css` + `sections.css` + `fonts.css`
(dos direcciones visuales Forge/Dossier vía config
`danemar_theme.settings:dp_direction`), `js/mobile-menu.js` y
`js/cookie-consent.js` (sin dependencias externas), `src/Hook/` (hooks en
clase — patrón moderno de Drupal 11, no `.module` procedural). El paquete
`diseño/handoff/` (fuera del repo git) sigue siendo la fuente de verdad visual.

## 9. Búsqueda de IA/MCP/harness en el repo — negativo

Búsqueda exhaustiva (case-insensitive) de "ai / mcp / agent / harness / llm /
openai / anthropic / claude" en `docs/`, `diseño/` y `site/` (excluyendo
core/contrib/vendor): sin integración funcional. Las únicas apariciones son
menciones documentales en los propios planes (discusión de bots de IA para
SEO, mención de skill `drupal-claude-skills`, la pregunta de FAQ ya
mencionada). **Es terreno completamente virgen** para lo que se diseña en esta
carpeta `harness/`.

## 10. Reglas de proceso que todo agente del harness debe respetar

De [AGENTS.md](../../AGENTS.md) — resumen orientado a agentes automatizados:

- **0.1** Documentación/conversación en español; todo el código (variables,
  funciones, rutas, campos, commits, comentarios) en inglés sin excepción.
- **0.2** Prohibido afirmar "funciona"/"listo" sin verificación real. Prohibido
  inventar módulos/hooks/APIs no confirmados. Prohibido silenciar errores.
- **0.3** Ningún agente implementa código/config sin aprobación humana
  explícita. Análisis, investigación y documentación son libres. → Esto define
  el **gate de aprobación** de todos los flujos diseñados en esta carpeta.
- **0.4** Entorno de pruebas: `https://site-dev.danemarparceros.net/` vía
  túnel Cloudflare hacia `localhost:33000`.
- **0.5** Commits a `develop` solo después de verificar; mensajes en inglés.
