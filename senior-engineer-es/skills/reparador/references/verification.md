# Verificación (Fase 5) — el estándar de "arreglado"

**Un fix sin este checklist completo NO se declara arreglado.** Se declara "cambio aplicado, verificación pendiente" — que es otra cosa y el usuario merece saber la diferencia.

## Checklist obligatorio (los 5 puntos, con evidencia)

### (a) La reproducción original ahora pasa
Ejecuta EXACTAMENTE la reproducción de la Fase 1 (mismo comando, mismos datos). Pega la salida real en tu reporte. "Corrí el repro y ahora responde 200 con el body esperado: [salida]". Si cambiaste el repro para que pase, violaste la prohibición #3 de diagnosis.

### (b) Test de regresión que falla sin el fix
1. Escribe el test que captura el bug.
2. **Pruébalo contra el bug:** revierte el fix temporalmente (`git stash` del fix, o comenta las líneas), corre el test, confirma que FALLA con el error original. Pega esa salida.
3. Restaura el fix, corre el test, confirma que PASA. Pega esa salida.

Un test de regresión que nunca viste fallar no prueba nada: puede estar testeando otra cosa, o nada (falso verde).

### (c) La suite completa pasa
No solo tu test nuevo: la suite entera del área afectada (idealmente todo el proyecto). Los fixes rompen cosas vecinas. Pega el resumen (`X passed, 0 failed`). Si algo falla, tu fix no está listo — no importa que "parezca no relacionado": demuéstralo o arréglalo.

### (d) El efecto real verificado en el estado
El exit code 0 y el HTTP 200 no son el efecto. Verifica el estado real que el bug corrompía:
- Bug de datos → consulta la fila en DB y pega el resultado.
- Bug de envío duplicado → cuenta los mensajes/jobs realmente creados (1, no 2).
- Bug de archivo → muestra el contenido del archivo generado.
- Bug de concurrencia → corre el escenario N veces / N requests paralelos y pega el conteo (exactamente 1 ganó, 20/20 corridas limpias).

### (e) Patrón simétrico buscado
Resultado de la búsqueda del patrón (ver root-cause.md): comando de búsqueda ejecutado + ocurrencias encontradas + qué se hizo con cada una. "Busqué `<patrón>` en todo el repo: 3 hits; corregí 2, el tercero no aplica porque X".

## Frases prohibidas sin evidencia adjunta

| Frase | Solo puedes decirla si… |
|---|---|
| "Ya está arreglado" | …los 5 puntos del checklist tienen salida pegada. |
| "Debería funcionar" | Nunca. O verificaste y "funciona", o no verificaste y lo dices: "apliqué el cambio, falta verificar X". |
| "El problema era X" | …tienes el experimento que confirma X y descarta las rivales (Fase 2). Si no, di "mi hipótesis es X". |
| "Los tests pasan" | …los corriste en ESTA sesión, después del último cambio, y pegas el resumen. |
| "No debería afectar nada más" | …corriste la suite completa (punto c). |

La honestidad sobre lo NO verificado no es debilidad: "arreglado y verificado (a)-(d); (e) pendiente porque el repo es enorme, patrón descrito para buscarlo" es un reporte profesional. "Ya quedó ✅" sin evidencia es el reporte que genera el mensaje "sigue fallando".

## Protocolo "sigue fallando"

Cuando el usuario reporta que algo que declaraste arreglado sigue roto:

1. **El reporte del usuario es correcto por defecto.** Prohibido responder "a mí me funciona", "ya lo arreglé, prueba de nuevo" o re-afirmar el fix sin nueva evidencia.
2. **Vuelve a Fase 1 con EL CASO DEL USUARIO**, no con tu repro anterior. Que tu repro pase y el caso del usuario falle significa que tu repro no capturaba el bug real (o hay un segundo bug). Pide/extrae los detalles exactos: qué hizo, con qué datos, qué vio, en qué entorno/commit.
3. **Audita tu verificación anterior:** ¿cuál de los 5 puntos del checklist saltaste o hiciste débil? Esa brecha es tu primera hipótesis. ¿Verificaste el status pero no el estado (d)? ¿El test de regresión nunca lo viste fallar (b)?
4. Considera hipótesis de segundo orden: fix correcto pero no desplegado (build viejo, cache, otro branch), fix correcto para UNA causa de un síntoma con DOS causas, fix que introdujo una regresión nueva con el mismo síntoma.
5. Al re-declarar arreglado, el checklist se corre COMPLETO otra vez, ahora incluyendo el caso exacto del usuario como reproducción.
