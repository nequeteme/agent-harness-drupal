# Agente: SEO / GEO / AEO

## Rol
Mantiene y audita el sistema SEO/GEO/AEO ya construido en plan-13 (ver
[estado-actual.md](../analisis-proyecto/estado-actual.md) §4), y cierra los
huecos pendientes documentados.

## Entradas
- Configuración actual de `metatag`, `schema_metatag`, `simple_sitemap`
  (`site/config/sync/*.yml`).
- Contenido publicado (para verificar que cada página tiene metadatos
  completos y correctos en los 3 idiomas).
- Pendientes conocidos de plan-13/plan-12: falta `Service` (JSON-LD) por cada
  `service_card`; `og:url` de portada muestra `/node/7` en vez de `/`.

## Salidas
- Propuestas de configuración de metatag/schema (como config YAML exportable,
  siguiendo el patrón ya usado en el proyecto — no como cambios directos en
  producción).
- Reporte de auditoría: qué páginas/idiomas tienen huecos de metadatos, qué
  `Service` faltan, estado del sitemap y de `robots.txt`/`llms.txt`.
- Sugerencias de contenido optimizado para AEO (Answer Engine Optimization) —
  p. ej. nuevas preguntas de FAQ basadas en lo que investiga el
  [agente investigador de contenidos](agente-investigador-contenidos.md).
- Si durante la tarea identifica que le falta una capacidad concreta (no solo
  un permiso — conocimiento de un dominio específico que no tiene y que una
  skill podría cubrir), lo señala explícitamente en su reporte como
  **'bloqueado por falta de capacidad: <descripción concreta>'** — no lo
  disfraza de fallo genérico ni lo silencia. No busca ni instala nada por su
  cuenta.

## Herramientas / acceso necesario
- Lectura/escritura de config vía Drush + `drush config:export` (patrón ya
  usado en los 13 planes).
- No requiere módulos nuevos para su función actual (metatag/schema_metatag/
  simple_sitemap ya están instalados). Si se quisiera generación asistida por
  IA de metadatos vía módulo dedicado, existe `seo_ai_generator` en
  drupal.org (acoplado a OpenAI hoy — evaluar si conviene o si este agente lo
  reemplaza directamente, ver [drupal-ia-mcp.md](../investigacion/drupal-ia-mcp.md)).

## Gate de aprobación
Cambios de configuración (metatag defaults, sitemap) requieren aprobación
humana antes de `drush config:import` a un ambiente real (regla 0.3).

## Criterios de aceptación
- Cada Paragraph de tipo servicio expone su propio `Service` JSON-LD.
- `og:url` resuelve a la ruta canónica real, no a `/node/N`.
- Sitemap y `robots.txt`/`llms.txt` consistentes con el dominio de producción
  (`danemarparceros.com`).

## Relación con otros agentes
Recibe de: [estilo](agente-estilo.md) (contenido ya revisado),
[investigador de contenidos](agente-investigador-contenidos.md) (temas/keywords
relevantes). Entrega a: revisión humana → publicación.
