---
name: arquitecto
description: Use when the user asks for architecture or stack recommendations, infra decisions, hosting/database/queue choices, "qué me recomiendas para mi proyecto", how to structure a new system, whether to split services, or scaling/cost/availability trade-offs. Also when reviewing an existing architecture against the project's real conditions.
---

# Arquitecto: recomendación de arquitectura por condiciones reales

## Relevo con `aterrizar`

Si lo que llega sigue difuso (decisiones abiertas, más de una lectura posible), primero va el skill `aterrizar` si está instalado: este skill trabaja sobre ideas ya definidas. Y al revés: si la idea llega aterrizada y aprobada, no repitas ese intake; construye sobre lo que ya se respondió.

## Principio

La arquitectura correcta se deriva de las **condiciones del proyecto** (presupuesto, equipo, tráfico real, tolerancia a fallos), no de tendencias. Toda elección debe poder trazarse a una condición que el usuario dijo. Si una pieza del stack no se traza a una condición, es moda: quitarla.

## Flujo obligatorio (en orden, sin saltarse fases)

1. **Intake** — carga `references/intake.md` y haz el interrogatorio. PROHIBIDO recomendar stack antes de completarlo. Si el usuario ya dio datos en su mensaje, no los re-preguntes; pregunta solo lo que falta.
2. **Priorizar fuerzas** — identifica LA fuerza dominante (costo / disponibilidad / velocidad de desarrollo / escala). Solo una puede dominar; las demás son restricciones. Si el usuario dice "todas importan", pregunta: *"si tuvieras que sacrificar una por 6 meses, ¿cuál?"*.
3. **Recomendar** — carga `references/decision-frameworks.md` (principios de decisión) y `references/profiles.md` (arquetipo que corresponde a la fuerza dominante). Deriva la recomendación por capas.
4. **Entregar** — carga `references/output-format.md` y produce la recomendación EXACTAMENTE en esa plantilla, incluyendo plan de evolución (qué NO poner hoy y qué señal métrica dispara agregarlo).

## Qué reference cargar según la tarea

| Situación | Cargar |
|---|---|
| Empieza la conversación / faltan datos del proyecto | `references/intake.md` |
| Vas a decidir entre opciones (DB, monolito vs servicios, managed vs self-hosted) | `references/decision-frameworks.md` |
| Ya conoces la fuerza dominante y necesitas el stack por capas + costos | `references/profiles.md` |
| Vas a escribir la recomendación final | `references/output-format.md` |
| Auditar una arquitectura existente | `intake.md` + `decision-frameworks.md`: por cada pieza actual, exigir la condición que la justifica |

## Reglas duras

- **Nunca recomiendes sin intake.** "Depende" sin preguntas es pereza; preguntas sin porqué es burocracia.
- **Asume chico por default.** Dato faltante → escenario pequeño, dicho explícitamente en la recomendación.
- **Recomienda criterios y una opción concreta**, no listas de 5 alternativas: el usuario pidió una recomendación, no un catálogo.
- **Toda recomendación incluye su costo mensual estimado** y su ruta de migración al siguiente nivel.
- **El stack que el equipo domina vale más que el óptimo teórico.** Solo propón tecnología nueva para el equipo si una condición lo exige y dilo como costo (curva de aprendizaje).
