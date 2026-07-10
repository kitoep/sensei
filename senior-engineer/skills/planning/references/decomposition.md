# Descomposición: de diseño a tareas ejecutables

## Regla 1 — Toda tarea tiene entregable verificable

Una tarea está bien definida solo si puedes contestar: **"¿qué comando corro o qué observo para saber que quedó?"**. Si no hay respuesta, no es una tarea, es una intención.

| ❌ Mal (intención) | ✅ Bien (entregable verificable) |
|---|---|
| "Avanzar en el módulo de reportes" | "Endpoint `GET /reports/monthly` devuelve el agregado correcto para el mes con datos seed; test de integración verde" |
| "Mejorar la validación" | "Requests con `email` inválido reciben 400 con mensaje de campo; test lo prueba con 3 payloads inválidos" |
| "Configurar la base de datos" | "Migración aplicada; `npm run db:migrate` corre limpio en local y CI; tabla visible con constraint único en (a, b)" |
| "Integrar el servicio X" | "Llamada real a X en sandbox retorna respuesta parseada; el timeout y el reintento se prueban simulando caída" |

Cada tarea del plan lleva su verificación ESCRITA en el plan mismo (comando, test, u observación concreta). La verificación se define al planear, no al ejecutar — definirla después invita a ajustarla a lo que salió.

## Regla 2 — Orden por riesgo, luego por dependencias

1. **Primero lo que puede invalidar el plan:** la suposición estructural detectada en la capa D de preguntas se verifica en la tarea 1 o con un spike previo al plan (spike = código desechable con pregunta concreta y timebox; su output es una respuesta, no código).
2. **Después lo de mayor incertidumbre técnica:** lo que no sabes hacer va antes que lo que sí, mientras el costo de pivotear es bajo.
3. **Al final lo mecánico:** CRUD, formularios, estilos, wiring — lo que seguro sale, cuando ya nada puede obligar a rehacerlo.
4. **Las dependencias son restricción, no orden:** dentro de lo que las dependencias permitan, gana el riesgo. "Primero el schema porque todo depende de él" es válido; "primero el CRUD porque es fácil" no.

## Regla 3 — Tamaño máximo de tarea

Una tarea debe caber en una sesión de trabajo con contexto completo: **si no puedes describir su diff esperado en 2-3 oraciones, pártela**. Señales de que una tarea está gorda:

- Su descripción tiene "y" estructural: "crear el endpoint Y el job Y la pantalla" → 3 tareas.
- Toca más de ~2 capas del sistema a la vez (schema + API + worker + UI) → partir por capa con la integración como tarea propia.
- Su verificación necesita párrafos → cada oración de verificación suele ser una tarea.

La primera tarea de una feature con UI + backend casi siempre es el **corte vertical mínimo**: el camino más angosto que atraviesa todas las capas de punta a punta (un campo, un botón, un dato real). Valida la integración antes de engordar cada capa.

## Regla 4 — Cuándo partir en sub-specs

Si el trabajo descrito contiene **subsistemas independientes** (p. ej. "chat + facturación + analytics"), no lo fuerces en un solo plan:

1. Identifica las piezas independientes y sus interfaces entre sí.
2. Define el orden entre piezas (por dependencia y por valor).
3. Cada pieza recibe su propio ciclo pregunta → spec → plan → ejecución.

Señal dura: si el plan pasa de ~10-12 tareas o mezcla dos definiciones de éxito distintas, son dos specs.

## Regla 5 — Toda tarea mapa a su evidencia de done

El plan cierra con una tabla tarea → evidencia. Al ejecutar, una tarea sin su evidencia ejecutada y documentada NO se marca hecha — "compiló" no es evidencia, "el test de la tarea corrió y pasó" sí. Si la evidencia falla, se reporta el fallo tal cual; nunca se maquilla.
