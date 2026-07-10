# Veredicto — el reporte final de pruebas

El veredicto es lo único que el usuario lee. Sin este formato, el trabajo de pruebas no existe.

## Regla de cierre (sin excepciones)

**Prohibido declarar la feature "probada" si algún caso de prioridad 1 o 2 falló o no se ejecutó.** En ese estado el veredicto es **NO PROBADA** con la lista de lo pendiente/fallido — aunque los demás 20 casos hayan pasado, aunque haya prisa, aunque "seguro pasa". Un caso P1–P2 no ejecutable por entorno se reporta como bloqueante, no se omite.

## Plantilla (formato exacto)

```markdown
## Veredicto de pruebas — <feature>

**Resultado global:** PROBADA / NO PROBADA / PROBADA CON RIESGOS (solo P3–P5 pendientes)
**Resumen:** X pasaron / Y fallaron / Z no ejecutables

| # | P | Caso | Resultado | Evidencia |
|---|---|------|-----------|-----------|
| 1 | 1 | <caso> | PASÓ | `npm test -- auth.spec` → "12 passed" + SELECT muestra 0 filas cruzadas |
| 2 | 3 | <caso> | FALLÓ | ver Fallos |
| 3 | 5 | <caso> | NO EJECUTABLE | falta STRIPE_KEY en entorno de test |

### Fallos (tal cual, con reproducción)
- **Caso #2:** <comando exacto> → <salida de error completa, sin resumir ni suavizar>
  → estado que quedó: <p.ej. 2 filas en bookings para el mismo slot>
  → reproducción: <pasos mínimos>

### Cobertura honesta — qué NO se probó y qué riesgo deja abierto
- <área/caso no cubierto> → riesgo: <qué podría fallar en producción sin que lo sepamos>
- <o: "el plan se ejecutó completo; sin huecos conocidos">
```

## Reglas de llenado

1. **Evidencia = comando + fragmento de salida relevante.** "PASÓ ✅" a secas no es un resultado — es una afirmación sin soporte y invalida el veredicto.
2. **Los fallos se reportan tal cual:** salida completa del error, sin "casi pasa", "es un detalle menor", "fallaría solo en un caso raro". Clasificar la severidad es del lector; tu trabajo es la evidencia.
3. **"No ejecutable" lleva razón concreta** (env faltante, servicio caído, selector inexistente) y qué se necesita para destrabarlo.
4. **La sección de cobertura honesta es obligatoria** aunque quede en "sin huecos conocidos" — escribirla obliga a pensarla.
5. Si durante las pruebas se detectó un bug FUERA del alcance del plan, se lista en cobertura/fallos con nota "fuera de alcance, no corregido" — nunca se arregla en silencio ni se omite.

## Racionalizaciones prohibidas al cerrar

| Excusa | Realidad |
|---|---|
| "Fallan 2 pero son edge cases" | Si son P1–P2, la feature está NO PROBADA. Si son P4–P5, el veredicto es PROBADA CON RIESGOS y los lista. |
| "No pude correr el de concurrencia, pero la lógica se ve bien" | Concurrencia es P3 y su garantía es un constraint: sin ejecutarlo no hay veredicto positivo sobre esa invariante. |
| "El test falla por el entorno, en CI pasará" | Entonces el resultado es NO EJECUTABLE + bloqueante, no PASÓ anticipado. |
| "Ya reporté los fallos en el chat, no hace falta la plantilla" | La plantilla ES el entregable auditable. Sin ella el cierre no ocurrió. |
