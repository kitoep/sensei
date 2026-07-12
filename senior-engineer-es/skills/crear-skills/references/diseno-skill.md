# Diseño del skill: de la falla al esqueleto

## Fallas baseline: la materia prima

Un skill existe para corregir conductas concretas que el modelo tiene SIN él. Antes de escribir una sola línea, documenta:

1. **La falla, en conducta observable.** No "el modelo planea mal" sino "entrega el plan completo sin preguntar nada y al final pregunta si la feature ya existía".
2. **Lo que el modelo escribe literalmente cuando falla.** Estas citas son oro: se convierten en la tabla de frases prohibidas y en las red flags. Ejemplos reales de esta suite: "Listo ✅" sin correr nada, "para tu escala probablemente es suficiente" sin un solo dato de la escala, "asumo que te refieres a...".
3. **El costo de la falla.** Si no cuesta nada, no amerita skill.

Junta mínimo 3 fallas con ejemplos. Si solo tienes una intuición ("siento que el modelo hace X"), reproduce la falla primero: dale a un modelo sin skill una tarea que la provoque y guarda lo que hizo. Eso es tu RED anticipado y tu evidencia de diseño.

## Un skill, una falla (familia de fallas)

La prueba de alcance: todas las fallas baseline deben caber en una sola oración de propósito. Si necesitas "y también...", son dos skills. Señales de que estás mezclando:

- La tabla de protocolo tiene fases que no comparten ninguna puerta.
- La description necesita dos listas de condiciones sin relación.
- Un usuario querría instalar la mitad del skill.

## Anatomía del SKILL.md (~50 a 60 líneas)

| Sección | Qué lleva | Error común |
|---|---|---|
| Frontmatter | `name` en kebab-case (idioma nativo de la suite), `description` en inglés | Description que resume el workflow en vez de disparar |
| Principio | Por qué existe, en 3 a 5 líneas, con el costo de la falla | Filosofía genérica que aplicaría a cualquier skill |
| Regla dura | La prohibición central que no se negocia | Regla suave ("procura...", "idealmente...") |
| Protocolo | Tabla fase → qué haces → PUERTA para avanzar → reference | Fases sin puerta verificable |
| Red flags | Lo que el modelo escribe LITERALMENTE cuando está fallando | Red flags abstractas que no atrapan texto real |
| Racionalizaciones | Tabla excusa → realidad, cerrando cada salida conocida | Excusas inventadas que ningún modelo diría |

El detalle vive en `references/` (2 a 4 archivos), cargados por fase. El SKILL.md orquesta y frena; los references enseñan.

## La description: dispara por condición, no por frase

La description es lo único que el modelo ve al decidir si activa el skill. Reglas:

1. **Empieza con la condición de fondo** ("Use when anything about to be built is not 100% defined..."), no con frases del usuario.
2. **Las frases van al final como ejemplos** ("e.g. ..."), porque ayudan al matching pero no lo limitan.
3. **Incluye la auto-detección si aplica:** los mejores disparadores son conductas del propio modelo ("when about to claim work is done", "when catching yourself writing 'I'll assume...'").
4. **Solo condiciones de activación.** Nada de resumir qué hace el skill: eso invita al modelo a creer que ya lo aplicó por haber leído la description.

## Red flags y racionalizaciones eficaces

- Las red flags citan texto que el modelo de verdad produce, en el idioma en que va a producirlo. Por eso las dos versiones de la suite no se traducen literal: se reescriben con lo que cada idioma genera ("Ya quedó" / "Done, all set").
- Cada racionalización lleva su realidad al lado, con el costo concreto ("reproducir toma minutos; el falso fix cuesta horas"). Una excusa sin refutación es una sugerencia.
- La fuente de ambas tablas son las fallas baseline del paso 1. Si te las estás inventando, te faltó evidencia de diseño.
