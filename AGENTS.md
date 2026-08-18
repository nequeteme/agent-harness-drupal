# AGENTS.md

Instrucciones generales para cualquier agente (humano o IA) que trabaje en este
repositorio. Deben cargarse y respetarse automáticamente antes de realizar
cualquier tarea.

## 0.1 Idioma

- Toda la conversación con el usuario (explicaciones, preguntas, resúmenes) va en
  ESPAÑOL.
- Todo el código va en INGLÉS: nombres de variables, funciones, clases, métodos,
  rutas, nombres de tablas/columnas, mensajes de commit y comentarios en código.
  Ningún identificador ni comentario en español, sin excepción (a diferencia de
  otros proyectos donde el vocabulario de dominio se conserva en español, aquí
  todo va en inglés salvo que el usuario indique lo contrario para un término sin
  traducción razonable).
- Si hay conflicto entre "código legible" y "todo en inglés", gana "todo en
  inglés".

## 0.2 Honestidad y transparencia (obligatorio)

- No afirmar que algo "funciona" o está "completado" sin haberlo verificado
  (correr el sitio, correr tests, revisar el resultado real). Si no se verificó,
  decirlo explícitamente: "esto no lo he probado todavía".
- No inventar módulos, APIs de Drupal, hooks, campos de configuración ni
  comportamiento que no se haya confirmado en la documentación oficial o en el
  código del proyecto. Si no se está seguro, decirlo y ofrecer verificarlo antes
  de continuar.
- No ocultar ambigüedades: si una instrucción admite más de una interpretación,
  explicitar la ambigüedad, presentar opciones con pros/contras, recomendar una y
  esperar confirmación antes de implementar.
- No "tragarse" errores: si un comando falla, un módulo no se instala, un test no
  pasa, reportarlo tal cual (no silenciarlo con un try/catch vacío ni seguir como
  si nada). Esto aplica también al código: prohibido un `catch` que solo loguea y
  continúa sin propagar o manejar el error de verdad.
- Si algo hecho en una sesión anterior resulta estar roto o mal documentado,
  decirlo y corregir el registro (memoria/plan) en vez de dejarlo inconsistente.
- Cuando algo está totalmente funcionando es que funciona al 100%; si no está
  funcionando al 100% hay que corregir el fallo.

## 0.3 Aprobación antes de implementar

- No se implementa código (módulos custom, tema, configuración exportada,
  migraciones) sin aprobación explícita del usuario ("implementa", "hazlo",
  "adelante" o equivalente).
- Análisis, diseño, investigación de módulos/enfoques y documentación NO
  requieren aprobación previa — eso sí se puede hacer libremente.
- Señales de detención: "quiero que analices/documentes/investigues/expliques",
  "no implementes todavía", "solo quiero entender", "ayúdame a planear".

## 0.4 Entorno de pruebas (URL pública + túnel)

Sección específica de cada proyecto concreto que use este harness — en el
proyecto original (Danemar Parceros) documenta la URL pública de pruebas y el
comando (con token real de Cloudflare Tunnel) para levantar el túnel. Ese
comando/token **no viaja en esta copia** por ser una credencial real; se
reemplaza acá por un placeholder de referencia:

```
cloudflared tunnel run --token <TOKEN_REAL_NO_INCLUIDO_EN_ESTE_REPO>
```

Al adoptar este harness en un proyecto nuevo, completar esta sección con los
datos reales de ese proyecto (URL de pruebas, comando de túnel si aplica) —
y, si el archivo llega a contener un secreto real otra vez, mantenerlo fuera
de cualquier repo que se comparta o publique.

## 0.5 Commits tras implementar un plan

- Después de implementar el código de un plan (o de una parte funcional de un
  plan), hacer commit al branch `develop` — no dejar cambios sin commitear de
  una tarea ya completada y verificada.
- Si `develop` no existe todavía, crearlo antes del primer commit de
  implementación.
- No commitear código a medio terminar ni que no se haya verificado
  (ver 0.2): primero verificar que funciona, después commitear.
- Mensajes de commit en inglés (ver 0.1).
