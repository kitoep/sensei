# Testing adversarial — anti "falsos verdes"

Reglas nacidas de bugs reales donde un test pasaba pero probaba lo equivocado. Un test de seguridad que no puede fallar cuando la protección desaparece es peor que no tener test: da confianza falsa.

## La prueba de fuego (aplícala a CADA test de seguridad)

> **"Si borro la lógica de enforcement del backend, ¿este test seguiría verde?"**

Si la respuesta es sí, el test es inútil. Bórralo o arréglalo. Un test de una invariante de seguridad DEBE ponerse rojo cuando la defensa se quita.

## Regla 1 — Adversarial, no confirmatorio

- **Confirmatorio (malo):** verifica que el flujo feliz funciona, o que aparece un flag/señal. Ejemplo inútil: "el response trae `canEdit: false`".
- **Adversarial (bueno):** intenta hacer la acción PROHIBIDA con un cliente que ignora la UI, y exige que FALLE y que el efecto real NO ocurra.
- Un flag para el frontend NO es enforcement. El test debe ignorar el flag e intentar la acción de todos modos.

## Regla 2 — Verifica el efecto real, no el status HTTP

- Tras una mutación denegada, no basta con recibir 403/404. **Consulta el estado real:** la fila NO se creó/modificó/borró en la DB; el mensaje NO se envió; el slot quedó libre.
- Un 200 puede esconder que no pasó nada, y un 403 puede llegar sin que se haya evaluado el permiso. Comprueba el efecto.

## Regla 3 — El 403/404/409 debe llegar por la razón correcta

Un rechazo puede venir por el motivo equivocado y dar un falso verde:
- **CSRF:** si pruebas permisos con una petición que además falla CSRF, el 403 es de CSRF, no de permisos. Usa Bearer token (que exceptúa CSRF) para aislar el control que quieres probar.
- **Throttler:** un 429 disfrazado, o un 403 tras rate limit.
- **Validación de formato:** un `ParseUUIDPipe`/schema que rechaza el ID malformado antes de llegar al check de permiso. Usa un ID válido y bien formado.
- **Ruta inexistente:** un 404 porque la ruta no existe, no porque el recurso sea de otro tenant.
- Verifica el motivo: inspecciona el cuerpo del error, o construye el caso para que el ÚNICO motivo posible de rechazo sea el que pruebas.

## Regla 4 — Contra dependencias reales, no mocks

- Los tests de seguridad corren contra Postgres/Redis reales (no mocks), porque lo que pruebas es el comportamiento real de RLS, constraints de unicidad, transacciones y colas. Un mock no tiene RLS ni constraints.
- Si mockeas la DB, no estás probando el aislamiento; estás probando tu mock.

## Regla 5 — Cada invariante crítica tiene su test de comportamiento

Antes de cerrar una feature, mapéala a las invariantes y asegúrate de que existe el test adversarial de cada una que toque:

1. **Aislamiento multi-tenant:** tenant A no ve/afecta datos de B (404, y efecto nulo en DB). Ver saas-multitenant.md.
2. **No doble-reserva / unicidad:** concurrencia probada (N requests simultáneos al mismo recurso → exactamente 1 gana, verificado en DB).
3. **Idempotencia de jobs:** reintentar el mismo job no duplica el efecto (envío/cobro/reserva).
4. **Auth default-closed:** una ruta sin token es rechazada; una excepción pública es intencional.
5. **Enforcement server-side:** un cliente que ignora la UI y manda la petición prohibida directo, falla.
6. **Modo humano (si hay bot):** en modo HUMANO el bot NO emite mensaje (verificar que no se creó/envió el mensaje).

## Regla 6 — Concurrencia se prueba de verdad

Para invariantes de unicidad (no doble-reserva, un solo ganador):
- Lanza N peticiones concurrentes al mismo slot/recurso.
- Verifica en la DB que exactamente 1 tuvo éxito y N-1 fallaron.
- Esto valida que la garantía vive en un constraint de DB, no en lógica de app que pierde la carrera. Si el test pasa con 1 request pero no lo corres con N, no probaste concurrencia.

## Regla 7 — Fixes simétricos

Cuando corriges un patrón defectuoso (dedupe con catch-en-tx, token sin cifrar, query sin filtro de tenant, secreto en log), el MISMO patrón suele vivir en otros apps del monorepo (api, worker, shared) y en otros módulos.
- Grep el patrón en todo el repo.
- Corrígelo en todos los lugares.
- Agrega/extiende el test para cubrir cada ubicación.
- Arreglar solo uno deja el hueco abierto y da la falsa sensación de estar resuelto.

## Regla 8 — Sin `as any` que oculte tipos en lógica de seguridad

Un cast que silencia al compilador puede esconder el bug (un valor que no es lo que crees, un campo que no existe). Si el tipo estorba, es señal de que algo está mal, no de que el tipo sobra. Prohibido `as any`/`# type: ignore` en código de enforcement.

## Checklist antes de declarar "probado y seguro"

1. [ ] Cada test de seguridad pasa la prueba de fuego (rojo si se quita el enforcement).
2. [ ] Los tests verifican el efecto en DB/estado, no solo el status.
3. [ ] El rechazo llega por la razón correcta (aislado de CSRF/throttler/validación/ruta).
4. [ ] Corren contra dependencias reales.
5. [ ] Existe test adversarial por cada invariante crítica que toca la feature.
6. [ ] La concurrencia está probada con N peticiones donde aplique.
7. [ ] Los fixes se aplicaron simétricamente y hay test en cada ubicación.
