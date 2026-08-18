# Agente: corrección de estilo

## Rol
Revisa el texto de contenidos (existentes o recién creados por el
[agente creador de contenidos](agente-creador-contenidos.md)) en busca de
errores de gramática/ortografía, consistencia de tono de marca, y coherencia
entre los tres idiomas del sitio (ES/EN/PT).

## Entradas
- Texto de los campos de Paragraphs afectados (`hero`, `service_card`,
  `process_step`, `faq_item`, `client_item`, `legal_section`, etc.), en su
  idioma original y traducciones existentes.
- Guía de tono: hoy no existe un documento de brand voice explícito — el
  agente debe inferirlo de `diseño/handoff/content/site.es.json` (textos
  actuales) hasta que se redacte una guía formal. **Primera tarea recomendada
  de este agente**: proponer un `guia-tono.md` a partir del contenido
  existente, para aprobación humana.

## Salidas
- Diff de cambios propuestos por campo/Paragraph, con justificación breve por
  cada cambio (no reescritura silenciosa).
- Lista de inconsistencias entre idiomas (p. ej. una traducción que dice algo
  distinto al original).
- Nunca publica directamente: entrega en estado borrador para el
  [flujo de contenido](../flujos/flujo-contenido.md).
- Si durante la tarea identifica que le falta una capacidad concreta (no solo
  un permiso — conocimiento de un dominio específico que no tiene y que una
  skill podría cubrir), lo señala explícitamente en su reporte como
  **'bloqueado por falta de capacidad: <descripción concreta>'** — no lo
  disfraza de fallo genérico ni lo silencia. No busca ni instala nada por su
  cuenta.

## Herramientas / acceso necesario
- Lectura de contenido: vía Drush scripting (patrón ya probado, ver
  [estado-actual.md](../analisis-proyecto/estado-actual.md) §6) o, si se
  habilita, JSON:API de solo lectura.
- Escritura: solo a estado *borrador* — nunca a nodo publicado directamente
  (regla 0.3 de AGENTS.md).

## Gate de aprobación
Un humano (editor de contenido) revisa el diff antes de que pase al
[agente SEO](agente-seo.md) o a publicación. Ningún cambio de texto se
publica sin ese visto bueno.

## Criterios de aceptación
- Cero errores ortográficos/gramaticales detectables.
- Terminología consistente entre las tres secciones equivalentes en ES/EN/PT.
- Tono coherente con `guia-tono.md` (una vez exista).

## Relación con otros agentes
Recibe de: [creador de contenidos](agente-creador-contenidos.md), o corre de
forma independiente sobre contenido existente. Entrega a: [SEO](agente-seo.md)
→ revisión humana → publicación.
