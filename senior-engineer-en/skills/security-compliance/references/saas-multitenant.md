# Multi-tenant isolation and RLS

Inviolable principle of a SaaS: **zero cross-tenant data leakage.** A tenant never sees, modifies, or infers the existence of another tenant's data. This isn't a feature; it's the property that, if it fails once, breaks the product.

## Defense in depth (three barriers, not one)

Isolation rests on three independent barriers. None alone is sufficient.

### Barrier 1 — Data model

- **Every business entity carries `tenant_id`** (business_id/org_id/account_id) with an index. Hard rule: if you're unsure whether a new table carries it, it carries it. Only design exception: global identity tables outside tenant scope, which are NEVER exposed through tenant endpoints.
- FKs don't cross tenants: a child record belongs to the same tenant as its parent.

### Barrier 2 — Tenant derivation in code (hard rule)

- The `tenant_id` **never** comes from the client (body/query/params/headers). Only from:
  - **HTTP:** the token verified by the auth guard.
  - **Webhooks:** a verified server-side mapping (`provider_id → tenant`), never a payload field.
- Every business query filters by the derived tenant. A `findMany` without a tenant filter is an isolation bug.

### Barrier 3 — RLS (Row-Level Security) in the database

Second wall for when the code fails (an unfiltered query, a misused ORM):

1. Business tables have RLS policies that filter by a session variable: `current_setting('app.current_tenant_id')`.
2. The application's runtime role has **NO RLS bypass** (no `BYPASSRLS`, not superuser/owner). Migrations run under a different role (owner) via a separate connection.
3. Every business operation runs inside a transaction that does `set_config('app.current_tenant_id', <tenant>, true)` with transaction scope (`true` = local to the tx). In HTTP an interceptor sets it when there's an authenticated user; in webhooks/worker it's called explicitly.
4. Repositories get the DB client from the active transaction's context (AsyncLocalStorage or equivalent) — never the root client directly in business code, because that one doesn't have the tenant context set.

Quick check that RLS is alive: try a `SELECT` with the app role without setting the session variable → it must return 0 rows, not all of them.

## Don't leak existence: 404, not 403 (hard rule)

When a user requests another tenant's resource, the response is **404 (doesn't exist)**, not 403 (exists but you can't). A 403 confirms the ID exists in another tenant — that's an information leak. The isolation test must expect 404.

## Permission and access segregation

- **Roles and permission keys:** RBAC with `deny > allow`. Permissions are checked on the server per action, not just when rendering the UI.
- **Revocable sessions:** short access token (e.g. 15 min) + opaque refresh token hashed in a sessions table, revocable. Logout and role change revoke the user's sessions. The refresh rotates on each use and reloads fresh permissions.
- **2FA:** TOTP required for administrative roles; optional for the rest.
- **Least privilege:** each role has the minimum permissions for its function. No "just in case" role.
- A permission/role change must take effect without waiting for the token to expire (session revocation).

## Cross-tenant isolation test (mandatory, adversarial)

Without this test, the multi-tenant feature is NOT done. It must be adversarial (see adversarial-testing.md), not confirmatory:

1. Create data for TWO tenants (A and B) with real dependencies (real Postgres, not mocks).
2. Authenticate as a user of A.
3. Try to **read** a resource of B by its real ID → expect **404**.
4. Try to **modify/delete** a resource of B → expect 404 and verify in the DB that B's record **did not change**.
5. Try to **create** a resource injecting B's `tenant_id` in the body → verify it was created under A (the token's), not under B.
6. Verify the 404 comes from RLS/permission, not from CSRF, throttler, a nonexistent route, or a `ParseUUIDPipe` (use a valid ID and a well-formed request).

Litmus test: if you remove the tenant filter from the business code, this test must go RED (because RLS catches it). If it stays green with and without the filter, check that RLS is really active and the role has no bypass.

## Checklist for every new table/endpoint in SaaS

1. [ ] `tenant_id` + index on the table.
2. [ ] Migration with an RLS policy for the table.
3. [ ] All queries run inside the tenant context (transaction with `set_config`).
4. [ ] No business query uses the root DB client.
5. [ ] The endpoint derives the tenant from the token, never from the client.
6. [ ] Cross-tenant isolation test (the 6 steps above).
7. [ ] The table was added to the test cleanup helpers in its FK-safe position.
