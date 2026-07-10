# Intake — el interrogatorio antes de recomendar

Objetivo: convertir "tengo una idea" en condiciones medibles. Haz las preguntas de una en una o en bloques de 2-3 (no las 9 de golpe). Registra cada respuesta; la recomendación final las citará.

**Regla de oro:** si el usuario no sabe una respuesta, asume el escenario CHICO y déjalo explícito en la recomendación ("asumí <1,000 usuarios/mes porque no hay dato"). Nunca asumas el escenario grande: sobredimensionar cuesta dinero real hoy; subdimensionar cuesta una migración mañana — y la migración desde algo simple es más barata que pagar infra fantasma 12 meses.

## Las 9 preguntas (cada una con su porqué)

1. **¿Cuántos usuarios/tráfico esperas REALMENTE en los próximos 6 meses?** (no el sueño a 5 años)
   *Porqué:* dimensiona todo. La diferencia entre 100 y 100,000 usuarios/mes cambia la arquitectura; la diferencia entre 100,000 y "algún día millones" no cambia nada hoy. Si la respuesta es una fantasía ("miles de negocios"), repregunta: *"¿cuántos clientes tienes HOY comprometidos o en pipeline?"*.

2. **¿Cuál es tu presupuesto mensual de infraestructura?** (número, no "poco")
   *Porqué:* es la restricción más honesta. $10/mes y $200/mes producen recomendaciones distintas y ambas válidas. Pide un número o un rango; si no lo hay, asume el mínimo.

3. **¿Quién opera esto?** (¿solo tú? ¿hay alguien de devops/on-call?)
   *Porqué:* determina managed vs self-hosted. Un servidor propio "barato" cuesta horas de parches, backups y despertarse a las 3 AM. Si opera una sola persona que además desarrolla: managed casi obligatorio.

4. **¿Qué pasa si el sistema se cae 1 hora un martes a las 11 AM?** (¿pierdes dinero? ¿clientes? ¿nada?)
   *Porqué:* define el presupuesto de disponibilidad. "Sería vergonzoso" → single-instance con buen restart automático basta. "Pierdo transacciones/citas de clientes" → redundancia y health checks. "Alguien puede salir lastimado" → otra liga de diseño. La mayoría sobreestima esto: repregunta con el escenario concreto.

5. **Datos: ¿cuánto volumen, qué tan sensibles, hay compliance?** (PII, salud, pagos, menores)
   *Porqué:* datos sensibles imponen requisitos no negociables (cifrado, residencia de datos, RLS multi-tenant, auditoría) que restringen proveedores y suben el piso de costo. Detectarlo tarde = re-arquitectura.

6. **¿Necesitas tiempo real?** (chat, notificaciones push, dashboards en vivo — ¿o "cada 30 segundos" basta?)
   *Porqué:* el tiempo real genuino (websockets, colas) agrega piezas con costo operativo. El 80% de los "necesito tiempo real" se resuelve con polling corto, que no agrega infra. Distingue cuál es.

7. **¿Qué integraciones son obligadas?** (WhatsApp, pasarela de pago, ERP, APIs de terceros)
   *Porqué:* las integraciones imponen restricciones técnicas (webhooks públicos → necesitas URL estable; jobs de reintento → necesitas worker/cola) y a veces dictan región o proveedor.

8. **¿Qué tecnologías domina ya el equipo?** (lenguajes, frameworks, DB con las que ya trabajaron en serio)
   *Porqué:* el stack conocido entrega en semanas; el óptimo teórico entrega en meses y con bugs de novato. Solo se justifica tecnología nueva si una condición la exige — y entonces se declara como costo (curva de aprendizaje) en la recomendación.

9. **¿Qué es lo MÁS incierto del negocio hoy?** (¿el modelo de precios? ¿el canal? ¿la feature central?)
   *Porqué:* lo incierto define qué debe ser barato de cambiar. Si el modelo de negocio puede pivotar, el dominio debe estar desacoplado de la infra; si el canal es incierto (¿web? ¿WhatsApp? ¿app?), la lógica no puede vivir en el frontend.

## Cierre del intake

Antes de pasar a recomendar, resume en 3-5 líneas lo entendido y pide corrección:

```
Entendí: [N usuarios a 6 meses], presupuesto [$X/mes], opera [quién],
caída de 1h = [consecuencia], datos [sensibilidad/compliance],
equipo domina [stack]. Lo más incierto: [X]. ¿Corrijo algo?
```

Si el usuario corrige, actualiza y vuelve a resumir. Solo con el resumen confirmado (o con supuestos explícitos donde no hubo respuesta) pasa a priorizar la fuerza dominante.
