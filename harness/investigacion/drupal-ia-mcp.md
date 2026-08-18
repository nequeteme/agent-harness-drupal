# Investigación: Drupal + agentes de IA

> Fuente: investigación web + `drupal.org` (agosto 2026). Ninguna versión ni
> compatibilidad exacta con Drupal 11.4 (la versión de este sitio) fue
> verificada por instalación real — confirmar al momento de instalar cualquiera
> de estos módulos.

## 1. Iniciativa oficial de IA de Drupal

Dries Buytaert publicó un roadmap 2026 con ocho capacidades de IA para Drupal
core/ecosistema: agentes en background reaccionando a triggers/schedules,
integración con design system, creación/descubrimiento de contenido,
gobernanza avanzada, campañas multicanal, entre otras. Objetivo declarado:
"democratizar" expertise de SEO, marca y accesibilidad dentro del flujo
editorial normal — no como plugin externo, sino como parte de Drupal.

Fuente: [Drupal's AI Roadmap for 2026](https://www.drupal.org/blog/drupals-ai-roadmap-for-2026)

## 2. Módulos contrib relevantes (no instalados en este proyecto hoy)

| Módulo | Qué hace | Relevancia para el harness |
|---|---|---|
| [`ai`](https://www.drupal.org/project/ai) | Capa de abstracción de proveedores de modelos (48+, incluye Anthropic). Requiere Drupal 10.5/11.2+ según su documentación. | Sería el punto de entrada si se quisiera correr agentes **dentro** de Drupal en vez de externos. |
| [`ai_agents`](https://www.drupal.org/project/ai_agents) | Framework de agentes "text-to-action" sobre la config de Drupal: trae de fábrica Field Type Agent, Content Type Agent, Taxonomy Agent. | Útil para un agente de *desarrollo Drupal* que modifique config con supervisión, no para contenido editorial. |
| `ai_agents_ossa` ("Open Standard Agents") | Tokens custom para prompts, integración con ECA (Event-Condition-Action) para workflows de agentes, orquestación vía MCP, políticas Cedar. | Es lo más cercano a "harness dentro de Drupal": ECA como motor de orquestación. |
| `ai_agents_experimental_collection` | Meta-módulo instalador de agentes individuales (`ai_agents_views`, `ai_agents_content_types`, etc.). | Aún experimental — evaluar madurez antes de depender de él en producción. |
| [`mcp_server`](https://www.drupal.org/project/mcp_server) | Convierte el sitio Drupal en un endpoint MCP: expone nodos/vistas/Drush como "tools" vía OAuth 2.1. | Camino más limpio para que un harness **externo** (Claude Code, otro cliente MCP) opere sobre el contenido sin necesitar Drush/SSH. |
| [`mcp_client`](https://www.drupal.org/project/mcp_client) | Conecta Drupal, como cliente, a servidores MCP externos. | Relevante solo si se quisiera que Drupal mismo consuma herramientas externas (p. ej. un servicio de noticias). |
| [`mcp`](https://www.drupal.org/project/mcp) | Proyecto original, en proceso de fusión hacia `mcp_server`. | Verificar cuál de los dos está activo al momento de instalar. |
| `seo_ai_generator` | Generación de metadatos SEO vía API de OpenAI. | Acoplado a OpenAI específicamente; este proyecto usa Anthropic — evaluar si conviene o si el agente SEO propio lo reemplaza. |
| [`simple_oauth`](https://www.drupal.org/project/simple_oauth) | OAuth2 para JSON:API/REST. | Módulo estándar para autenticar cualquier agente externo que necesite escribir contenido vía API. |
| [`rest_api_authentication`](https://www.drupal.org/project/rest_api_authentication) | Alternativa/complemento de autenticación para REST. | — |

## 3. "Outside AI — The State of Agent Experience in Drupal"

Artículo de drupal.org que identifica gaps reales al conectar agentes de IA a
un sitio Drupal ya existente: fricción de setup/autenticación, falta de
inventario legible por máquina de estructura/permisos/acciones, fragilidad de
configuración bypasseable por código custom, y falta de verificación
independiente de lo que el agente afirma haber hecho (exactamente el tipo de
problema que [AGENTS.md](../../AGENTS.md) 0.2 ya intenta prevenir a nivel de
proceso humano).

Propone una arquitectura en 5 etapas, directamente aplicable a este proyecto:

1. **Conectar** con identidad delegada y auditable (no una cuenta admin
   compartida).
2. **Entender** el sitio vía discovery legible por máquina (qué content types,
   campos, permisos existen — hoy eso vive en `config/sync/*.yml` y en
   [estado-actual.md](../analisis-proyecto/estado-actual.md)).
3. **Actuar** por interfaces gobernadas con inputs tipados y "recibos"
   estructurados — "un modelo de acción, muchas puertas" en vez de acceso
   directo a la base de datos.
4. **Verificar y recuperar**, aprovechando los flujos draft/review/approve
   nativos de Drupal (`content_moderation` + `workflows`).
5. **Reconstruir/migrar** con auditoría de paridad (no aplica hoy — es para
   sitios grandes en migración).

Fuente: [Outside AI: The State of Agent Experience in Drupal](https://www.drupal.org/about/ai/initiatives/blog/outside-ai-the-state-of-agent-experience-in-drupal)

## 4. Qué expone Drupal hoy en este proyecto

Confirmado en `site/web/core.extension.yml` y `site/composer.json`: `jsonapi`,
`rest`, `content_moderation` y `workflows` **no están habilitados**. La única
vía de automatización ya probada en este proyecto es **Drush scripting**
(`drush php:script -`), usada extensivamente en los planes 06 y 08 para crear
content types y cargar contenido mediante las APIs nativas de Drupal. Ver
[estado-actual.md](../analisis-proyecto/estado-actual.md) para el detalle.

Para que agentes externos (fuera de una sesión con acceso directo a
DDEV/Drush) interactúen con el sitio, hace falta uno de:

- Habilitar `jsonapi`/`rest` (core) + `simple_oauth` (contrib), o
- Instalar `mcp_server` como puente directo hacia un cliente MCP (Claude Code,
  Claude Desktop, etc.).

Ambas opciones son evaluadas en [opciones-implementacion.md](../opciones-implementacion.md).
