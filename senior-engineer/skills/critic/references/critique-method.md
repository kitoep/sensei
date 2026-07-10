# Método de crítica

Ejecutar las 5 fases en orden. Ninguna es opcional.

## Fase 1 — Steelman

Antes de criticar, reformula la idea del usuario en su versión MÁS fuerte: completa sus huecos con la interpretación más favorable, agrega los argumentos que él no dio pero que apoyan su plan. Preséntalo en 2-4 líneas al inicio ("Entiendo que propones X porque Y, y el mejor argumento a favor es Z").

Propósito doble: (a) confirmar comprensión — si el steelman está mal, el usuario corrige antes de que critiques un hombre de paja; (b) obligarte a conocer los méritos reales antes de atacar. **Una crítica a un plan que no puedes defender mejor que el usuario es una crítica prematura.**

## Fase 2 — Suposición de carga

Identifica la creencia que, si es falsa, tumba TODO el plan (no un detalle: el cimiento). Ejemplos de forma: "esto asume que los usuarios van a X", "esto asume que el volumen cabe en Y", "esto asume que el equipo puede mantener Z".

Para cada suposición de carga, propón la **prueba más barata** que la valida o refuta antes de construir encima: un experimento de un día, 5 llamadas a clientes, un benchmark de una hora, un prototipo desechable. Si el plan no tiene forma barata de probar su suposición de carga, ESO es un riesgo Critical por sí mismo.

## Fase 3 — Preguntas de demolición

Aplica TODAS al plan; reporta las que produzcan hallazgos:

1. **¿Qué tendría que ser cierto para que esto funcione?** — lista las precondiciones implícitas; las no verificadas son riesgos.
2. **¿Cómo se ve esto fallando en 6 meses?** — escribe el post-mortem imaginario más plausible. Si es fácil de escribir, el riesgo es real.
3. **¿Quién ya intentó esto y qué le pasó?** — patrones conocidos de la industria: si es una idea obvia que nadie hace, averigua por qué antes de asumir que todos son tontos.
4. **¿Qué alternativa más barata logra el 80%?** — si existe y no fue considerada, el plan tiene un problema de costo de oportunidad.
5. **¿Esto resuelve el problema o el síntoma?** — pregunta qué problema origina la petición; si el plan trata la manifestación, dilo.
6. **¿El costo real está completo?** — el precio visible es construirlo; el invisible es mantenerlo, migrarlo, monitorearlo, y lo que NO se construyó por construir esto.

## Fase 4 — Mínimo obligatorio

Toda crítica entrega, como piso:

- **3 riesgos principales** con escenario concreto de cómo muerde cada uno (no "podría haber problemas de escala" sino "con 200 negocios haciendo X a la vez, Y se satura porque Z").
- **1 alternativa no considerada** con sus trade-offs honestos (incluyendo en qué es PEOR que el plan del usuario).

Si tras las fases 1-3 no los tienes, **el problema es tu análisis, no el plan**: vuelve a la fase 3 y profundiza. Declarar un plan sin riesgos está prohibido — todo plan real intercambia algo por algo.

## Fase 5 — Calibración de severidad

Etiqueta cada objeción; no todo pesa igual y mezclar niveles destruye la credibilidad de los graves:

| Etiqueta | Definición operativa | Qué exige |
|---|---|---|
| **FATAL** | Si esto es cierto, el plan no debe ejecutarse en su forma actual. | Bloquea el veredicto "hazlo". Exige resolver o refutar antes de avanzar. |
| **MANEJABLE** | Riesgo real con mitigación conocida y costeable. | Se ejecuta el plan CON la mitigación incorporada. |
| **ESTILO** | Tú lo harías diferente, pero la versión del usuario funciona. | Se menciona una vez, etiquetado como preferencia, y no se insiste. |

Regla anti-inflación: si etiquetas todo FATAL, nada lo es. Máximo honesto, no dramático. Regla anti-deflación: no degradas un FATAL a MANEJABLE para suavizar el veredicto.
