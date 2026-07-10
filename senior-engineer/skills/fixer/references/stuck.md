# Atorado — protocolo de desatore y escalación

Activa este protocolo cuando: 3 hipótesis muertas, >30 minutos sin avance medible, o te descubres re-probando algo que ya probaste.

## Paso 1 — PARA y cuestiona las suposiciones de nivel superior

Cuando todas las hipótesis "razonables" mueren, la falla está en algo que diste por hecho. Verifica MECÁNICAMENTE (no de memoria) cada una:

| Suposición | Cómo verificarla en 30 segundos |
|---|---|
| "Estoy editando el archivo que realmente corre" | Mete un `throw new Error('MARCA-123')` o log único en el archivo; si no aparece al ejecutar, estás editando el archivo equivocado (build viejo, copia, otro paquete del monorepo, dist/ vs src/). |
| "El build está fresco" | Borra el build/cache y recompila (`rm -rf dist .next node_modules/.cache && build`). Los watchers mienten. |
| "Corro la versión/entorno que creo" | Imprime en runtime: versión, commit (`git rev-parse HEAD`), variables de entorno relevantes, URL de DB (host, no credenciales). Compara contra lo que asumías. |
| "El bug está donde estoy mirando" | Loggea el valor sospechoso EN LA ENTRADA del módulo que investigas. Si ya llega corrupto, el bug está aguas arriba y llevas horas mirando el lugar equivocado. |
| "Los datos son los que creo" | SELECT directo a la fila / print del payload real. No el fixture, no el tipo: el dato vivo. |
| "El test que corro es el que creo" | ¿El runner está filtrando/skipeando? ¿Hay dos tests con nombre parecido? Corre con el path completo del archivo. |

El clásico: una hora "debuggeando" un cambio que nunca se estaba ejecutando.

## Paso 2 — Técnicas de desatore

- **Reduce al caso mínimo:** copia el flujo a un script aislado y ve quitando piezas hasta que el bug desaparezca. La última pieza quitada es la culpable (o su interacción). Funciona también al revés: empieza de cero y agrega piezas hasta que aparezca.
- **Compara contra un caso que SÍ funciona:** encuentra el gemelo sano (otro endpoint que sí guarda, otro tenant donde sí llega el mensaje, el mismo flujo en otro entorno). Diffea TODO entre ambos: código, config, datos, headers, logs. La diferencia contiene la causa.
- **Salida limpia vs sucia:** captura la traza/log completo de una corrida buena y una mala y diffealas línea por línea. El primer punto de divergencia delimita el bug mejor que cualquier teoría.
- **Cambia de sensor:** si llevas rato leyendo código, deja de leer y observa el runtime (logs, DB, red, procesos). Si llevas rato en el runtime, lee el código del punto de divergencia. Alternar rompe la ceguera.
- **Re-lee el error original completo** una vez más, literal. Con el contexto acumulado, frases que ignoraste al inicio ahora significan algo.

## Paso 3 — Cuándo y cómo escalar al usuario

Escala cuando: agotaste los pasos 1–2, o necesitas información/acceso que solo el usuario tiene (datos de prod, credenciales, "¿qué hiciste exactamente?"), o el costo de seguir supera el valor (2+ horas sin reducir el espacio del problema).

**Escalar NO es rendirse; seguir quemando tiempo en círculos sí es fallarle al usuario.**

Formato de escalación honesta (obligatorio — nunca escales con "no sé qué pasa"):

```
ATORADO en <bug>. Resumen de lo descartado:

REPRODUCCIÓN: <comando y qué tan determinística es>
HIPÓTESIS MUERTAS:
  1. <hipótesis> — descartada porque <evidencia>
  2. <hipótesis> — descartada porque <evidencia>
  3. <hipótesis> — descartada porque <evidencia>
SUPOSICIONES VERIFICADAS: archivo correcto ✓, build fresco ✓, env ✓, ...
LO QUE SÉ: <hechos confirmados con evidencia>
LO QUE NO SÉ: <la incógnita concreta>
NECESITO DE TI: <pregunta específica o acceso concreto>
SIGUIENTE PASO SI NO HAY RESPUESTA: <qué probarías después>
```

Este resumen convierte tus horas de descarte en valor: el usuario (u otro agente) arranca desde tu frontera, no desde cero.
