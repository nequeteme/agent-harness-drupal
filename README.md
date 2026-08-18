# Harness de agentes (alpha)

Extracción de la arquitectura de agentes (orquestador, planificador,
documentador, 13 agentes de ejecución especializados, flujos de contenido y
desarrollo, sistema de memoria de corto/largo plazo) usada originalmente para
el sitio de [Danemar Parceros](https://danemarparceros.pt/) (Drupal 11).

Este repo contiene **solo el harness** (`harness/` + `.claude/` + `.agents/`
+ este `AGENTS.md`), separado del código del sitio que lo usó como primer
caso real. Es una **versión alpha**: documenta honestamente un caso de uso
concreto (un sitio Drupal institucional en ES/EN/PT), no todavía una
plantilla genérica lista para cualquier proyecto — ver
`harness/mejoras/2026-08-18-propuestas.md`, sección 7 ("Auditoría de
compatibilidad Drupal y portabilidad"), para el detalle de qué es reusable
tal cual y qué sigue acoplado a ese proyecto original.

## Estructura

- `harness/agentes/` — especificación en lenguaje natural de cada agente
  (rol, entradas, salidas, gates de aprobación).
- `harness/flujos/` — los dos flujos principales (contenido, desarrollo+
  testing) que encadenan agentes.
- `harness/memoria/` — sistema de memoria de corto y largo plazo (qué
  funcionó, decisiones de arquitectura, patrones a repetir, métricas).
- `harness/mejoras/` — informes periódicos de investigación sobre cómo
  mejorar el harness mismo.
- `.claude/agents/` — implementación runtime (subagentes de Claude Code) de
  cada agente descrito en `harness/agentes/`.
- `.claude/skills/` — skills de sesión principal (`/dp`, `/dp-cont`,
  `/dp-dev`) más las skills de dominio Drupal/frontend precargadas en los
  subagentes correspondientes.
- `.agents/skills/` — contenido real de las skills instaladas del catálogo
  externo (`.claude/skills/<nombre>` son symlinks hacia acá).
- `AGENTS.md` — reglas base (idioma, honestidad, aprobación previa). La
  sección 0.4 (entorno de pruebas) trae un placeholder — completarla con los
  datos reales de cada proyecto que adopte este harness, sin subir
  credenciales reales a ningún repo compartido.

## Qué falta para ser una plantilla genérica

Ver `harness/mejoras/2026-08-18-propuestas.md` sección 7.B: hay referencias
puntuales al proyecto original (nombre de repo, tema, tablero de GitHub
Projects) repetidas en varios agentes. Portable tal cual hoy; parametrizarlas
en un único archivo de config queda pendiente para cuando exista un segundo
proyecto real que lo justifique.
