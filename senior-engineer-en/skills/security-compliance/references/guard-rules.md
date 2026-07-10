# GUARD — Non-negotiable rules while writing code

These rules apply WHILE you write the feature, not after. Each is a verifiable "if X then Y." If the compiler or linter fights you while you comply, that's a signal, not an obstacle — never dodge them with `as any`/`# type: ignore`/casts that hide types.

## 1. Never trust client input

- Every incoming value (body, query, params, headers, webhook payload, queue message, uploaded file) is validated against a schema/DTO on the server BEFORE use. Types, ranges, lengths, format, valid enums.
- The UI that "already validates" does NOT count. Write the server-side validation assuming a client that ignores the frontend.
- Rule: if you can't point to where a field is validated before it touches the DB, it isn't validated.

## 2. Identity and tenant NEVER from the client (hard rule)

- The tenant identifier (business_id, org_id, account_id) and the user identifier are NEVER accepted from body/query/params/custom headers.
- **HTTP requests:** derive from the verified token (`user.tenant_id` set by the auth guard).
- **Webhooks:** derive from a verified server-side mapping (e.g. `provider_id → tenant`), never from a payload field.
- **Jobs/queues:** the tenant travels in the payload YOU enqueued from already-authenticated code, and the tenant context is re-applied when processing.
- If you see `req.body.tenantId` or equivalent used to filter/write, that's an isolation bug. Stop and fix it.

## 3. Auth default-closed

- The authentication guard is **global by default**. Public routes are the exception, marked explicitly (`@Public`/allowlist) and justified.
- Never "open everything and protect some routes." Always "close everything and open the minimum."
- Every public exception carries a comment on why it's safe without auth.

## 4. Authorization: RBAC deny > allow

- Denied permission wins over granted permission on conflict.
- The permission is checked on the server for every action, not just when rendering the UI.
- Resource identifiers in the route (`/orders/:id`) are validated against the token's tenant/user: the resource must belong to the requester. Without this you have IDOR (see attack-surface.md).
- For SaaS: apply the full segregation in `saas-multitenant.md`.

## 5. Secrets: encrypted at rest, never in logs, never in the repo (hard rule)

- Third-party tokens, API keys, OAuth credentials, refresh tokens: stored **encrypted** (AES-256-GCM with an env key, or a secrets manager). Never plaintext in the DB.
- Opaque/refresh tokens: hashed (not reversible encryption if you only need to compare).
- Secrets ONLY from environment variables / vault. Zero hardcoded secrets. If you find one in the code or in git history, that's a finding and it must be rotated.
- Logs: no PII, no secrets. Log `trace_id` + `tenant_id`, not the token, email, or phone.
- **Symmetric fix:** if you encrypt a secret in one app, grep for the SAME kind of secret left unencrypted in the other apps of the monorepo and in shared.

## 6. Always parameterize queries

- No concatenating input into SQL/queries. Use the ORM's parameters/bindings/query builders.
- If you need raw SQL, use placeholders (`$1`, `?`), never string interpolation.
- Same for shell commands (use arg arrays, not a string), file paths (validate against path traversal), and NoSQL queries (don't pass client objects straight into `$where`/filters).

## 7. Rate limiting on sensitive surfaces

- Login, signup, password reset, OTP verification endpoints: rate limit per IP and/or per account.
- Webhooks: signature validation (HMAC) before rate limit; the rate limit must not drop legitimate provider traffic.
- Expensive endpoints (export, reports, AI): per-tenant limit.

## 8. Idempotent jobs

- Every queue job must be idempotent: retrying does NOT duplicate sends, charges, or reservations.
- Use a deterministic `jobId` where applicable to dedupe.
- **Dedupe inside a transaction:** NEVER via `try/catch` on the unique-violation error inside a tx — in Postgres a uniqueness violation aborts the entire transaction and every subsequent query fails. Check existence BEFORE (`findUnique`/`SELECT`) and decide.

## 9. Concurrency guaranteed in the DB, not the app

- Uniqueness / no-double-booking invariants are guaranteed by **database constraints** (unique, exclusion), not by application logic that can lose the race.
- App logic is the first barrier; the constraint is the guarantee. Both.

## Definition of done checklist (run BEFORE calling the feature complete)

For every new table / endpoint / job / feature, check each item or explain why it doesn't apply:

1. [ ] Does every new entity carry `tenant_id` + index? (if SaaS)
2. [ ] Is the tenant/user derived only from the token/mapping, never from the client?
3. [ ] Is there server-side input validation with a schema/DTO?
4. [ ] Is the route covered by the auth guard (or a justified `@Public`)?
5. [ ] Is it validated that the resource belongs to the requester (anti-IDOR)?
6. [ ] Are the involved secrets encrypted and kept out of logs?
7. [ ] Are the queries parameterized?
8. [ ] If there's a migration, does it include the matching RLS policy? (if applicable)
9. [ ] Are the jobs idempotent, and is the dedupe a pre-check, not a catch-in-tx?
10. [ ] Does the adversarial test that exercises what's DENIED exist? (see adversarial-testing.md)
11. [ ] Did I grep the same pattern in the other apps for a symmetric fix?
12. [ ] PII/retention covered if the feature touches personal data? (see data-protection.md)

One unchecked item with no justification = the feature is NOT done.
