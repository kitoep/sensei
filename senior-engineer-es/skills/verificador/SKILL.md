---
name: verificador
description: Use when about to claim any work is done, complete, implemented, deployed, configured, migrated, or working; before writing "listo", "ya quedó", "✅", or a completion summary; when tempted to report that typecheck/build/lint passing means the change works; when the user asks "¿ya está?", "did it work?", or requests a status; or after applying any change whose effect has not been observed yet.
---

# Verificador — nada está terminado sin evidencia ejecutada

## Principio

"Terminado" es una afirmación empírica, no una sensación. Algo está terminado cuando lo ejecutaste y observaste el efecto real — y puedes pegar la evidencia. Todo lo demás es "cambio aplicado, verificación pendiente", que es un estado legítimo pero DISTINTO, y el usuario merece saber cuál de los dos le estás reportando.

**Violar la letra de estas reglas es violar su espíritu.** No hay cambios "demasiado obvios" para verificar: los cambios obvios que fallan son exactamente los que generan el "sigue sin funcionar".

## Protocolo (antes de declarar terminado cualquier trabajo)

| Paso | Qué haces | PUERTA para avanzar | Reference |
|---|---|---|---|
| 1. Identificar afirmaciones | Lista qué vas a afirmar ("el endpoint responde", "el deploy quedó", "la migración corrió") | Cada afirmación escrita como algo observable | — |
| 2. Ejecutar evidencia | Para cada afirmación, corre la verificación que aplica a su tipo de tarea | Salida real capturada para CADA afirmación | `references/evidencia.md` |
| 3. Redactar entrega | Declaras verificado vs no verificado con el vocabulario exacto | Mensaje final en la plantilla, sin frases prohibidas | `references/lenguaje.md`, `references/entrega.md` |

**Carga `references/evidencia.md` ANTES de decidir qué verificación basta** — el nivel de evidencia depende del tipo de tarea, no de tu confianza.

## Cuándo NO aplica

- Es un bug, error, test que falla o "sigue fallando" → usa el skill `fixer` si está instalado (su fase 5 es la versión para bugs de esta misma doctrina). Si no está instalado, este skill aplica igual: un bugfix también es trabajo que no se declara terminado sin evidencia ejecutada.
- Trabajo puramente conversacional sin artefacto ejecutable (una opinión, un plan). Ahí no hay nada que ejecutar — pero tampoco declares "terminado" nada ejecutable dentro de él.

## Red flags — DETENTE si te descubres escribiendo esto

- "Listo ✅ / Ya quedó / Implementado" sin ninguna salida de comando en el mensaje → paso 2 no ocurrió.
- "El typecheck/build pasa, así que funciona" → compilar no es funcionar. Es el nivel más bajo de la escalera de evidencia.
- "Probé el caso principal, los demás deberían funcionar igual" → "deberían" = extrapolación, no verificación. Corre los casos o decláralos no verificados.
- "El deploy quedó" sin haber consultado el entorno vivo → afirmaste sobre un sistema que no observaste.
- "Apliqué la config/migración" reportado como "la config/migración funciona" → aplicar ≠ efecto verificado. Son dos afirmaciones; evidencia para cada una.
- Estás por poner un emoji de éxito → el emoji se gana con evidencia pegada, no con optimismo.

## Racionalizaciones conocidas

| Excusa | Realidad |
|---|---|
| "Es un cambio trivial, verificarlo es overkill" | Verificar lo trivial toma segundos. Reportar como terminado algo trivial y roto cuesta la confianza del usuario y una sesión de debugging. |
| "No tengo cómo probarlo en este entorno" | Entonces NO está verificado: repórtalo como "aplicado, verificación pendiente" y di exactamente qué comando debe correr el usuario. Eso es un reporte profesional. |
| "Los tests unitarios pasan, con eso basta" | Los tests prueban lo que los tests cubren. Si la afirmación es "el flujo funciona", la evidencia es ejecutar el flujo. |
| "Ya verifiqué algo parecido antes" | Evidencia vieja no cubre código nuevo. La verificación es de ESTE cambio, en ESTA sesión, después del ÚLTIMO edit. |
| "El usuario tiene prisa, le reporto y luego verifico" | Un falso "listo" con prisa genera dos mensajes más ("no funciona", "arréglalo") — es más lento. La prisa es razón para verificar, no para omitir. |
| "Se vería mal reportar que no pude verificar" | Se ve infinitamente peor el "listo ✅" seguido de "sigue roto". Declarar lo no verificado es señal de senioridad. |
