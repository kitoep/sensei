# Safe migrations — expand/contract, locks, and rollback

**Supreme rule: every migration runs while the OLD version of the app is still alive** (non-atomic deploy, rollback possible, lagging replicas). A migration that only works once the new code is deployed is badly designed.

## Expand/contract doctrine

No incompatible change happens in one step. Always:

1. **EXPAND** — add the new without touching the old (new column, new table, new index). Compatible with old and new app.
2. **MIGRATE** — deploy code that writes to both sides (dual-write) and backfill historical data in batches.
3. **SWITCH** — deploy code that reads from the new. Verify (counts, checksums) before continuing.
4. **CONTRACT** — only when nothing reads/writes the old (verify it: logs, grep the repo, a grace period), drop the old in a separate later migration.

Each step is an independent, reversible deploy. If step N fails, steps 1..N-1 are still correct.

## Catalog: dangerous operation → safe procedure

| What you want | Why it's dangerous in one step | Safe procedure |
|---|---|---|
| Rename a column | The old app does SELECT/INSERT with the old name → 500s for the whole deploy | Expand: `ADD COLUMN new`; dual-write + backfill; switch reads; contract: `DROP COLUMN old`. (Alternative: don't rename — the old name plus a comment costs less than 4 deploys.) |
| Change a column type | `ALTER TYPE` rewrites the table with an `ACCESS EXCLUSIVE lock` (minutes on large tables) | New column of the correct type + dual-write + batched backfill + switch + drop. Safe exception in Postgres: widenings without rewrite (`VARCHAR(50)→TEXT`, `NUMERIC(10,2)→NUMERIC(12,2)`). |
| `NOT NULL` on an existing table | Full scan with a lock to validate | Postgres: 1) `ADD CONSTRAINT ck CHECK (col IS NOT NULL) NOT VALID;` 2) `VALIDATE CONSTRAINT ck;` (weak lock) 3) `ALTER COLUMN col SET NOT NULL;` (uses the CHECK, no re-scan) 4) `DROP CONSTRAINT ck;`. First: backfill the existing NULLs. |
| New column with `DEFAULT` | On Postgres ≥11 it's safe (virtual default). On Postgres <11 and old MySQL: full rewrite | Check the version. If there's a rewrite: add NULL column → batched backfill → set default → phased NOT NULL. |
| Add a FK to tables with data | Validates the whole table with a lock | `ADD CONSTRAINT ... FOREIGN KEY ... NOT VALID;` then `VALIDATE CONSTRAINT ...;` in a separate step. |
| Create an index | A plain `CREATE INDEX` blocks writes | `CREATE INDEX CONCURRENTLY` (outside a transaction; if it fails it's left INVALID: drop and retry). MySQL/InnoDB: `ALGORITHM=INPLACE, LOCK=NONE`. |
| `DROP COLUMN` / `DROP TABLE` | Irreversible; something you haven't seen yet reads it (reports, ETL, another service) | Only in the contract phase, after verifying zero reads. Cheap prior step: rename to `_deprecated_<name>` or revoke permissions a week ahead — if nothing breaks, drop. Verified backup/snapshot first. |
| Bulk backfill (`UPDATE table SET ...` without WHERE) | Long lock, giant transaction, replica lag, table bloat | In batches: `UPDATE ... WHERE id BETWEEN a AND b` (1k-10k rows), commit per batch, pause between batches, resumable (store the last processed id). NEVER inside the schema migration's transaction. |
| Change the PK / ID strategy | All of the above at once | Treat it as a project, not a migration: new column + new FKs + dual-write + coordinated switch. Question whether it's really necessary. |

## Execution rules

- **Every schema migration is short.** Schema (fast DDL) and data (slow backfill) go in SEPARATE migrations/processes. A backfill inside the migration = hung deploy and an eternal lock.
- **Postgres: set timeouts when migrating** so the migration dies before it takes down prod:
  ```sql
  SET lock_timeout = '5s'; SET statement_timeout = '60s';
  ```
  If the DDL can't get the lock in 5s (because a long query holds it), it fails and you retry — instead of queueing ALL queries behind your ALTER.
- **Read the SQL your ORM/framework generates** (`--dry-run`, `sqlmigrate`, `--pretend`). The ORM doesn't know how many rows your table has or which app version is live; you do.
- **One migration = one purpose.** Don't mix create table + backfill + drop of something else.

## Rollback plan — written BEFORE running

For every migration, answer in writing:
1. How is it reverted? (a real, tested down migration, or "roll forward" with a corrective migration — DROPs have no honest down: the down recreates the column but not the data).
2. Does the old app work DURING and AFTER this migration? (if not: it doesn't satisfy expand/contract, redesign).
3. What do I verify after running it? (row count, `\d table`, the key query with EXPLAIN, zero errors in logs for 10 min).

## Gate checklist — before running in prod

- [ ] Real SQL read (not just the ORM file).
- [ ] Every operation classified against the catalog above; none in the "dangerous in one step" column.
- [ ] Table size checked (`SELECT reltuples::bigint FROM pg_class WHERE relname='...'`) — "small" is a fact, not a hunch.
- [ ] `lock_timeout`/`statement_timeout` set (Postgres).
- [ ] Backfill separate, batched, and resumable (if applicable).
- [ ] Rollback plan written (the 3 questions).
- [ ] Tested against a copy/staging with realistic volume, not an empty DB — on an empty DB EVERYTHING is instant and teaches you nothing.
