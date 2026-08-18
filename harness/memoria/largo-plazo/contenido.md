# Contenido y tono de marca

Conocimiento consolidado — patrones del flujo editorial que ya demostraron
ser estables y se repiten, para que `creador-contenidos`/`estilo` no los
re-deriven cada vez.

- **Tono de marca ES/EN/PT: redactar primero en español, coordinar
  traducciones después vía `content_translation`, nunca reescribir desde
  cero.** El idioma origen del sitio es ES (`diseño/handoff/content/site.es.json`);
  `creador-contenidos` redacta siempre primero en ese idioma, y las versiones
  EN/PT se generan después como traducción del mismo borrador vía el módulo
  `content_translation` de Drupal — nunca redactando cada idioma desde cero
  de forma independiente, y nunca sin pasar por el flujo de revisión normal
  (`estilo` → `seo` → aprobación humana) para cada versión. Esto evita que
  las tres versiones de un mismo texto diverjan en contenido o mensaje, no
  solo en idioma. Ya documentado como práctica en `agente-creador-contenidos.md`
  (sección "Entradas"/"Cómo operar" según el archivo) — confirmado ahí, no
  es una regla nueva, esta entrada solo lo gradúa a memoria de largo plazo
  para que otros agentes (p. ej. `estilo`, al auditar consistencia entre
  idiomas) lo tengan como contexto sin tener que releer el prompt completo
  de `creador-contenidos`.

- **`guia-tono.md`/`guia-tono-danemar` (guía de tono explícita): todavía no
  existe.** `agente-estilo.md` se auto-asignó como primera tarea proponerla
  a partir de `diseño/handoff/content/site.es.json`. Hasta que exista, el
  tono se infiere de ese mismo archivo de contenido actual. Cuando se
  redacte, considerar el formato de Skill compartida
  (`.claude/skills/guia-tono-danemar/SKILL.md`, precargada en `estilo` y
  `creador-contenidos`) en vez de un documento suelto — ver
  `harness/mejoras/2026-08-18-propuestas.md`, sección 4 Propuesta 3, para el
  razonamiento completo de por qué esa forma es mejor que un `.md` plano.
