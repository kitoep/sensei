# Aislamiento multi-tenant y RLS

Principio inviolable de un SaaS: **cero cruce de datos entre tenants.** Un tenant jamás ve, modifica ni infiere la existencia de datos de otro. Esto no es una feature; es la propiedad que, si falla una vez, quiebra el producto.

## Defensa en capas (las tres barreras, no una)

El aislamiento se sostiene con tres barreras independientes. Ninguna sola es suficiente.

### Barrera 1 — Modelo de datos

- **Toda entidad de negocio lleva `tenant_id`** (business_id/org_id/account_id) con índice. Regla dura: si dudas si una tabla nueva lo lleva, lo lleva. Única excepción de diseño: tablas de identidad global fuera del scope de tenants, que NUNCA se exponen por endpoints de tenant.
- Las FKs no cruzan tenants: un registro hijo pertenece al mismo tenant que su padre.

### Barrera 2 — Derivación del tenant en el código (regla dura)

- El `tenant_id` **nunca** viene del cliente (body/query/params/headers). Solo de:
  - **HTTP:** token verificado por el guard de auth.
  - **Webhooks:** mapeo server-side verificado (`provider_id → tenant`), nunca un campo del payload.
- Todo query de negocio filtra por el tenant derivado. Un `findMany` sin filtro de tenant es un bug de aislamiento.

### Barrera 3 — RLS (Row-Level Security) en la base de datos

Segunda muralla para cuando el código falla (un query sin filtro, un ORM mal usado):

1. Las tablas de negocio tienen políticas RLS que filtran por una variable de sesión: `current_setting('app.current_tenant_id')`.
2. El rol de la aplicación en runtime **NO tiene bypass de RLS** (nada de `BYPASSRLS`, no es superusuario/owner). Las migraciones corren con un rol distinto (owner) vía una conexión separada.
3. Toda operación de negocio corre dentro de una transacción que hace `set_config('app.current_tenant_id', <tenant>, true)` con alcance de transacción (el `true` = local a la tx). En HTTP esto lo pone un interceptor cuando hay usuario autenticado; en webhooks/worker se llama explícito.
4. Los repositorios obtienen el cliente de DB del contexto de la transacción activa (AsyncLocalStorage o equivalente) — nunca el cliente raíz directo en código de negocio, porque ese no tiene el contexto de tenant seteado.

Verificación rápida de que RLS está viva: intenta un `SELECT` con el rol de app sin setear la variable de sesión → debe devolver 0 filas, no todas.

## No filtrar existencia: 404, no 403 (regla dura)

Cuando un usuario pide un recurso de otro tenant, la respuesta es **404 (no existe)**, no 403 (existe pero no puedes). Un 403 confirma que el ID existe en otro tenant — es una fuga de información. El test de aislamiento debe esperar 404.

## Segregación de permisos y accesos

- **Roles y permission keys:** RBAC con `deny > allow`. Los permisos se validan en el servidor por acción, no solo al pintar la UI.
- **Sesiones revocables:** access token corto (ej. 15 min) + refresh token opaco hasheado en tabla de sesiones, revocable. Logout y cambio de rol revocan las sesiones del usuario. El refresh rota en cada uso y recarga permisos frescos.
- **2FA:** TOTP obligatorio para roles administrativos; opcional para el resto.
- **Principio de mínimo privilegio:** cada rol tiene el mínimo de permisos para su función. Ningún rol "por si acaso".
- Un cambio de permisos/rol debe surtir efecto sin esperar a que expire el token (revocación de sesión).

## Test de aislamiento cross-tenant (obligatorio, adversarial)

Sin este test, la feature multi-tenant NO está terminada. Debe ser adversarial (ver adversarial-testing.md), no confirmatorio:

1. Crea datos de DOS tenants (A y B) con dependencias reales (Postgres real, no mocks).
2. Autentícate como usuario de A.
3. Intenta **leer** un recurso de B por su ID real → espera **404**.
4. Intenta **modificar/borrar** un recurso de B → espera 404 y verifica en la DB que el registro de B **no cambió**.
5. Intenta **crear** un recurso inyectando `tenant_id` de B en el body → verifica que se creó bajo A (el del token), no bajo B.
6. Verifica que el 404 llega por RLS/permiso, no por CSRF, throttler, ruta inexistente o un `ParseUUIDPipe` (usa un ID válido y una petición bien formada).

Prueba de fuego: si borras el filtro de tenant del código de negocio, este test debe ponerse ROJO (porque RLS lo atrapa). Si sigue verde con y sin el filtro, revisa que RLS esté realmente activa y el rol sin bypass.

## Checklist para toda tabla/endpoint nuevo en SaaS

1. [ ] `tenant_id` + índice en la tabla.
2. [ ] Migración con política RLS para la tabla.
3. [ ] Todos los queries corren dentro del contexto de tenant (transacción con `set_config`).
4. [ ] Ningún query de negocio usa el cliente raíz de DB.
5. [ ] El endpoint deriva el tenant del token, nunca del cliente.
6. [ ] Test de aislamiento cross-tenant (los 6 pasos de arriba).
7. [ ] La tabla se agregó a los helpers de limpieza de tests en su posición FK-segura.
