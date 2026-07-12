# Cierre: el aterrizaje final

El aterrizaje termina en un texto aprobado por el usuario. No en "creo que ya quedó claro", no en un acuerdo implícito de la conversación: un texto concreto que el usuario leyó y aprobó. Ese texto es el contrato de lo que sigue.

## Escala exprés: el resumen de ~5 líneas

Para ideas chicas a mitad de proyecto. Se escribe en el chat y se pide aprobación explícita:

```markdown
**Aterrizado así:**
- Qué es: <una línea concreta>
- Para quién / qué problema: <una línea>
- Incluye: <lo mínimo útil, concreto>
- NO incluye: <anti-alcance explícito, al menos un punto>
- Se valida con: <conducta observable o criterio de éxito>

¿Lo apruebas así o ajusto algo?
```

Sin aprobación explícita del usuario no se avanza. "Suena bien" del propio modelo no cuenta.

## Escala completa: diseño por secciones

Para features grandes o proyectos. El diseño se presenta POR SECCIONES, no en un bloque de 800 palabras:

- Secciones típicas: contexto y problema, alcance (incluye / no incluye), enfoque elegido y por qué, componentes y responsabilidades, flujo de datos, manejo de errores, cómo se valida.
- Cada sección escala a su complejidad: dos líneas si es simple, un párrafo si tiene matices.
- **Aprobación incremental**: después de cada sección (o cada 2-3 si son cortas) preguntas si va bien ANTES de seguir. Corregir la sección 2 a tiempo evita reescribir de la 3 a la 7.
- Si el proyecto lo amerita, el diseño aprobado se guarda como archivo (por ejemplo `docs/design/<tema>.md`) para que sobreviva a la conversación.

## Auto-revisión ANTES de pedir la aprobación

Lee tu propio resumen o diseño con ojos de auditor y corrige en el momento:

| Chequeo | Pregunta | Si falla |
|---|---|---|
| Huecos | ¿Queda algún "por definir", "TBD", "luego vemos"? | Esa línea es tu siguiente pregunta al usuario; no se entrega con huecos. |
| Contradicciones | ¿Alguna sección choca con otra? ¿El alcance contradice el mínimo útil? | Resuélvelo o pregúntalo antes de presentar. |
| Ambigüedad | ¿Algo se puede leer de dos formas? | Elige una lectura y hazla explícita en el texto. |
| Tamaño | ¿Esto cabe en UN plan de implementación? | Si no, propón la partición en piezas y aterriza la primera. |

## El destino: a dónde va lo aterrizado

- Con la aprobación en mano, el siguiente paso es PLANEAR, no implementar. Si el skill `planeacion` está instalado, pásale el resultado: el aterrizaje es exactamente el insumo que su intake espera. Si no está instalado, propón igual un plan corto antes de tocar código.
- **NUNCA saltes directo a implementar desde el aterrizaje.** El texto aprobado dice QUÉ se construye; el plan dice CÓMO y en qué orden. Son dos pasos distintos y el segundo también se aprueba.
- Si durante el plan o la implementación la idea vuelve a ponerse difusa (aparece un caso no contemplado, el usuario cambia una pieza), se regresa aquí: se aterriza el cambio con la escala exprés y se sigue. El aterrizaje no es una fase del inicio, es la respuesta estándar a lo difuso en cualquier momento.
