---
name: planeacion
description: Use when planning a new feature, project, or significant change before writing code — when the user says "cómo lo hacemos", "planea esto", "quiero construir X", asks for an implementation plan, or is about to start anything non-trivial without a written plan. Also use when a task keeps growing mid-implementation, a symptom that planning was skipped.
---

# Planeacion: planear como un ingeniero senior, no como un generador de listas

## Relevo con `aterrizar`

Si lo que llega sigue difuso (decisiones abiertas, más de una lectura posible), primero va el skill `aterrizar` si está instalado: este skill trabaja sobre ideas ya definidas. Y al revés: si la idea llega aterrizada y aprobada, no repitas ese intake; construye sobre lo que ya se respondió.

## Principio

Un plan no es una lista de pasos: es un conjunto de **decisiones tomadas con información que hoy es barata y mañana será cara**. El 80% del valor de planear está en las preguntas que haces ANTES de escribir el plan; un plan escrito sin preguntar solo documenta suposiciones.

**Regla de oro:** si no puedes decir qué pregunta le hiciste al usuario y qué respuesta cambió el plan, no planeaste — adivinaste.

## Flujo obligatorio

1. **Entender el contexto existente** — explora el código, docs y tracker del proyecto ANTES de preguntar nada. Nunca preguntes algo que el repo ya responde.
2. **Preguntar** — carga `references/questions.md` y recorre las 5 capas. De UNA pregunta por mensaje, opción múltiple cuando se pueda. Detente cuando las respuestas dejen de cambiar el plan.
3. **Descomponer y ordenar** — carga `references/decomposition.md`. Tareas con entregable verificable, incertidumbre primero.
4. **Escribir el plan** — carga `references/plan-template.md` y llena la plantilla exacta. Pasa el checklist de auto-revisión antes de presentarlo.
5. **Aprobación del usuario** — el plan no se ejecuta hasta que el usuario lo apruebe. Si pide cambios, vuelve al paso que corresponda.

## Qué reference cargar

| Momento | Archivo |
|---|---|
| Antes de hacer la primera pregunta | `references/questions.md` |
| Al partir el trabajo en tareas | `references/decomposition.md` |
| Al redactar el plan final | `references/plan-template.md` |

## Señales de que estás fallando

- Escribiste un plan sin hacer ni una pregunta → borra el plan, vuelve al paso 2.
- El plan tiene tareas tipo "avanzar en X" o "mejorar Y" → no son verificables, re-descompón.
- Preguntas 6 cosas en un solo mensaje → el usuario contesta 2 y pierdes las otras 4.
- El usuario dice "haz lo que creas mejor" a todo → presenta 2-3 opciones con tu recomendación y trade-offs; una recomendación concreta obtiene mejor información que una pregunta abierta.
- Descubres a media implementación algo que una pregunta de la capa (c) o (d) habría revelado → registra la pregunta faltante; el costo de esa omisión es el argumento para no saltarte este flujo la próxima vez.
