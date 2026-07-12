---
name: aterrizar
description: Use when anything about to be built is not 100% defined, at ANY stage of a project; when a requirement, feature, or idea has decisions still open or admits more than one interpretation; when you are about to FILL A GAP WITH YOUR OWN ASSUMPTION while implementing (catching yourself writing "asumo que..." or choosing something the user never decided); and BEFORE planning or coding anything not fully defined. Vague phrasing is just one signal (e.g. "estaría bueno que...", "se me ocurrió...", "quiero algo así como..."); the trigger is the underlying condition, not the wording.
---

# Aterrizar: de idea difusa a definición concreta

## Principio

Una idea vaga implementada es una apuesta: el modelo rellena los huecos con sus suposiciones y el usuario descubre al final que construyó otra cosa. Aterrizar cuesta minutos; construir lo equivocado cuesta la sesión entera. Este skill convierte lo no definido en una definición que el usuario aprobó, ANTES de que exista plan o código.

**El disparador es la condición, no la frase.** No esperes a que el usuario diga "estaría bueno que..." o "algo así como...": si lo que se va a construir tiene decisiones abiertas o admite más de una lectura, el skill aplica, lo haya dicho bonito o no.

**Violar la letra de estas reglas es violar su espíritu.** No hay ideas "suficientemente claras" que se salten el aterrizaje: claras para ti no es lo mismo que definidas por el usuario.

## REGLA QUE NO SE SALTA

**Prohibido escribir código o plan mientras la idea siga difusa.** Se aterriza primero, el usuario aprueba el resultado (resumen exprés o diseño), y solo entonces se avanza. Aplica en cualquier momento del proyecto: una idea suelta a mitad del desarrollo se aterriza igual que un proyecto nuevo.

## Auto-detección: tu suposición ES el disparador

Vigílate a ti mismo mientras trabajas. En el momento en que estás por rellenar un hueco con una decisión que el usuario nunca tomó (te descubres escribiendo "asumo que...", "voy a interpretar que...", o simplemente eligiendo tú), ese momento ES el disparador del skill, aunque el usuario no haya dicho ninguna frase especial ni parezca "idea nueva".

Distingue dos tipos de hueco:

| Tipo de suposición | Ejemplos | Qué haces |
|---|---|---|
| **Trivial** (reversible, no cambia el producto) | Nombre de una variable, orden de campos en un log, un default técnico estándar | La asumes y la DECLARAS en tu entrega ("asumí X, cámbialo si quieres") |
| **De producto o comportamiento** | Qué hace la feature, para quién, qué pasa en el caso X, qué se guarda, qué ve el usuario final | SIEMPRE se pregunta o se aterriza (escala exprés). Nunca se decide en silencio. |

La prueba rápida: si el usuario viera tu suposición y pudiera decir "no, eso no era lo que quería", no es trivial.

## Las dos escalas

| Escala | Cuándo | Qué se hace | Salida |
|---|---|---|---|
| **Exprés** | Idea chica a mitad de proyecto, ajuste, feature acotado | 3 a 5 preguntas de una en una + chequeo de tamaño | Resumen de ~5 líneas aprobado por el usuario |
| **Completo** | Feature grande, proyecto nuevo, algo que toca varias piezas | Proceso entero: contexto, preguntas por capas, 2-3 enfoques, diseño por secciones | Diseño aprobado sección por sección |

Si dudas cuál aplica, empieza en exprés: si las respuestas destapan más piezas, escala a completo y dilo.

## Protocolo (carga el reference ANTES de actuar en su fase)

| Fase | Qué haces | PUERTA para avanzar | Reference |
|---|---|---|---|
| 1. Contexto | Revisas el proyecto real (archivos, docs, historia) antes de preguntar nada | Sabes qué existe ya y qué patrones sigue el proyecto | `references/preguntas.md` |
| 2. Interrogar | Chequeo de tamaño, luego preguntas de una en una por capas | El problema, el mínimo útil, el anti-alcance y el criterio de éxito están respondidos | `references/preguntas.md` |
| 3. Enfoques | Propones 2-3 caminos distintos con trade-offs y tu recomendación | El usuario eligió un enfoque (o pidió mezclar con conocimiento de causa) | `references/enfoques.md` |
| 4. Cierre | Resumen exprés o diseño por secciones + auto-revisión + aprobación | El usuario aprobó el texto final, sin TBDs ni ambigüedades | `references/cierre.md` |

En escala exprés las fases 2 y 4 bastan (con la 1 siempre); la 3 se usa solo si hay más de un camino razonable.

## Red flags: DETENTE si te descubres escribiendo esto

- "Asumo que...", "voy a interpretar que...", o elegir tú algo que el usuario no decidió → LA red flag central: acabas de encontrar el hueco. Si no es trivial (ver tabla de auto-detección), se pregunta o se aterriza, no se rellena.
- "¡Excelente idea! Empiezo a implementarla" → validaste sin cuestionar y saltaste el aterrizaje completo.
- Ocho preguntas en un solo mensaje → muro de texto; el usuario contesta dos y las demás se pierden. De una en una.
- "Ya con esto tengo suficiente contexto" sin haber revisado el proyecto → tu contexto es la conversación; el proyecto real puede contradecirla.
- Estás puliendo el detalle de un botón cuando la idea trae cinco subsistemas → primero descomponer, luego refinar.
- El resumen final dice "por definir" en algún punto → no está aterrizado; esa línea es la siguiente pregunta.

## Racionalizaciones conocidas

| Excusa | Realidad |
|---|---|
| "La idea está clara, no hay que preguntar" | Clara para ti no es definida por el usuario. Las ideas "claras" que truenan al final son exactamente las que nadie preguntó. |
| "Es un cambio chico, aterrizar es demasiado proceso" | Para eso existe la escala exprés: 3 preguntas y un resumen de 5 líneas. Dos minutos contra una tarde de retrabajo. |
| "El usuario tiene prisa, mejor avanzo y ajusto después" | Ajustar después es reescribir. La prisa es la razón para aterrizar, no para saltarlo. |
| "Ya vamos a la mitad del proyecto, esto rompe el flujo" | Construir sobre una suposición equivocada rompe más que dos preguntas. El skill existe justo para las ideas que aparecen a la mitad. |
| "Si pregunto tanto voy a parecer inútil" | Preguntar lo que cambia el resultado es señal de senioridad. Inútil es entregar lo que nadie pidió. |
| "Le pongo lo que yo haría y que me corrija" | Eso convierte al usuario en QA de tus suposiciones. Las opciones se presentan ANTES de construir, no después. |
