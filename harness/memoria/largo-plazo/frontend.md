# Frontend del tema `danemar_theme`

Conocimiento consolidado — patrones de frontend que ya demostraron ser
estables y se repiten, para que `frontend`/`revisor-pr` no los re-deriven
cada vez. El detalle de diseño vivo está en `diseño/handoff/`; esto es la
capa de "por qué está construido así".

- **Toggle Forge/Dossier vía custom properties CSS, no temas/CSS duplicados
  por dirección.** El sitio tiene dos direcciones visuales (Forge, Dossier)
  controladas por la config `danemar_theme.settings:dp_direction`. El
  mecanismo real es un único juego de variables CSS (`--dp-*`, definidas en
  `css/tokens.css`) cuyo valor cambia según la dirección activa — nunca dos
  hojas de estilo paralelas ni un tema Drupal por dirección. Cualquier
  componente nuevo debe consumir esas custom properties (`var(--dp-*)`), no
  hardcodear un color/valor fijo, para que funcione en ambas direcciones sin
  código adicional. Ver `agente-frontend.md` ("nunca hardcodear una sola
  dirección") y el bug real ya resuelto del banner de cookies
  (`harness/historia/2026-08-18-el-banner-de-cookies-que-se-transparentaba.md`),
  que se corrigió cambiando qué custom property se usaba, no agregando CSS
  nuevo por dirección.

- **Pipeline SASS real: `scss/` → `css/` vía Dart Sass CLI, sin Gulp.**
  Migración implementada 2026-08-18 (commits `83e5cf4`/`572209c`, rama
  `feature/sass-migration` del repo `site/`, **todavía sin mergear** — PR
  pendiente al momento de escribir esto). Reemplaza la nota anterior de que
  "el proyecto no usa SASS" (`docs/plans/plan-09-estilos-sass.md`), que había
  quedado desactualizada — ver también la corrección hecha en
  `agente-frontend.md`/`agente-revisor-pr.md` el mismo día.
  - Alcance: solo los 4 archivos propios del sistema de diseño (`tokens`,
    `fonts`, `base`, `sections`, declarados por ruta fija en
    `danemar_theme.libraries.yml`). `css/components/` es CSS vendorizado de
    Drupal core/Classy y **no** entra al build SASS.
  - Comandos: `npm run build:css` (compila `scss/*.scss` → `css/*.css`,
    mismas rutas que antes), `npm run watch:css`, `npm run lint:css`
    (stylelint sobre el `.css` compilado, no sobre `.scss`, por ahora — ver
    nota de abajo).
  - Sin CI en este proyecto: el `.css` compilado **debe quedar commiteado**
    junto con el `.scss` fuente en el mismo commit — nunca solo el fuente.
  - Detalle completo del pipeline y el razonamiento:
    `site/web/themes/custom/danemar_theme/README.md`, sección "CSS build
    (SASS)".

- **Las custom properties `--dp-*` deben seguir siendo CSS real en el
  output compilado, nunca `$variables` de Sass.** El toggle Forge/Dossier
  cambia su valor en tiempo de ejecución (config/JS), no en tiempo de
  compilación — si se convirtieran a variables Sass, se resolverían una sola
  vez al compilar y el cambio de dirección en vivo dejaría de funcionar. Este
  es el motivo concreto por el que la migración a SASS no reemplaza el
  mecanismo del punto anterior, lo preserva tal cual.

- **Pendiente documentado, no implementado todavía**: pasar `lint:css` de
  lintear el `.css` compilado a lintear `scss/**/*.scss` directamente (con
  `stylelint-config-standard-scss`) una vez el contenido real esté migrado a
  `scss/` con nesting/`@use`/partials — el parser CSS plano de
  `stylelint-config-standard` no va a entender esa sintaxis. Recomendación ya
  dejada por el agente `frontend` en el propio README del tema; se evaluará
  al revisar `revisor-pr`, no implementada aún.
