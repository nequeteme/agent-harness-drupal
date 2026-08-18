# Desarrollo Drupal (backend)

Conocimiento consolidado — patrones de modelo de datos y contenido que ya
demostraron ser estables y se repiten, para que `desarrollo-drupal`/
`creador-contenidos`/`planificador` no los re-deriven cada vez.

- **Contenido estructurado como Paragraphs anidados, referenciados desde
  `field_sections` de `landing_page`/`legal_page`.** El modelo de datos real
  del sitio son 14 Paragraph types (`hero`, `service_card`, `process_step`,
  `faq_item`, `client_item`, `legal_section`, etc. — ver
  `harness/analisis-proyecto/estado-actual.md` §3), cada uno con sus propios
  campos, referenciados en orden desde el campo `field_sections` de los
  content types `landing_page`/`legal_page`. No hay un content type genérico
  de "blog/artículo" — cualquier tarea que redacte o modele contenido debe
  encajar en esta estructura de Paragraphs concreta, no inventar un formato
  nuevo sin pasar por `desarrollo-drupal`.

  **Corrección (2026-08-18, misma fecha, pasada posterior): el tema NO usa
  SDC como capa de renderizado — es Twig clásico, decisión deliberada ya
  tomada, no un hueco pendiente.** Una versión anterior de esta entrada
  afirmaba que SDC (Single Directory Components) era la capa de renderizado
  del tema y que "ambas capas coexisten" con los Paragraphs — eso es
  **falso**, verificado directamente contra el filesystem: `find
  site/web/themes/custom/danemar_theme -name "*.component.yml"` no devuelve
  ningún resultado. No hay un solo componente SDC real implementado en el
  tema. Lo que sí existe es:
  - Los Paragraphs siguen siendo la capa de **modelo de datos** (qué campos
    existen, cómo se relacionan) — eso no cambia.
  - El **renderizado real** es Twig **clásico** por bundle de Paragraph
    (`templates/paragraph/paragraph--hero.html.twig`,
    `paragraph--service-card.html.twig`, etc., 14 archivos, uno por tipo),
    sin ninguna invocación de componente SDC (`{{ component(...) }}` o
    equivalente).
  - Esos templates clásicos fueron **traducidos a mano** desde componentes
    SDC que sí existen — pero como mockups de referencia visual en
    `diseño/handoff/components/` (el paquete de diseño, no código que corre
    en el sitio). El propio comentario de cabecera de
    `paragraph--hero.html.twig` lo documenta: *"Traducido de
    `diseño/handoff/components/dp-hero/dp-hero.twig` a template de Paragraph
    clásico"*.
  - Esto es una **decisión deliberada ya tomada**, no un vacío por completar:
    ver `docs/plans/plan-08-componentes-theme.md` ("SDC vs. templates de
    Paragraph clásicos: se mantiene la decisión original... clásicos, sin
    SDC"). Un agente nuevo no debería intentar "editar el componente SDC de
    `service_card`" en el tema — no existe; el template a editar es
    `templates/paragraph/paragraph--service-card.html.twig`.
  - Detalle completo del hallazgo y su verificación:
    `harness/mejoras/2026-08-18-propuestas.md`, sección 7.A.2.

- Ver también `harness/memoria/largo-plazo/frontend.md` para los patrones
  específicos de la capa de renderizado (SDC, custom properties del toggle
  Forge/Dossier, pipeline SASS) que consumen estos mismos Paragraphs.
