---
name: dp-dev
description: Orquesta el flujo de desarrollo+testing del harness de Danemar Parceros (desarrollo/frontend → testing en loop → commit+push+PR → revisor de PRs → merge humano). Úsalo cuando el usuario pida implementar y verificar un cambio de código hasta que funcione correctamente, o invoca /dp-dev.
---

Orquesta el flujo documentado en `harness/flujos/flujo-desarrollo-testing.md`.
Desde 2026-08-18 este flujo termina siempre en un Pull Request revisado, no
en un commit directo a `develop`.

1. Confirma que la tarea tiene aprobación explícita del usuario para
   implementar (regla 0.3 de `AGENTS.md`) — si no la tiene, detente y
   pregunta antes de invocar `desarrollo-drupal`/`frontend`.
2. Según la naturaleza del cambio, invoca `desarrollo-drupal` (backend:
   content types, campos, config, hooks) y/o `frontend` (templates/CSS/JS del
   tema), en una rama de trabajo creada desde `develop` (nombre descriptivo:
   `fix/...` o `feat/...`).
3. Invoca `tester` sobre el resultado.
4. Si `tester` reporta fail: vuelve al agente que implementó con la evidencia
   concreta del fallo, y repite 2-3. Este loop puede iterar libremente sin
   intervención humana — el usuario no debería ver una iteración fallida.
5. **Si `tester` reporta pass: nunca dejes la tarea sin commit.** Commitea
   los cambios en la rama de trabajo (mensaje en inglés, regla 0.5), pushea a
   `origin` (`git@github.com:nequeteme/danemarparceros-site.git`), y abrí un
   PR contra `develop` con `gh pr create` (título/descripción en inglés,
   resumen de la verificación del tester incluido).
6. Invoca `revisor-pr` sobre el PR recién abierto (corre `phpcs`/`stylelint`
   + revisión razonada, comenta en el PR).
7. Si `revisor-pr` reporta hallazgos bloqueantes: vuelve al agente que
   implementó (nuevo commit en la misma rama — el PR se actualiza solo),
   repetí tester si el fix pudo afectar comportamiento, y volvé a
   `revisor-pr`.
8. Si `revisor-pr` no tiene hallazgos bloqueantes: **no le pidas la
   aprobación al usuario vos mismo, y NUNCA ejecutes `gh pr merge`.**
   Devolvé el link del PR + el resumen de tester/revisor al `orquestador`
   (skill `/dp`) — es él quien avisa al usuario que el PR está listo. Si
   este flujo se invocó directo (sin pasar por `/dp` primero), en este paso
   actuá vos mismo con las reglas de `harness/agentes/agente-orquestador.md`
   — el usuario solo debe recibir ese aviso desde el rol de orquestador,
   nunca desde el flujo en bruto. **El merge lo hace el usuario manualmente,
   nunca un agente.**
9. Invoca `documentador` para registrar lo ocurrido en `harness/memoria/`
   (incluido el número/link del PR).

Este flujo nunca le habla al usuario directamente — reporta siempre hacia el
orquestador (real o asumido, ver paso 8).
