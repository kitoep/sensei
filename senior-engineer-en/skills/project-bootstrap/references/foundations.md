# Foundations — cheap on day 1, brutal in month 6

Every item is decided EXPLICITLY: apply it, or dismiss it with a written reason in `docs/01_decisions.md`. Nothing is skipped silently. Format per item: **Rule → What good looks like → Sign it was done wrong → Retrofit cost**.

## 1. Repo structure

**Rule:** a single repo (monorepo with workspaces if there are several apps) until there's a real operational reason to split (separate teams deploying independently).
**Good:** `apps/` (web, api, worker), `packages/` (shared code: types, validation), one lockfile, one CI.
**Wrong:** two repos that have to be versioned in lockstep; types duplicated by hand between front and back.
**Retrofit:** merging separate repos = weeks of git history, CI, and permissions. Splitting a monorepo later is easy; merging repos is not.

## 2. Per-environment config and secrets

**Rule:** from commit 1: config only via environment variables, `.env.example` committed with ALL the keys (no values), `.env` in `.gitignore`, and env validation at boot (the app crashes on startup if a variable is missing, not at 3am when it's used).
**Good:** a single `config` module that reads, validates, and types the env; nobody does a loose `process.env.X` scattered through the code.
**Wrong:** a secret committed "temporarily" (it's in git history FOREVER; rotating it is the only way out); hardcoded values that differ between dev and prod.
**Retrofit:** purging a secret from git history + rotating it across every service. Hours to days, and the exposure already happened.

## 3. CI from commit 1

**Rule:** a pipeline with typecheck + tests + build green BEFORE the first feature. Green is required to merge.
**Good:** CI runs on every PR; it spins up the real dependencies (DB, queue) as services; an `npm audit` (or equivalent) where critical findings block.
**Wrong:** "I'll set up CI when there's more code" — by then there are 40 accumulated typecheck errors nobody wants to pay off.
**Retrofit:** every week without CI accrues invisible debt; installing it late = days fixing everything that would have failed incrementally.

## 4. Tests against real dependencies

**Rule:** integration tests run against the REAL database and queue (docker compose locally and in CI), not mocks. Mocks don't test constraints, transactions, or DB policies.
**Good:** `docker compose up` brings up Postgres+Redis; tests migrate a clean schema and exercise real behavior.
**Wrong:** green suite with mocks + production broken by a constraint or a transaction no mock ever simulated.
**Retrofit:** migrating a mocked suite to real dependencies = rewriting the tests. Starting right costs an afternoon.

## 5. Versioned DB migrations

**Rule:** from the FIRST table, every schema change is a versioned, committed migration. Never `db push` / hand-edits on any shared DB.
**Good:** a migrations folder in the repo; CI migrates from scratch on every run (that proves the whole chain works); two DB roles if there's RLS (runtime with no bypass, migrations with the owner).
**Wrong:** prod schema diverges from the repo and nobody knows since when.
**Retrofit:** reconstructing the real state of a diverged schema is archaeology. Days, with risk of data loss.

## 6. Dates in UTC

**Rule:** the DB stores EVERYTHING in UTC. The user's/business's timezone is presentation data, applied only at the edge (UI, messages).
**Good:** `timestamptz` columns; conversion to local zone only at render time; the business TZ stored as data.
**Wrong:** "naive" datetimes that work until the first user in another zone or the first daylight-saving switch.
**Retrofit:** migrating already-stored ambiguous timestamps is one of the most expensive bugs there is: you don't know which zone each row was in.

## 7. Structured logging without PII

**Rule:** logs in structured JSON with a `trace_id` (per request/job correlation) and a context identifier (e.g. tenant); NEVER log PII, tokens, or secrets from log #1.
**Good:** a central logger the whole app uses; personal data is referenced by ID, never by value.
**Wrong:** `console.log(user)` with email and phone — every log with PII is a data incident waiting to leak.
**Retrofit:** scrubbing PII from historical logs is impossible (it's already replicated to the logging platform); all you can do is purge and hope.

## 8. Global error handling

**Rule:** a global error handler from the skeleton onward: every uncaught exception ends in (a) a generic response to the client WITH no stack or internal detail, (b) a structured log with the full detail and trace_id.
**Good:** typed domain errors that map to HTTP codes; the stack trace never travels to the client.
**Wrong:** 500 with a full stack exposed (a gift to attackers) or, worse, errors swallowed silently.
**Retrofit:** cheap to add late, but every week without it is invisible production errors nobody knew happened.

## 9. Default-closed auth

**Rule:** if there are users, the auth guard is GLOBAL and by default EVERY endpoint requires a session; public routes are marked as explicit exceptions (`@Public` or equivalent). Never the other way around.
**Good:** a new endpoint = protected without anyone having to remember anything; the role model denies by default (deny > allow).
**Wrong:** "I'll add auth to the sensitive endpoints" — the list of sensitive ones is always incomplete, and the forgotten endpoint is the one that shows up in the vuln report.
**Retrofit:** auditing every existing endpoint to flip the default. Days, with guaranteed embarrassing findings.

## 10. Multi-tenancy (if the intake said yes) — THE irreversible one

**Rule:** every business entity carries `tenant_id` (indexed) from creation; the tenant NEVER comes from the client (it's derived from the token or a verified channel mapping); and there's a second barrier at the database level (Row-Level Security with a non-bypass app role, or equivalent) because application discipline breaks once a year and that one time is enough.
**Good:** a mandatory checklist for every new table: tenant_id + index → RLS policy → queries in tenant context → cross-tenant isolation test (tenant A never sees/affects tenant B's data; expect 404, not 403, so you don't leak existence).
**Wrong:** `where tenant_id = ?` filters added "by hand where it applies" — the day a dev forgets it on one endpoint, two businesses see each other's data and the product is dead.
**Retrofit:** CANNOT be done safely. Adding tenancy later = touching every table, every query, every test, and praying. That's why it's asked in the intake.

## 11. Idempotency in async jobs

**Rule:** if there are queues/jobs, every job is idempotent from the first: retries do NOT duplicate effects (emails, charges, bookings). Deterministic `jobId` where it applies; existence check BEFORE inserting (don't catch the unique-constraint error inside a transaction — in Postgres it aborts the whole thing).
**Good:** running the same job 3 times produces exactly the same final state; there's a backup reconciler, but it's not the primary mechanism.
**Wrong:** a queue retry sends the same email/charge to the customer twice.
**Retrofit:** medium — but the duplicates already sent to real customers can't be un-sent.

## 12. Third-party secrets encrypted at rest

**Rule:** tokens/credentials for external services (APIs, customer integrations) are stored encrypted (AES-256-GCM with the key in env), NEVER in plaintext in the DB. And when you fix a secret-handling pattern in one app, search for the SAME pattern in the other apps in the repo (pattern bugs live duplicated).
**Good:** shared `encryptSecret/decryptSecret` helpers; the encryption key is just another env var, rotatable.
**Wrong:** a leaked DB dump exposes the WhatsApp/Stripe tokens of ALL your customers — the incident stops being yours and becomes theirs.
**Retrofit:** medium (encryption migration), but only if you catch it before an attacker does.

## Phase output

`docs/01_decisions.md`: a table with the 12 items, an "apply/dismiss" column, and the reason for each dismissal. Dismissals without a written reason are not valid.
