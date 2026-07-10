# Formato de salida — plantilla de la recomendación

La recomendación final se entrega EXACTAMENTE con estas 7 secciones, en este orden. Ninguna es opcional. Si una sección quedaría vacía, es señal de que el intake está incompleto — regresa a preguntar.

**Regla de trazabilidad (se aplica antes de entregar):** recorre cada elección de la sección 3; si no puedes citar la condición del intake que la justifica, es moda — quítala o muévela a la sección 5 (todavía no).

---

```markdown
## 1. Contexto entendido
[3-6 líneas: usuarios reales a 6 meses, presupuesto, quién opera, consecuencia
de 1h de caída, sensibilidad de datos, stack que domina el equipo, lo más
incierto del negocio. Marca con "(supuesto)" todo dato que asumiste por falta
de respuesta — el usuario corrige aquí, no después de construir.]

## 2. Fuerza dominante
[UNA: costo | disponibilidad | velocidad de desarrollo | escala.
1-2 líneas de por qué esa y qué se subordina a ella.]

## 3. Recomendación por capas
| Capa | Elección | Por qué (condición del usuario) |
|---|---|---|
| Hosting | ... | ... |
| Base de datos | ... | ... |
| Backend | ... | ... |
| Frontend | ... | ... |
| Jobs/colas | ... | (o "no aplica: <condición>") |
| Storage | ... | ... |
| Observabilidad | ... | ... |
[La columna 3 cita la condición, no genéricos: "porque operas solo y sin
devops" ✔ — "porque es escalable" ✘]

## 4. Costo estimado mensual
[Desglose por pieza + total en rango: "$X–Y/mes". Si excede el presupuesto
del intake, decirlo y ofrecer qué recortar.]

## 5. Todavía NO — y la señal para agregarlo
[Tabla: pieza descartada hoy → métrica/umbral concreto que dispara agregarla
→ qué seam ya quedó preparado para ese día.
Ej.: "Cache Redis → misma query domina el p95 >500ms → repos ya encapsulan
el acceso a datos".]

## 6. Riesgos de esta recomendación
[2-4 riesgos REALES de lo recomendado, con su mitigación o su aceptación
explícita. Ej.: "single-instance: deploy con ~10s de blip — aceptado por
presupuesto". Una recomendación sin riesgos declarados está incompleta.]

## 7. Alternativa principal descartada
[La segunda opción seria que consideraste y POR QUÉ perdió contra las
condiciones (no contra tu gusto). 2-4 líneas. Da al usuario la puerta:
"si [condición] cambia a [X], esta alternativa gana".]
```

---

## Checks finales antes de enviar

1. ¿Cada fila de la sección 3 cita una condición del intake? (trazabilidad)
2. ¿El total de la sección 4 cabe en el presupuesto declarado?
3. ¿La sección 5 tiene al menos 2 piezas? (si todo entró hoy, sobredimensionaste)
4. ¿Los supuestos están marcados "(supuesto)" en la sección 1?
5. ¿La decisión irreversible más grande (DB / modelo multi-tenant / lenguaje) quedó explicada con más profundidad que las reversibles?
6. ¿Le dijiste al usuario algo que no quería oír? (si la recomendación coincide 100% con lo que él ya pensaba, revisa si estás complaciendo en vez de analizando)
