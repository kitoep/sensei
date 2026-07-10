# Entrega de resultados — plantilla y reglas del mensaje final

## Regla central: 1 afirmación → 1 evidencia

Cada afirmación de comportamiento o efecto en tu mensaje final mapea a exactamente una pieza de evidencia ejecutada en el mismo mensaje. Afirmación sin evidencia → o consigues la evidencia, o mueves la afirmación a "No verificado". No hay tercera opción.

## Plantilla del mensaje final

```markdown
## Qué cambió
[1-3 líneas: archivos/sistemas tocados y el efecto esperado. Sin narrativa del proceso.]

## Verificado (evidencia ejecutada en esta sesión, después del último cambio)
- [Afirmación 1] — nivel [3: flujo ejecutado / 4: estado consultado]
  ```
  $ [comando exacto]
  [líneas relevantes de la salida real]
  ```
- [Afirmación 2] — ...

## No verificado
- [Afirmación X] — por qué no: [sin acceso a prod / falta credencial / entorno no disponible]
  Para verificar: `[comando exacto]` — salida esperada: [qué debe verse]

## Riesgos residuales (si los hay)
- [Qué podría fallar aun con lo verificado, p. ej. "solo probé con datasets < 1k filas"]
```

Las secciones "No verificado" y "Riesgos residuales" se omiten SOLO si están genuinamente vacías — no porque se vean mal. Una entrega con "No verificado" poblado es más confiable que una sin la sección.

## Reglas sobre la evidencia pegada

1. **Salida real, no reconstruida.** Pega lo que la terminal imprimió. Si escribes de memoria "respondió 200", eso es una afirmación, no evidencia.
2. **Truncar está bien; las líneas que prueban, no.** De una salida de 200 líneas pega las 5-10 que demuestran la afirmación (el resumen de tests, la fila de la DB, el status + body relevante) y marca el corte con `[...]`.
3. **Incluye el comando junto con la salida.** La salida sin comando no es auditable ni reproducible por el usuario.
4. **Timestamp lógico:** la evidencia es válida solo si se ejecutó DESPUÉS del último edit. Si editaste algo tras verificar, la verificación afectada regresa a "No verificado" hasta re-ejecutarla.
5. **Para UI:** la evidencia es el screenshot o la salida de Playwright, con el estado que muestra nombrado ("estado de error con mensaje visible").

## Anti-patrones de entrega

| Anti-patrón | Corrección |
|---|---|
| Lista de checkmarks ✅ por cada tarea, cero salidas | Cada ✅ se convierte en una entrada de "Verificado" con evidencia, o pierde el checkmark |
| "Resumen de todo lo que hice" de 30 líneas, evidencia en 0 | La entrega se organiza por afirmaciones verificables, no por cronología de edits |
| Evidencia de nivel 1-2 presentada bajo "funciona" | Renombra a lo que es: "compila", "los tests unitarios pasan". Ver escalera en `evidencia.md` |
| "Verificado manualmente" sin decir qué comando ni qué salida | "Manualmente" no es un método. Comando + salida, o va a "No verificado" |
| Prometer verificación futura ("luego lo pruebo") y cerrar como terminado | El estado es "aplicado, sin verificar" hasta que la verificación ocurra |

## Relación con `fixer`

Si el trabajo era un bugfix y el skill `fixer` está instalado, el estándar de evidencia es su checklist de 5 puntos (`fixer/references/verification.md`: reproducción original, test de regresión visto fallar, suite completa, efecto en estado, patrón simétrico) — es un superset de esta plantilla. Si `fixer` no está instalado, usa esta plantilla exigiendo como mínimo la reproducción original ejecutada y pasando, más la suite del área. Esta plantilla aplica a todo lo demás: features, refactors, config, deploys, migraciones, scripts, UI.
