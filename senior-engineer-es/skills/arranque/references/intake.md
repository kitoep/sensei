# Intake — las preguntas antes de la primera línea de código

Objetivo: obtener las respuestas que CAMBIAN la arquitectura. No es un formulario burocrático: cada pregunta desbloquea una decisión concreta. Si una respuesta no cambiaría nada de lo que vas a construir, no la preguntes.

## Reglas de aplicación

1. Haz las preguntas **de una en una** o en grupos de máximo 3 con opciones múltiples. El usuario abandona los cuestionarios largos.
2. Ofrece siempre tu recomendación con su porqué ("Recomiendo X porque..."), no un menú neutro.
3. Registra cada respuesta en `docs/00_intake.md` conforme llega. Al final, muestra el documento completo al usuario y pide confirmación explícita.
4. Si el usuario contesta "no sé", propón el default más barato de cambiar después y márcalo como `[SUPUESTO]` en el doc — los supuestos se revisan al primer milestone.
5. Si detectas contradicción entre respuestas (p. ej. "presupuesto $0" + "alta disponibilidad"), señálala en el momento, no la resuelvas en silencio.

## Cuestionario

### Bloque A — Propósito (define el scope)

**1. ¿Qué problema resuelve y para quién, en un párrafo?**
Por qué: si no cabe en un párrafo, el proyecto no está definido y todo lo demás es especulación.
Desbloquea: el criterio para decir NO a features (todo lo que no sirva a ese párrafo se corta).

**2. ¿Cómo se ve el éxito en 3 meses, en números?** (usuarios activos, negocios pagando, horas ahorradas — algo medible)
Por qué: "que funcione" no es criterio; sin número no hay forma de priorizar.
Desbloquea: qué milestone es el primero y qué es v2.

**3. ¿Qué NO se va a construir en v1?** (scope negativo explícito — pide mínimo 5 ítems)
Por qué: el scope negativo es la única defensa contra el crecimiento silencioso. Lo que no está en la lista de exclusión, alguien lo va a pedir.
Desbloquea: la sección "NO implementar" del CLAUDE.md.

### Bloque B — Restricciones (definen el stack)

**4. ¿Presupuesto mensual de infraestructura?** (rangos: $0 / <$25 / <$100 / <$500 / no importa)
Por qué: el presupuesto decide el hosting y la arquitectura más que cualquier preferencia técnica. Con <$25 no hay Kubernetes ni multi-región, y eso está bien.
Desbloquea: hosting, base de datos gestionada vs propia, y cuánta redundancia es honesta.

**5. ¿Quién va a mantener esto?** (solo el usuario + agentes / un equipo chico / un equipo que crecerá)
Por qué: el techo de complejidad lo pone el mantenedor, no la ambición. Un stack que el mantenedor no domina es deuda desde el día 1.
Desbloquea: cuántas piezas móviles se permiten (¿monolito modular o servicios?) y cuánta documentación es obligatoria.

**6. ¿Plazo? ¿Qué pasa si se entrega un mes tarde?** (nada / se pierde un cliente / se muere el proyecto)
Por qué: la respuesta calibra cuánto riesgo técnico puedes tomar. "Se muere el proyecto" = cero experimentos, todo stack aburrido y probado.
Desbloquea: la agresividad del plan de milestones.

**7. ¿Usuarios concurrentes realistas en el año 1?** (decenas / cientos / miles / no sé)
Por qué: la sobre-ingeniería para escala imaginaria mata más proyectos que la falta de escala. Cientos de usuarios concurrentes los aguanta un monolito con Postgres sin despeinarse.
Desbloquea: permiso explícito para NO construir para escala que no existe.

### Bloque C — Naturaleza del sistema (decisiones irreversibles)

**8. ¿Es multi-tenant?** — ¿más de un negocio/organización va a usar la misma instancia con datos que JAMÁS deben cruzarse?
Por qué: multi-tenancy NO se retrofitea. Agregarlo después implica tocar cada tabla, cada query y cada test. Es la decisión más irreversible de todas.
Desbloquea: si sí → `tenant_id` en todo el schema + aislamiento a nivel DB desde la primera tabla (ver foundations.md #10).

**9. ¿Maneja datos personales o sensibles? ¿De qué jurisdicción son los titulares?** (México/LFPDPPP, UE/GDPR, salud, pagos)
Por qué: compliance no es una feature que se agrega: define qué se cifra, qué se loggea, qué se puede borrar y dónde se hospeda.
Desbloquea: cifrado en reposo, política de logs sin PII, mecanismo de borrado/anonimización, y si aplica el skill `seguridad` desde el día 1.

**10. ¿De qué integraciones externas depende el corazón del producto?** (APIs de terceros, pasarelas, WhatsApp, aprobaciones de plataforma)
Por qué: la integración externa es el riesgo que tú no controlas — cuotas, aprobaciones, cambios de API. Si el producto depende de una, esa integración se valida ANTES de construir alrededor de ella.
Desbloquea: el primer spike del plan (ver walking-skeleton.md, orden por riesgo).

**11. ¿Hay usuarios con login? ¿Hay roles distintos?**
Por qué: auth y permisos son cimiento, no feature. Meter roles después de que "todos pueden todo" implica auditar cada endpoint existente.
Desbloquea: auth default-closed + modelo de roles desde el esqueleto (foundations.md #9).

### Bloque D — Riesgo

**12. ¿Qué es lo único que, si resulta falso, invalida el proyecto completo?**
Por qué: todo proyecto tiene una apuesta central (la API lo permite / la gente pagará / el modelo puede hacerlo). Nombrarla obliga a probarla primero, barato.
Desbloquea: el milestone 1 real — que casi nunca es el que el usuario tenía en mente.

## Salida de la fase

`docs/00_intake.md` con las 12 respuestas (y supuestos marcados), confirmado por el usuario. Solo entonces pasa a `references/foundations.md`.
