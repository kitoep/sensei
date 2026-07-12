---
name: arranque
description: Use when starting a new project or repository from scratch, setting up a greenfield codebase, scaffolding a new app or service, or when the user asks "cómo empiezo", "arranca el proyecto", "new project", or wants to begin building something that doesn't exist yet. Also use before writing the first line of code in an empty or near-empty repo.
---

# Project Bootstrap — arranque de proyectos que no se caen después

## Principio

Casi todos los proyectos que colapsan a los 3-6 meses murieron en la primera semana: se saltaron preguntas, cimientos o el esqueleto andante. Este skill existe para que NO escribas código el día 1. El orden de fases es obligatorio y cada fase tiene un gate: no avanzas sin completarlo.

**Violar el orden de fases "porque el proyecto es simple" es la causa #1 de retrabajo.** Un proyecto simple completa las fases en una hora; uno complejo, en un día. Ninguno las salta.

## Fases (en orden, con gates)

| Fase | Qué haces | Reference a cargar | Gate para avanzar |
|---|---|---|---|
| 0. Intake | Preguntas al usuario ANTES de decidir nada | `references/intake.md` | Respuestas registradas en `docs/00_intake.md` y confirmadas por el usuario |
| 1. Cimientos | Decisiones baratas hoy, carísimas después | `references/foundations.md` | Checklist de cimientos aplicado o descartado ítem por ítem con razón escrita |
| 2. Esqueleto andante | Ruta end-to-end desplegada antes que features | `references/walking-skeleton.md` | El esqueleto corre en un entorno desplegado y CI está en verde |
| 3. Gobernanza | CLAUDE.md + docs para que agentes no degraden el proyecto | `references/docs-setup.md` | CLAUDE.md y estructura de docs commiteados |

## Reglas duras

1. **Cero código antes de terminar la Fase 0.** Ni scaffold, ni `npm init`, ni "voy adelantando el repo". Si el usuario pide código directo, responde: "Antes necesito 5-10 respuestas que cambian la arquitectura; sin ellas voy a construir lo incorrecto" y arranca el intake.
2. **Las preguntas del intake se hacen de una en una** (o en grupos de máximo 3 con opciones). No aviente el cuestionario completo.
3. **Cada ítem de cimientos se decide explícitamente.** "Aplicar" o "descartar con razón escrita". Nunca se omite en silencio — lo omitido en silencio es lo que revienta en el mes 4.
4. **El primer deploy va antes que la primera feature.** Si no hay pipeline de deploy al terminar la Fase 2, la fase no está terminada.
5. **Ninguna fase se declara terminada sin su evidencia**: intake escrito, checklist con decisiones, esqueleto desplegado y verificado, docs commiteados.

## Señales de que estás violando el skill (detente)

- "Es un proyecto chico, empiezo por el CRUD y luego ordeno" → el CRUD fácil primero es el anti-patrón #1 (ver walking-skeleton.md).
- "El deploy lo vemos al final" → el deploy al final SIEMPRE revela problemas de config/secretos/migraciones cuando ya es caro.
- "Luego agrego los tests / el CI" → un proyecto sin CI desde el commit 1 acumula deuda invisible que nadie paga voluntariamente.
- "Multi-tenant lo agregamos si hay más clientes" → multi-tenancy no se retrofitea; se decide en el intake (ver foundations.md).
- "Ya sé lo que quiere el usuario, me salto el intake" → lo que crees saber y lo que el usuario confirma difieren en el 100% de los casos en al menos un punto que cambia la arquitectura.

## Salida esperada del skill completo

Al terminar las 4 fases el repo tiene: `docs/00_intake.md` (respuestas), `docs/01_decisiones.md` (cimientos aplicados/descartados con razón), esqueleto andante desplegado con CI verde, `CLAUDE.md` con las reglas no-negociables del proyecto, y un tracker con los primeros 5 milestones ordenados por riesgo.
