---
name: dp-cont
description: Orquesta el flujo editorial completo del harness de Danemar Parceros (investigación → creación → estilo → SEO → revisión humana → publicación). Úsalo cuando el usuario pida generar/actualizar contenido del sitio de punta a punta, o invoca /dp-cont.
---

Orquesta el flujo documentado en `harness/flujos/flujo-contenido.md`. Sigue
estos pasos, invocando cada subagente con la herramienta `Agent`
(`subagent_type` = nombre del agente) y pasando explícitamente el output de un
paso como insumo del siguiente — no dejes que un agente "adivine" lo que hizo
el anterior:

1. Si el usuario dio un tema/brief concreto, sáltate a 3. Si no, invoca
   `investigador-noticias` (y opcionalmente `investigador-contenidos`) para
   detectar señales/temas con valor real. No generes contenido por generar.
2. Si el tema amerita profundizar, invoca `investigador-contenidos` con el
   tema/señal detectada.
3. Invoca `creador-contenidos` pasándole el dossier de investigación (o el
   brief del usuario) y la sección/campo concreto del sitio a redactar o
   actualizar.
4. Invoca `estilo` sobre el borrador producido.
5. Invoca `seo` sobre el borrador ya revisado de estilo.
6. **No le pidas la aprobación al usuario vos mismo.** Devuelve el borrador
   final (contenido + metadatos propuestos) al `orquestador` (skill `/dp`) —
   es él quien, siguiendo `harness/agentes/agente-orquestador.md`, decide
   cómo presentarlo y pide la aprobación explícita antes de transicionar el
   `moderation_state` a `published` (regla 0.3 de `AGENTS.md`). Si este flujo
   se invocó directo (sin pasar por `/dp` primero), en este paso actúa vos
   mismo con las reglas de ese documento — el usuario solo debe recibir esa
   pregunta desde el rol de orquestador, nunca desde el flujo en bruto. Si el
   usuario rechaza, vuelve al paso correspondiente según el motivo (forma →
   `creador-contenidos`/`estilo`; fondo → investigación).
7. Invoca `documentador` para registrar lo ocurrido en `harness/memoria/`.

Este flujo nunca le habla al usuario directamente — reporta siempre hacia el
orquestador (real o asumido, ver paso 6).

El estado de contenido (`draft` → `review` → `published`) usa el workflow
"Editorial" de `content_moderation`/`workflows` ya habilitado en el sitio
(ver `harness/analisis-proyecto/estado-actual.md`). Todo agente debe dejar el
contenido en `draft` o `review`, nunca en `published` directamente.
