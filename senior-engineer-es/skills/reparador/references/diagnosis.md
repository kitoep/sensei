# Diagnóstico sistemático (Fases 1–2)

## Fase 1 — REPRODUCIR PRIMERO (puerta dura)

**Regla:** si no puedes reproducir el bug de forma determinística, NO tienes permiso de proponer fixes. Ninguno. Tampoco "fixes preventivos por si acaso".

Cumples la puerta cuando tienes las dos cosas:
1. **Comando exacto** que dispara el bug (o script/test que lo dispara), ejecutable por cualquiera.
2. **Salida real del error pegada** (stack trace, respuesta HTTP, estado incorrecto en DB) — no una descripción de memoria.

Procedimiento:
1. Ejecuta el caso que reporta el usuario TAL CUAL lo describe, con sus datos si los dio.
2. Si reproduce: congela la reproducción como script o test (`repro.sh`, test `.skip`-eado, curl guardado). Esta reproducción es tu criterio de éxito en Fase 5.
3. Si NO reproduce: la diferencia entre tu entorno y el del usuario ES la primera pista. Compara: versión/commit, variables de entorno, datos, permisos/rol del usuario, zona horaria, estado previo. No sigas hasta cerrar esa brecha o pedir al usuario el dato faltante exacto.

### Reproducir lo difícil

| Tipo de bug | Técnica |
|---|---|
| Concurrencia (doble reserva, race) | Dispara N requests simultáneos al mismo recurso (`Promise.all` de N llamadas, o `xargs -P`); el bug aparece cuando >1 gana. Repite 20+ veces: una corrida limpia no prueba nada. |
| Timing/flaky | Corre el test en loop hasta fallar (`for i in $(seq 50); do npm test -- -t "nombre" || break; done`). Agrega latencia artificial (sleep en el punto sospechoso) para ampliar la ventana de carrera. |
| Dependiente de estado | Captura el estado exacto previo (dump de las filas involucradas, fixture). El bug que "solo pasa a veces" suele ser "solo pasa con este estado". |
| Solo en prod | Replica los datos (anonimizados) y la config de prod localmente. Si es imposible, agrega logging dirigido (ver bisección) y espera la siguiente ocurrencia — pero dilo explícitamente, no adivines. |
| Dependiente de fecha/hora | Fija el reloj (mock de `Date.now`, `jest.useFakeTimers`, variable TZ). Bugs de "solo falla en fin de mes / cambio de horario" se reproducen fijando esa fecha. |

## Fase 1.5 — Leer el error DE VERDAD

Antes de hipotetizar:

1. **Lee el mensaje completo**, palabra por palabra. No lo que esperas que diga: lo que dice. El 30% de los bugs se resuelven aquí.
2. **Lee el stack completo**, no solo el primer frame. Busca el frame más profundo que sea TU código (no framework).
3. **Busca la PRIMERA causa, no la última.** En logs con múltiples errores, el último suele ser consecuencia (conexión cerrada, transacción abortada). Scrollea hacia arriba hasta el primer error de la cadena; ese es el que diagnósticas.
4. **Errores encadenados:** `caused by`, `errno`, códigos internos (`P2002`, `ECONNRESET`, `EADDRINUSE`) — busca el código exacto en la doc de la herramienta antes de teorizar.

## Fase 2 — Hipótesis única y falsable

**Regla:** UNA hipótesis a la vez, escrita antes de tocar código, con el experimento que la mata.

Formato obligatorio (escríbelo literalmente en tu razonamiento o en un comentario de avance):

```
HIPÓTESIS: el hold expirado no se libera porque el job de expiración usa la
           fecha del servidor y no UTC.
PREDICE:   si creo un hold con expiración en UTC-6 pasada, el job NO lo libera.
EXPERIMENTO: crear hold con expiresAt = now()-1h en UTC, correr job, ver fila.
SI FALLA LA PREDICCIÓN: la hipótesis está muerta; NO la parcho con sub-cláusulas.
```

Reglas:
- El experimento debe poder **matar** la hipótesis. Si cualquier resultado la "confirma", no es un experimento.
- Experimento mínimo: el cambio/consulta más pequeño que distingue tu hipótesis de las rivales. Un `SELECT`, un log, un test unitario — no un refactor.
- Hipótesis muerta = muerta. Anótala en una lista de descartadas (con la evidencia) y formula la siguiente. Esta lista es oro cuando escales o te atores.
- **Confirmada ≠ terminado.** Confirmada te pasa a Fase 3 (causa raíz), no al fix.

## Bisección — cuando no hay hipótesis clara

Corta el espacio del problema a la mitad repetidamente:

- **En el tiempo:** `git bisect start; git bisect bad HEAD; git bisect good <último-commit-bueno>` + tu script de reproducción como criterio. Encuentra el commit culpable en log₂(n) pasos.
- **En el código:** comenta/desactiva la mitad del flujo (middleware, hooks, pasos del pipeline). ¿Sigue el bug? La causa está en la mitad activa. Repite.
- **En los datos:** ¿falla con el dataset completo? Prueba con la mitad. Encuentra el registro exacto que dispara el bug.
- **Con logging dirigido:** loggea el valor sospechoso en 3 puntos del flujo (entrada, transformación, salida). El punto donde el valor se corrompe delimita la mitad culpable. Borra estos logs al terminar.

## Prohibiciones absolutas en diagnóstico

1. **No cambies cosas "a ver si funciona".** Cada cambio sin hipótesis contamina el estado y puede enmascarar el bug.
2. **No agregues try/catch para que "no truene".** Silenciar el síntoma no es arreglar; es esconder el cadáver.
3. **No toques el test que falla para que pase** (ni el assert, ni el timeout, ni `.skip`). El test es la evidencia.
4. **No declares causa raíz sin evidencia que la distinga de las rivales.** "Seguro es X" con dos hipótesis vivas = no sabes cuál es.
5. **No arregles dos cosas a la vez.** Un fix por causa, verificado por separado.
