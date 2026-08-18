# El banner de cookies que se transparentaba

## Qué pasó

El usuario reportó algo vago: "el banner de cookies tiene un background
clarito y no se ve normal, además revisa que funcione en todas las
pantallas y que se mueva bien con el scroll". Ninguna de esas tres cosas
resultó ser lo que sonaba a primera vista.

En vez de adivinar un color y cambiarlo, el agente `frontend` abrió el sitio
real con Playwright, limpió el `localStorage` para forzar que el banner
apareciera, y sacó capturas en tres tamaños de pantalla. Ahí apareció el
problema real: el fondo no era "clarito" — era **3% de opacidad**
(`var(--dp-surface)`, que en el tema oscuro del sitio vale
`rgba(232,237,242,.03)`). Con el banner fijo en la parte de abajo de la
pantalla, el contenido detrás (el título del hero, las preguntas del FAQ) se
veía mezclado con el texto del propio banner — capturas mostraron
literalmente "Presencia en UE y LatAm" atravesando el botón "Aceptar
todas".

Lo del scroll tampoco era un bug de posicionamiento: se midió la posición
del banner antes y después de un scroll grande, y no se movió ni un píxel.
Lo que el usuario percibía como "raro al hacer scroll" era el mismo efecto
de transparencia — el fondo del banner cambiaba de aspecto todo el tiempo
porque literalmente dejaba ver lo que pasaba detrás.

## Qué se aprendió

- **Un reporte de bug en lenguaje humano casi nunca describe la causa real**
  — "se ve raro" resultó ser "3% de opacidad", no un color mal elegido. La
  única forma de saberlo fue mirarlo de verdad en un navegador, no leer el
  CSS y asumir.
- **El bug ya se había resuelto antes, en otro componente, y no se
  reutilizó la lección.** El menú móvil del mismo tema tenía exactamente
  este problema (contenido transparentándose detrás de un overlay fijo), ya
  documentado con un comentario explícito en el propio CSS. El fix del
  banner de cookies terminó siendo aplicar el mismo patrón ya usado ahí
  (`var(--dp-bg)` sólido en vez de un token semitransparente) — la lección
  ya existía en el código, solo no se había propagado.
- **Verificar de más de una forma encuentra cosas que un solo método no
  encuentra.** La verificación final combinó Playwright (visual + medido),
  lectura de config real de Drupal (alternando entre las dos direcciones
  visuales del sitio), revisión de logs, y un caso borde (consentimiento ya
  aceptado) — cuatro ángulos distintos antes de decir "listo".

## Por qué le importaría a alguien de afuera

Es un caso chico y concreto de algo que cualquiera que construye con
agentes de IA se pregunta: ¿de verdad "verifican" o solo dicen que
verificaron? Acá hay una respuesta con evidencia — capturas de pantalla
reales, mediciones de `getComputedStyle`/`getBoundingClientRect`, y un
diagnóstico que terminó siendo distinto de lo que el reporte original
sugería. Es un buen ejemplo chico para un post sobre "cómo se ve un agente
que realmente prueba las cosas antes de decir que están listas".

Evidencia: PR [#2](https://github.com/nequeteme/danemarparceros-site/pull/2)
en `nequeteme/danemarparceros-site`.
