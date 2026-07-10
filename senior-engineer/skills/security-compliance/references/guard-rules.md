# GUARD — Reglas no-negociables al escribir código

Estas reglas se aplican MIENTRAS escribes la feature, no después. Cada una es un "si X entonces Y" verificable. Si el compilador o el linter te estorba al cumplirlas, es señal, no obstáculo — nunca las evites con `as any`/`# type: ignore`/casts que oculten tipos.

## 1. Nunca confiar en input del cliente

- Todo dato que entra (body, query, params, headers, payload de webhook, mensaje de cola, archivo subido) se valida contra un schema/DTO en el servidor ANTES de usarse. Tipos, rangos, longitudes, formato, enum válidos.
- La UI que "ya valida" NO cuenta. Escribe la validación del lado servidor asumiendo un cliente que ignora el frontend.
- Regla: si no puedes señalar dónde se valida un campo antes de tocar la DB, no está validado.

## 2. Identidad y tenant JAMÁS del cliente (regla dura)

- El identificador del tenant (business_id, org_id, account_id) y del usuario NUNCA se aceptan del body/query/params/headers custom.
- **Requests HTTP:** derivan del token verificado (`user.tenant_id` puesto por el guard de auth).
- **Webhooks:** derivan de un mapeo server-side verificado (ej. `provider_id → tenant`), nunca de un campo del payload.
- **Jobs/colas:** el tenant viaja en el payload que TÚ encolaste desde código ya autenticado, y se re-aplica el contexto de tenant al procesar.
- Si ves `req.body.tenantId` o equivalente usándose para filtrar/escribir, es un bug de aislamiento. Detente y corrígelo.

## 3. Auth default-closed

- El guard de autenticación es **global por defecto**. Las rutas públicas son la excepción, marcadas explícitamente (`@Public`/allowlist) y justificadas.
- Nunca "abrir todo y proteger algunas rutas". Siempre "cerrar todo y abrir las mínimas".
- Cada excepción pública lleva un comentario de por qué es segura sin auth.

## 4. Autorización: RBAC deny > allow

- Permiso denegado gana sobre permiso concedido cuando hay conflicto.
- El permiso se valida en el servidor en cada acción, no solo al pintar la UI.
- Los identificadores de recurso en la ruta (`/orders/:id`) se validan contra el tenant/usuario del token: el recurso debe pertenecer al solicitante. Sin esto tienes IDOR (ver attack-surface.md).
- Para SaaS: aplica la segregación completa de `saas-multitenant.md`.

## 5. Secretos: cifrados en reposo, nunca en logs, nunca en el repo (regla dura)

- Tokens de terceros, API keys, credenciales OAuth, refresh tokens: se guardan **cifrados** (AES-256-GCM con clave de entorno, o un secrets manager). Nunca en texto plano en la DB.
- Refresh tokens/opacos: hasheados (no cifrado reversible si solo necesitas comparar).
- Secretos SOLO desde variables de entorno / vault. Cero secretos hardcodeados. Si encuentras uno en el código o en el historial de git, es un hallazgo y hay que rotarlo.
- Logs: ni PII ni secretos. Loggea `trace_id` + `tenant_id`, no el token ni el email ni el teléfono.
- **Fix simétrico:** si cifras un secreto en un app, busca el MISMO tipo de secreto sin cifrar en los otros apps del monorepo y en shared.

## 6. Queries siempre parametrizadas

- Nada de concatenar input en SQL/consultas. Usa parámetros/bindings/query builders del ORM.
- Si necesitas SQL crudo, usa placeholders (`$1`, `?`), nunca interpolación de strings.
- Igual para comandos de shell (usa arrays de args, no string), rutas de archivo (valida contra path traversal), y consultas NoSQL (no pasar objetos del cliente directo a `$where`/filtros).

## 7. Rate limiting en superficies sensibles

- Endpoints de login, registro, recuperación de contraseña, verificación de OTP: rate limit por IP y/o por cuenta.
- Webhooks: validación de firma (HMAC) antes que rate limit; el rate limit no debe tirar tráfico legítimo del proveedor.
- Endpoints costosos (export, reportes, IA): límite por tenant.

## 8. Jobs idempotentes

- Todo job de cola debe ser idempotente: reintentar NO duplica envíos, cobros ni reservas.
- Usa `jobId` determinístico cuando aplique para deduplicar.
- **Dedupe dentro de transacción:** NUNCA con `try/catch` sobre el error de unique violation dentro de una tx — en Postgres una violación de unicidad aborta la transacción entera y toda query posterior falla. Verifica existencia ANTES (`findUnique`/`SELECT`) y decide.

## 9. Concurrencia con garantía en la DB, no en la app

- Invariantes de unicidad/no-doble-reserva se garantizan con **constraints de base de datos** (unique, exclusion), no con lógica de aplicación que puede perder la carrera.
- La lógica de app es la primera barrera; el constraint es la garantía. Ambas.

## Checklist de cierre (ejecutar ANTES de declarar la feature terminada)

Para toda tabla / endpoint / job / feature nueva, marca cada punto o explica por qué no aplica:

1. [ ] ¿Toda entidad nueva lleva `tenant_id` + índice? (si es SaaS)
2. [ ] ¿El tenant/usuario se deriva solo del token/mapeo, nunca del cliente?
3. [ ] ¿Hay validación de input con schema/DTO en el servidor?
4. [ ] ¿La ruta está cubierta por el guard de auth (o es `@Public` justificado)?
5. [ ] ¿Se valida que el recurso pertenece al solicitante (anti-IDOR)?
6. [ ] ¿Los secretos involucrados están cifrados y fuera de logs?
7. [ ] ¿Las queries están parametrizadas?
8. [ ] ¿Si hay migración, incluye la política RLS correspondiente? (si aplica)
9. [ ] ¿Los jobs son idempotentes y el dedupe es pre-check, no catch-en-tx?
10. [ ] ¿Existe el test adversarial que prueba lo DENEGADO? (ver adversarial-testing.md)
11. [ ] ¿Busqué el mismo patrón en los otros apps para un fix simétrico?
12. [ ] ¿PII/retención cubiertos si la feature toca datos personales? (ver data-protection.md)

Un punto sin marcar y sin justificación = la feature NO está terminada.
