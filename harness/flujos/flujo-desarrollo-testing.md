# Flujo de desarrollo y testing

Flujo para cambios de código/config (backend o frontend), hasta que "todo
funcione correctamente" antes de darse por terminado — este es literalmente
el pedido del usuario y coincide con la regla 0.2 de AGENTS.md. Desde
2026-08-18, este flujo termina siempre en un **Pull Request revisado**, no en
un commit directo — el merge final lo hace el usuario.

```
┌─────────────────────────┐   ┌──────────────────────────┐
│ Desarrollo Drupal        │   │ Frontend                  │
│ (agente-desarrollo-      │   │ (agente-frontend.md)      │
│ drupal.md)                │   │                            │
└────────────┬──────────────┘   └────────────┬───────────────┘
             │  implementa en rama de trabajo │
             └────────────────┬───────────────┘
                               ▼
                  ┌──────────────────────────┐
                  │ Tester                    │
                  │ (agente-tester.md)        │
                  │ Drush + Playwright contra │
                  │ site-dev.danemarparceros  │
                  └────────────┬───────────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
                  FAIL                  PASS
                    │                     │
                    ▼                     ▼
        vuelve al agente que      ┌──────────────────────────┐
        implementó, con           │ Commit + push + PR         │  ← SIEMPRE,
        evidencia concreta        │ (regla nueva: ninguna       │    sin excepción
        del fallo                 │ tarea termina sin esto)     │
                                   └────────────┬─────────────────┘
                                                │
                                                ▼
                                   ┌──────────────────────────┐
                                   │ Revisor de PRs              │
                                   │ (agente-revisor-pr.md)      │
                                   │ phpcs + stylelint + revisión │
                                   │ razonada, comenta en el PR   │
                                   └────────────┬─────────────────┘
                                                │
                                     ┌──────────┴──────────┐
                                     │                     │
                          hallazgos bloqueantes    sin hallazgos bloqueantes
                                     │                     │
                                     ▼                     ▼
                       vuelve al agente que    ┌──────────────────────────┐
                       implementó (nuevo        │ Orquestador avisa al      │
                       commit en la misma       │ usuario: PR listo          │
                       rama, re-tester si        └────────────┬─────────────┘
                       afecta comportamiento)                │
                                                              ▼
                                                 ┌──────────────────────────┐
                                                 │ USUARIO hace el merge      │  ← gate humano,
                                                 │ (nunca un agente)           │    regla 0.3/0.5
                                                 └────────────┬─────────────────┘
                                                              │
                                                              ▼
                                    ┌──────────────────────────┐   ┌──────────────────────────┐
                                    │ Documentador                │   │ Historiador                  │
                                    │ (registra en memoria/,      │   │ (si hay una historia real,   │
                                    │ siempre)                     │   │ registra en historia/)       │
                                    └──────────────────────────┘   └──────────────────────────┘
```

`project_writer` sigue la tarjeta correspondiente en el tablero
("site danemar") a lo largo de todo este flujo: `Backlog` al crearse →
`Ready`/`In progress` mientras se implementa/testea → `In review` con el PR
abierto → `Done` al mergear. Ver
[agente-project-writer.md](../agentes/agente-project-writer.md).

> Desde que existen [orquestador](../agentes/agente-orquestador.md) y
> [planificador](../agentes/agente-planificador.md), este flujo normalmente
> se dispara desde ellos (ver [flujo-orquestacion.md](flujo-orquestacion.md))
> en vez de invocarse directo — aunque invocarlo directo sigue siendo válido
> para tareas puntuales.

## Regla de ciclo

El loop `implementar → testear → corregir` se repite tantas veces como haga
falta **antes** de abrir el PR — el usuario no debería ver una iteración
fallida, solo la versión ya verificada por el tester. Esto reduce ruido y
respeta 0.2 ("cuando algo está funcionando es que funciona al 100%").

**"PASS" en este diagrama significa Done al 100%, verificado por el tester
con más de un método** (Drush/SQL, Playwright, revisión de logs/regresiones —
ver [agente-tester.md](../agentes/agente-tester.md)). No hay paso a PR con un
resultado parcial; si algún criterio de aceptación no se cumple al 100%, es
FAIL y vuelve a implementación, sin excepción.

## Regla nueva: ninguna tarea termina sin commit

Una vez que el tester da PASS, la tarea **siempre** se cierra con commit +
push + PR — nunca se deja una rama de trabajo con cambios verificados y sin
subir. Esto es así incluso si el usuario no pidió explícitamente "comitea
esto": terminar una tarea de desarrollo implica dejarla en un PR, salvo que
el usuario haya dicho explícitamente que no lo haga.

## Regla nueva: branches + PR, nunca commit directo a `develop`

Desde 2026-08-18 (a pedido explícito del usuario), este flujo ya no comitea
directo a `develop`:

1. El agente que implementó (`desarrollo-drupal`/`frontend`) commitea sus
   cambios en la rama de trabajo (creada desde `develop`, nombre descriptivo
   tipo `fix/lo-que-sea` o `feat/lo-que-sea`), en inglés (regla 0.1/0.5 de
   AGENTS.md).
2. Push de la rama a `origin` (remoto: `git@github.com:nequeteme/danemarparceros-site.git`).
3. Se abre un PR contra `develop` con `gh pr create` — título y descripción
   en inglés, incluyendo qué cambió y un resumen de la verificación del
   tester.
4. El [revisor de PRs](../agentes/agente-revisor-pr.md) corre `phpcs`
   (estándares Drupal/DrupalPractice) sobre PHP y `stylelint` sobre CSS, más
   revisión razonada, y comenta en el PR.
5. Si hay hallazgos bloqueantes: vuelve al agente que implementó, nuevo
   commit en la misma rama (el PR se actualiza solo), re-tester si el fix
   pudo afectar comportamiento, y de vuelta al revisor.
6. Cuando el revisor no tiene hallazgos bloqueantes: el orquestador le avisa
   al usuario que el PR está listo, con el link.
7. **El usuario hace el merge.** Ningún agente ejecuta `gh pr merge` ni
   comitea directo a `develop` — es el único paso de este flujo reservado
   estrictamente a una acción humana manual fuera del control de los
   agentes.

Esto reemplaza, para este proyecto, la lectura literal de la regla 0.5 de
AGENTS.md ("commit al branch develop") por su versión con revisión: el
commit llega a `develop` igual, pero pasando por PR + revisor antes del
merge humano.

## Disparadores

- Un plan nuevo en `docs/plans/` (patrón ya usado en las 13 sesiones previas
  del proyecto).
- Un hallazgo de otro agente (p. ej. el [agente SEO](../agentes/agente-seo.md)
  pide un campo `Service` por `service_card` → tarea de
  [desarrollo Drupal](../agentes/agente-desarrollo-drupal.md)).
- Un pedido directo del usuario.

## Diferencia clave con el flujo de contenido

Este flujo tiene **un gate humano al final** (el merge del PR), pero el loop
de corrección con el tester y con el revisor de PRs puede iterar libremente
sin intervención humana — el usuario solo ve la versión ya verificada y
revisada, lista para mergear.
