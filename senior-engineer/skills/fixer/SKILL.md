---
name: fixer
description: Use when debugging any bug, error, test failure, crash, or flaky behavior; when the user reports "no funciona", "sigue fallando", or that something previously claimed as fixed is still broken; before proposing any fix; or when about to declare a bug resolved.
---

# Fixer — debugging sistemático con puertas duras

## Principio

Un bug no está arreglado cuando el código "se ve bien". Está arreglado cuando la reproducción original pasa, existe un test de regresión que falla sin el fix, y el patrón defectuoso fue buscado en todo el repo. Todo lo demás es una opinión.

**Violar la letra de estas reglas es violar su espíritu.** No hay bugs "demasiado simples" para el proceso.

## Flujo de fases (con puertas duras)

Cada fase tiene una PUERTA. No avanzas a la siguiente fase sin cumplirla. Si la saltaste, regresa.

| Fase | Qué haces | PUERTA para avanzar | Reference |
|---|---|---|---|
| 1. Reproducir | Reproduces el bug de forma determinística | Tienes comando exacto + salida del error pegada | `references/diagnosis.md` |
| 2. Diagnosticar | Hipótesis única y falsable + experimento mínimo | Evidencia que confirma TU hipótesis y descarta rivales | `references/diagnosis.md` |
| 3. Causa raíz | Del síntoma a la causa real; clasificar puntual vs clase | Puedes explicar POR QUÉ ocurre, no solo DÓNDE explota | `references/root-cause.md` |
| 4. Fix | Corriges la causa, no el síntoma | El fix ataca la causa raíz identificada en fase 3 | `references/root-cause.md` |
| 5. Verificar | Checklist completo de "arreglado" | Los 5 puntos del checklist con evidencia pegada | `references/verification.md` |

**Carga el reference de la fase en la que estás ANTES de actuar en ella.**

Casos especiales:
- Usuario reporta "sigue fallando" tras un fix tuyo → ve directo a `references/verification.md`, sección "Protocolo sigue-fallando".
- Llevas 3+ hipótesis muertas o >30 min sin avance → carga `references/stuck.md`.

## Red flags — DETENTE si te descubres pensando esto

- "Probablemente es X, déjame cambiarlo y ver" → sin reproducción ni hipótesis falsable. Fase 1.
- "Ya está arreglado" (sin salida de comandos pegada) → no está arreglado. Fase 5.
- "Debería funcionar ahora" → "debería" = no verificaste. Fase 5.
- "Voy a agregar un try/catch aquí" → estás escondiendo el síntoma, no arreglando la causa.
- "Ajusto el test para que pase" → estás borrando la evidencia del bug.
- "Es un caso raro, no vale la pena reproducirlo" → si no se reproduce, no puedes saber que lo arreglaste.
- "A mí me funciona" → el reporte del usuario es correcto hasta demostrar lo contrario.

## Racionalizaciones conocidas

| Excusa | Realidad |
|---|---|
| "El fix es obvio, no necesito reproducir" | Los fixes "obvios" que no atacan la causa real crean el reporte "sigue fallando". Reproducir toma minutos; el falso fix cuesta horas. |
| "No hay tiempo para test de regresión" | Sin test de regresión el bug regresa en el siguiente refactor y pagas todo de nuevo. |
| "El error ya no aparece, listo" | ¿No aparece porque lo arreglaste o porque cambiaste las condiciones? Sin la reproducción original ejecutada, no lo sabes. |
| "Cambié 3 cosas y ya funciona" | No sabes cuál arregló ni qué rompieron las otras dos. Revierte y aplica una por una. |
| "Es flaky, seguro era el entorno" | Flaky = bug de concurrencia/timing/estado hasta demostrar lo contrario. |
