# Lenguaje — frases prohibidas y vocabulario honesto

## Los tres estados legítimos de un trabajo

Todo reporte declara el trabajo en exactamente UNO de estos estados. No existen intermedios difusos.

| Estado | Cuándo lo usas | Formulación exacta |
|---|---|---|
| **Verificado** | Ejecutaste la evidencia del tipo de tarea (ver `evidencia.md`) y la pegas | "Verificado: [afirmación]. Evidencia: [comando + salida]" |
| **Parcialmente verificado** | Verificaste unas afirmaciones y otras no | "Verificado: A, B (evidencia adjunta). NO verificado: C, porque [razón]; para verificarlo: [comando]" |
| **Aplicado, sin verificar** | Hiciste el cambio pero no ejecutaste nada | "Cambio aplicado. NO lo he verificado. Para verificar: [comando exacto] — salida esperada: [qué]" |

Declarar los dos últimos no es debilidad: es la diferencia entre un reporte de ingeniería y una promesa. El usuario puede trabajar con "aplicado, sin verificar"; no puede trabajar con un "listo" falso.

## Frases prohibidas sin evidencia pegada en el MISMO mensaje

| Frase | Solo puedes escribirla si… |
|---|---|
| "Listo" / "Ya quedó" / "Terminado" / "✅" | …cada afirmación del trabajo tiene su evidencia pegada (escalera nivel 3-4). |
| "Funciona" / "Ya funciona correctamente" | …ejecutaste el flujo real y pegas la salida. Los tests unitarios no autorizan esta frase. |
| "Debería funcionar" | Nunca. O verificaste (y dices "funciona: [evidencia]") o no (y dices "aplicado, sin verificar"). |
| "Implementado exitosamente" | "Exitosamente" es una afirmación de resultado → nivel 3 mínimo. Sin evidencia, di solo "implementado, sin verificar". |
| "El deploy quedó" / "Ya está en producción" | …consultaste el entorno vivo (versión/commit) y pegas la respuesta. |
| "La migración corrió" | …consultaste el schema/conteos reales después y los pegas. |
| "Todo pasa" / "Los tests pasan" | …los corriste en ESTA sesión, después del último cambio, y pegas el resumen numérico. |
| "No debería romper nada más" | …corriste la suite completa del área y pegas el resumen. Si no, di "impacto no evaluado". |
| "Probé y funciona" (en pasado, sin salida) | Nunca sin la salida. La evidencia que no se pega no existe para el lector. |

## Reglas de redacción

1. **La evidencia va en el mismo mensaje que la afirmación.** "Lo probé hace rato" o evidencia en un mensaje anterior no cuenta si hubo ediciones después.
2. **Extrapolación se declara como extrapolación.** "Probé el caso A; B y C usan el mismo código y NO los ejecuté" — nunca "funciona para todos los casos" habiendo probado uno.
3. **Los emojis de éxito (✅ 🎉 🚀) siguen las mismas reglas que las frases.** Un ✅ junto a un ítem afirma que ese ítem está verificado. Ítem sin evidencia → sin emoji.
4. **El nivel de evidencia se nombra, no se infla.** "Compila" ≠ "los tests pasan" ≠ "funciona" ≠ "el efecto quedó en la DB". Usa la palabra del nivel que realmente alcanzaste (ver escalera en `evidencia.md`).
