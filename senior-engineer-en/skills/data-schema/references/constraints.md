# Constraints — the DB as the final guarantee

**Principle: the application validates to give good error messages; the database guarantees that corrupt data is IMPOSSIBLE.** The app has bugs, runs in N concurrent processes, and is never the DB's only client (console, ETL, scripts, the next microservice). A rule that lives only in the app is a suggestion.

## What goes in the DB vs what goes in the app

| Rule | Constraint in DB? | Validation in app? |
|---|---|---|
| "This field always exists" | `NOT NULL` — yes, always | Yes, for the form's error message |
| "The email is unique" | `UNIQUE` — yes, always (the app CANNOT guarantee it: race between the SELECT-check and the INSERT) | Yes, for UX; and handle the unique violation as an expected case |
| "The order belongs to a customer that exists" | `FOREIGN KEY` — yes, always | Usually implicit via the ORM |
| "The total isn't negative", "status ∈ {…}" | `CHECK` — yes | Yes |
| "The discount requires manager approval" | No (process logic) | Yes — workflow/permission rules live in the app |
| "The phone number is nicely formatted" | No (presentation format) | Yes — normalize before storing |

Quick rule: if violating it leaves the DB in a state no legitimate flow produces → constraint. If it depends on who/when/how → app.

## Catalog with exact rules

### NOT NULL — the default, not the exception
Every column is `NOT NULL` unless NULL carries a business meaning you can say out loud ("`deleted_at` NULL = alive", "`paid_at` NULL = not paid"). "In case they don't send it" is not a meaning: that's a default or an app bug.

### UNIQUE — where the app mathematically can't
Every "there can't be two X with the same Y" is a `UNIQUE`/unique index. Cases people always forget:
- Composite and per-tenant uniqueness: `UNIQUE (tenant_id, email)` — the email repeats across tenants, not within one.
- With soft-delete, partial unique: `CREATE UNIQUE INDEX uq_users_email ON users(tenant_id, email) WHERE deleted_at IS NULL;`
- Idempotency of external writes (webhooks, jobs): `UNIQUE (provider, external_id)` turns the duplicate event into a controlled error instead of a double row.

### FOREIGN KEY — with an explicit ON DELETE decision
Every `*_id` column that references another table carries a FK. The part that separates the senior: choosing `ON DELETE` on purpose, per relationship:

| ON DELETE | Use it when | Example |
|---|---|---|
| `RESTRICT` (sane default) | Deleting the parent with children is a business error | You don't delete a customer with orders |
| `CASCADE` | The child makes no sense without the parent | `order_items` when deleting `orders` |
| `SET NULL` | The relationship is optional and the child survives | `assigned_agent_id` when deleting the agent |

`CASCADE` on long chains is a bomb: audit what it would drag before setting it. And every FK column needs an index (see `indexes.md`).

### CHECK — row invariants
```sql
CHECK (amount >= 0),
CHECK (status IN ('pending','paid','shipped','cancelled','refunded')),
CHECK (ends_at IS NULL OR ends_at > starts_at),
CHECK ((paid_at IS NULL) = (payment_method IS NULL))   -- fields that go together or not at all
```
States/enums ALWAYS with CHECK (or a lookup table + FK if the business edits the set). Adding a new state = altering the CHECK in a migration: that's a feature, not a nuisance — it forces you to review every place that switches on the state.

### DEFAULT — for machine columns, not business ones
`DEFAULT now()` on timestamps, `DEFAULT false` on flags, `DEFAULT 'pending'` on an initial status. Do NOT use a default to hide that the app forgot to send a business value (a `DEFAULT 0` on `amount` turns a bug into a silent free sale).

## Adding constraints to tables that ALREADY have data

1. Measure the current violation: `SELECT count(*) FROM t WHERE <violates the rule>;`
2. Clean up or migrate those rows (batched script, documented business decision).
3. Add the constraint `NOT VALID` and then `VALIDATE CONSTRAINT` (Postgres) to avoid locking (see `migrations.md`).

If step 1 returns > 0 and "there's no time to clean up": the constraint is NOT postponed indefinitely — create it `NOT VALID` (protects new rows right away) and the cleanup becomes a task with an owner.

## Handling in the app

Constraints will fire errors (unique violation, FK violation): the app catches them and translates them to useful messages. A raw 500 on a unique violation isn't a reason to drop the constraint — it's a reason to handle the error. In concurrent flows use the constraint as the mechanism: `INSERT ... ON CONFLICT DO NOTHING/UPDATE` instead of SELECT-then-INSERT.
