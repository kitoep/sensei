# Cómo derivar el plan de pruebas de una feature

## Paso 1 — Mapear invariantes (antes de escribir un solo caso)

Responde por escrito estas preguntas sobre la feature. Cada respuesta genera casos:

1. **¿Qué debe ser IMPOSIBLE que pase?** (doble reserva, ver datos de otro tenant/usuario, cobrar dos veces, enviar el mensaje dos veces, saltarse un permiso). → casos prioridad 1–2.
2. **¿Quién NO debe poder hacer esto?** (rol sin permiso, usuario anónimo, usuario de otra cuenta, token expirado). → casos adversariales.
3. **¿Qué pasa si llegan N a la vez?** (mismo slot, mismo submit, mismo webhook con retry). → casos de concurrencia/idempotencia.
4. **¿Qué garantiza la invariante: lógica de aplicación o constraint/mecanismo de plataforma?** Si es lógica de app, el caso debe intentar esquivarla; si es constraint (unique, RLS, FK), el caso debe ejecutarlo de verdad (por eso: DB real).
5. **¿Qué estado queda en el sistema tras la operación?** El caso verifica ese estado (fila creada/ausente, job encolado, mensaje NO enviado), no solo la respuesta HTTP.

Si no puedes enunciar ninguna invariante, no entiendes la feature todavía: lee el spec/código antes de planear.

## Paso 2 — Ordenar por prioridad fija

| P | Categoría | Qué prueba |
|---|---|---|
| 1 | Adversarial / seguridad | Lo prohibido FALLA: sin permiso → rechazado; otro tenant → 404 (no 403, para no filtrar existencia); input malicioso → rechazado por el servidor, no por la UI. |
| 2 | Invariantes de negocio | Unicidad, aislamiento de datos, no doble reserva, estados válidos. Verificando el EFECTO en DB, no el status code. |
| 3 | Concurrencia e idempotencia | N requests simultáneos al mismo recurso → exactamente 1 gana. Retry del mismo job/webhook → no duplica efectos. |
| 4 | Happy path | El flujo normal completo funciona de punta a punta. |
| 5 | Bordes | Vacíos, nulls, límites de longitud, unicode/emoji, zonas horarias y DST, paginación en el límite, datos legacy en estados viejos. |

No toda feature tiene casos en todas las categorías — pero la AUSENCIA de una categoría se declara explícitamente en el plan ("no hay superficie de concurrencia porque X"), nunca se omite en silencio.

## Paso 3 — Plantilla del plan (formato exacto)

```markdown
## Plan de pruebas — <feature>

**Invariantes identificadas:**
1. <invariante y qué la garantiza (constraint / guard / lógica)>
2. ...

| # | P | Caso | Tipo | Cómo se ejecuta | Evidencia esperada |
|---|---|------|------|-----------------|--------------------|
| 1 | 1 | Usuario sin permiso X no puede <acción> | integración | <comando de test / request concreto> | 403 por permiso (verificar body.message) Y efecto ausente en DB |
| 2 | 3 | 10 requests concurrentes al mismo <recurso> | integración | <script/test con Promise.all> | exactamente 1 con 201; 9 con 409; 1 sola fila en DB |
| ... |

**Categorías sin casos y por qué:** <o "ninguna">
**Prueba de fuego:** por cada caso P1–P2, indicar qué enforcement se está ejerciendo
(si se borrara <guard/constraint>, el caso #N fallaría porque <razón>).
```

Reglas de llenado:

- **Tipo** ∈ unit / integración / E2E. Lo que ejerce constraint, RLS, permisos o colas es integración o E2E contra dependencias reales — nunca unit con mocks.
- **Cómo se ejecuta** es un comando o pasos reproducibles, no "verificar que funcione".
- **Evidencia esperada** incluye siempre el efecto real (estado en DB, mensaje enviado/no enviado, archivo creado), no solo el código de respuesta.
- Un caso P1/P2 cuyo test seguiría verde sin el enforcement es un caso mal diseñado: rediseñar antes de ejecutar.

## Paso 4 — Decidir ejecución

- Plan ≤ ~8 casos y un solo área: ejecútalo tú directamente.
- Plan con áreas independientes (API / UI / concurrencia) o > ~8 casos: despliega subagentes con briefs de `subagent-briefs.md`, uno por área, nunca dos tocando el mismo estado.
- Si la feature tiene UI: los casos de UI salen de `ui-testing.md` y son parte de ESTE plan, no un anexo opcional.

## Errores comunes al planear

- Derivar casos de la lista de endpoints/pantallas → produce solo happy paths. Derivar de invariantes.
- Probar que aparece un flag/señal en la respuesta en vez de intentar la acción prohibida y exigir que falle.
- Escribir el caso de concurrencia como secuencial (uno tras otro) — así nunca ejercita la carrera; usar requests realmente simultáneos.
- Confiar en que "el framework valida" — el caso debe mandar el input inválido crudo (curl/cliente HTTP), saltándose la UI.
- Olvidar el caso de retry: todo job/webhook/handler con reintentos necesita su caso de idempotencia.
