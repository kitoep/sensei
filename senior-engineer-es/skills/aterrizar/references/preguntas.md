# Preguntas: cómo interrogar una idea difusa

## Regla 0: contexto primero, siempre

Antes de hacer la primera pregunta, revisa el proyecto real: archivos relevantes, docs, tracker si existe, commits recientes. Dos razones:

1. **No preguntes lo que el proyecto ya responde.** Si el stack, el patrón o la convención ya están decididos, preguntarlos desperdicia turnos y confianza.
2. **Detecta choques temprano.** Si la idea ya existe (total o parcial), o contradice algo construido, dilo ANTES de refinarla: "esto ya existe en X, ¿quieres extenderlo o es otra cosa?".

En escala exprés esta revisión puede ser de un minuto (el archivo que la idea tocaría). Pero nunca es de cero.

## Chequeo de tamaño ANTES de refinar

Con el contexto en mano, pregunta lo primero a ti mismo: ¿esta idea es UNA pieza o varias?

- Si trae varias piezas independientes ("un panel con reportes, notificaciones y permisos"), NO pulas detalles todavía. Primero descompón: cuáles son las piezas, cómo se relacionan, cuál va primero. Luego aterriza la primera pieza con el proceso normal.
- Refinar el detalle de un sistema que había que partir es la forma más cara de perder el tiempo: todas esas respuestas se tiran cuando el alcance cambie.

## Preguntas de una en una

**Prohibido el cuestionario en muro de texto.** Ocho preguntas juntas producen dos respuestas y seis huecos que después rellenas con suposiciones.

- UNA pregunta por mensaje. Si un tema necesita más exploración, son varios mensajes.
- **Opción múltiple preferida**, con tu recomendación marcada: "¿A, B o C? Yo iría por B porque...". Es más fácil de contestar y deja tu criterio sobre la mesa.
- Abierta solo cuando las opciones sesgarían la respuesta (por ejemplo, "¿qué problema te está causando hoy?").
- Cada pregunta debe poder cambiar el resultado. Si la respuesta no altera lo que construirías, no la hagas.

## Las capas (en este orden)

No todas las ideas necesitan todas las capas; la escala exprés suele cubrir 1, 2 y 3. Pero el orden no se invierte: no preguntes el "cómo" antes de tener el "qué" y el "para quién".

| # | Capa | La pregunta de fondo | Por qué va antes que la siguiente |
|---|---|---|---|
| 1 | Problema | ¿Qué problema real resuelve y de quién es ese problema? | Una idea sin problema detrás es una solución buscando justificación; a veces la respuesta honesta es "no vale la pena". |
| 2 | Mínimo útil | ¿Qué es lo más chico que ya sirve y se puede usar? | Define la primera versión real y corta la fantasía de alcance. |
| 3 | Anti-alcance | ¿Qué NO es esto? ¿Qué queda explícitamente fuera? | Lo no dicho es lo que el modelo rellena solo. El "no incluye" evita el scope creep silencioso. |
| 4 | Éxito | ¿Cómo sabremos que funcionó? (métrica, conducta observable, quién lo valida) | Sin criterio de éxito, "terminado" es una opinión. |
| 5 | Restricciones | Datos disponibles, permisos, integraciones, presupuesto, plazos, compliance | Son las condiciones que descartan enfoques enteros; mejor saberlas antes de proponer. |

## Señales de que ya puedes parar de preguntar

- Puedes escribir el resumen de cierre sin inventar nada.
- Las últimas dos respuestas no cambiaron tu entendimiento.
- El anti-alcance tiene al menos un elemento explícito (si nada quedó fuera, no preguntaste suficiente).

Si para escribir el resumen tienes que suponer algo, esa suposición es tu siguiente pregunta.
