# Plantilla del plan escrito

Usa esta estructura exacta. Un plan que omite una sección debe decir por qué la omite (una línea), no saltársela en silencio. Guárdalo como archivo en el repo (`docs/plans/YYYY-MM-DD-<tema>.md` o donde el proyecto ya guarde planes) — un plan que solo vive en el chat se pierde.

```markdown
# Plan: <nombre corto>

**Fecha:** YYYY-MM-DD · **Estado:** borrador | aprobado | en ejecución | completado

## 1. Contexto y objetivo
<2-5 oraciones: qué problema se resuelve, para quién, y cómo se ve el éxito
en términos verificables. Sin jerga inventada durante la conversación.>

## 2. Decisiones tomadas
<Tabla. Solo decisiones REALES (hubo alternativas), con su porqué.
Marca las irreversibles.>

| Decisión | Alternativas descartadas | Por qué | ¿Irreversible? |
|---|---|---|---|

## 3. Fuera de alcance
<Lista explícita de lo que NO incluye esta versión. Si algo se pospuso
de una lista/tracker previo, nómbralo y di a dónde se pospuso.>

## 4. Suposiciones y riesgos
<La suposición estructural y cómo/cuándo se verifica. Riesgos con su
mitigación u "aceptado". Datos legacy y fronteras externas relevantes.>

## 5. Tareas
<Numeradas, en orden de ejecución (riesgo primero, dependencias respetadas).>

### Tarea N: <título con entregable>
- **Qué:** <2-3 oraciones; el diff esperado descrito>
- **Archivos probables:** <rutas>
- **Verificación:** <comando/test/observación concreta que prueba que quedó>
- **Depende de:** <tareas previas o "—">
- **Riesgo:** <qué puede salir mal aquí, o "bajo">

## 6. Criterios de done del conjunto
<Checklist final: qué debe ser cierto para declarar TODO terminado.
Incluye: todos los tests del plan verdes, typecheck/build/CI verde,
docs actualizados si el QUÉ cambió, y la demo/flujo end-to-end que
el usuario puede ejecutar para verlo funcionando.>
```

## Checklist de auto-revisión (antes de presentar el plan)

Recorre TODOS los puntos; corrige inline y solo entonces presenta:

1. **Placeholders:** ¿queda algún TBD, "por definir", sección vacía o "etc." que esconde trabajo? → resuélvelo o conviértelo en pregunta explícita al usuario.
2. **Contradicciones:** ¿alguna tarea contradice una decisión de la sección 2? ¿el fuera-de-alcance contradice el objetivo?
3. **Verificabilidad:** ¿CADA tarea tiene verificación concreta? Prueba dura: ¿podría otro agente, sin esta conversación, ejecutar la tarea y saber si quedó? Si necesita contexto del chat, el contexto falta en el plan.
4. **Tamaño:** ¿alguna tarea con "y" estructural o diff no descriptible en 2-3 oraciones? → pártela.
5. **Orden:** ¿la mayor incertidumbre está al inicio? ¿la suposición estructural se verifica en la tarea 1 o antes?
6. **Reconciliación:** si existía tracker/spec/backlog previo, ¿cada ítem de ahí está en tareas, en fuera-de-alcance, o marcado para re-planificar? Ninguno puede simplemente no aparecer.
7. **Ambigüedad:** ¿alguna frase interpretable de dos formas? Elige una y escríbela.
8. **YAGNI:** ¿hay tareas que nadie pidió y ningún criterio de éxito necesita? → fuera (a la sección 3 si vale la pena registrarlas).

## Presentación al usuario

Presenta el plan por secciones (no un muro de texto), pide aprobación, e incorpora cambios donde corresponda (una decisión nueva puede reordenar tareas). **El plan no se ejecuta sin aprobación explícita.** Tras aprobarse, los cambios de alcance durante la ejecución se negocian contra el plan — "ya que estamos, agreguemos X" reabre la sección 3, no se cuela como tarea 7b.
