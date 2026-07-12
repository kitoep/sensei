---
name: crear-skills
description: Use when creating a new skill, editing an existing one, or reviewing a skill before publishing; when a repeated model failure suggests a new skill should exist; and before adding any skill to the suite without a RED/GREEN test.
---

# Crear skills: el método con que se construyó esta suite

## Principio

Un skill no es un documento de buenas intenciones: es un correctivo contra fallas concretas que el modelo comete sin él. Si no puedes nombrar la falla que corrige, con ejemplos de lo que el modelo hace mal hoy, el skill no se escribe. Y un skill sin prueba RED/GREEN es una hipótesis, no un skill: puede leerse precioso y no cambiar ninguna conducta.

**Violar la letra de estas reglas es violar su espíritu.** No hay skills "demasiado chicos" para el método: los chicos se diseñan, se validan y se prueban igual.

## REGLA QUE NO SE SALTA

**Nada se publica sin pasar las tres puertas:** falla baseline identificada por escrito, validación personal leyendo todo el contenido (no el reporte de quien lo escribió), y prueba RED/GREEN con el resultado verificado en disco. Si una puerta no pasó, el skill se queda en borrador y se dice así.

## Protocolo (carga el reference ANTES de actuar en su fase)

| Fase | Qué haces | PUERTA para avanzar | Reference |
|---|---|---|---|
| 1. Diseño | Defines las fallas baseline (qué hace mal el modelo sin el skill, con ejemplos literales) y el alcance | Lista escrita de fallas concretas; si no hay falla, no hay skill | `references/diseno-skill.md` |
| 2. Redacción | Escribes SKILL.md + references con las convenciones de la suite; si delegas, das rol de experto + propósito + fallas baseline | Estructura completa, description por condición, cero calcos y cero guiones medios | `references/redaccion.md` |
| 3. Validación personal | Lees TODO lo escrito, archivo por archivo, y corriges lo que no esté a tu nivel | Leíste cada archivo completo; lo que reportó el redactor no cuenta como lectura | `references/redaccion.md` |
| 4. Prueba RED/GREEN | Mismo pedido neutro sin el skill (RED) y con él (GREEN), con trampas y verificación en disco | Conducta comparada y evidencia real (git log, archivos, tests corridos), no dichos | `references/prueba-red-green.md` |
| 5. Publicar | Entrada en el marketplace, fila en el README, versión del plugin arriba, validación del paquete | `claude plugin validate .` pasa y el README refleja el skill nuevo | `references/prueba-red-green.md` |

## Red flags: DETENTE si te descubres escribiendo esto

- "Este skill se explica solo, no necesita prueba" → sin RED/GREEN no sabes si cambia la conducta o solo decora el contexto.
- "El subagente reporta que quedó bien" → el reporte no es el contenido. La validación es leerlo tú, completo.
- Una description que resume el workflow del skill → la description dispara, no enseña. Condiciones de activación, nada más.
- Triggers atados a frases exactas del usuario → las frases son ejemplos; el disparador es la condición de fondo.
- "Le agrego también esta otra doctrina ya que ando aquí" → un skill, una falla. Lo que no ataca la falla baseline, estorba.
- El RED se portó bien y lo reportas como éxito del skill → un RED limpio debilita la evidencia; se dice honesto y se endurece la trampa.

## Racionalizaciones conocidas

| Excusa | Realidad |
|---|---|
| "Es obvio que el skill ayuda, la prueba es burocracia" | Los skills que "obviamente ayudan" y no cambian nada existen. La prueba cuesta dos corridas; publicar placebo cuesta credibilidad. |
| "No tengo ejemplos de la falla, pero sé que pasa" | Si no puedes citar qué escribe mal el modelo, no puedes escribir la tabla de frases prohibidas que lo atrape. Junta ejemplos primero. |
| "Ya leí el resumen del redactor, con eso basta" | El resumen dice lo que el redactor cree que hizo. Los errores viven en lo que no notó. Se lee el archivo, no el resumen. |
| "La probé mentalmente y funciona" | Una corrida mental no tiene trampas ni disco que verificar. RED/GREEN real o queda en borrador. |
| "El skill quedó largo pero todo es importante" | Un SKILL.md gordo no se carga completo en la práctica. El detalle va en references; el SKILL.md orquesta. |
