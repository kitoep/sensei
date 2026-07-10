# Indexes — derived from queries, verified with EXPLAIN

**Rule: every index traces to a concrete query you can write; every frequent query traces to an index that serves it.** An index with no query is pure cost (every INSERT/UPDATE maintains it); a frequent query with no index is a full scan that grows with the table.

## Method (in this order)

1. **List the real queries** for the table (from the intake in `design.md` or from the code: find the `WHERE`, `JOIN`, `ORDER BY` on the table).
2. **Design the index for the query**, with the rules below.
3. **Verify with EXPLAIN** that the query uses it. Without EXPLAIN, the index is a hope.

## Design rules

### Composite index: the column order IS the design
Order: **equality first → range/order after**.

```sql
-- Query: WHERE tenant_id = ? AND status = ? ORDER BY created_at DESC LIMIT 20
CREATE INDEX idx_orders_tenant_status_created
  ON orders (tenant_id, status, created_at DESC);
```
- Serves prefixes: it also covers `WHERE tenant_id = ?` alone. It does NOT serve `WHERE status = ?` without tenant_id (not a prefix).
- An index `(a, b)` makes an index `(a)` redundant — don't keep both.
- The range column (`created_at > ?`, `ORDER BY created_at`) goes LAST; anything after a range in the index no longer filters efficiently.
- Multi-tenant: `tenant_id` is the first column of nearly every index, because nearly every query filters by tenant.

### Every FK gets an index
Postgres does NOT index FKs automatically (MySQL/InnoDB does). Without an index on the FK: JOINs scan and every parent DELETE scans the entire child table (with a lock). When you create a FK, create its index in the same migration — or justify it in writing.

### Partial indexes — for the subset you query
When you always query the same slice of the table:
```sql
-- You only query the pending ones; the 95% historical doesn't clutter the index
CREATE INDEX idx_jobs_pending ON jobs (created_at) WHERE status = 'pending';
-- Unique with soft-delete (see constraints.md)
CREATE UNIQUE INDEX uq_users_email ON users (tenant_id, email) WHERE deleted_at IS NULL;
```
Smaller, faster, cheaper to maintain. MySQL doesn't have them: use a normal index and accept the cost.

### Other cases with a fixed rule
- Free-text search (`ILIKE '%x%'`): a B-tree does NOT help. Postgres: `pg_trgm` + GIN index, or full-text search. Don't add a normal index "to see if it helps".
- Filters inside JSONB: a GIN index on the column or an expression index on the key (`CREATE INDEX ... ON t ((payload->>'type'))`) — only if the query exists and you measured it.
- Low cardinality alone (boolean, a 3-value status) is NOT indexed alone: it rarely filters enough. It goes as a column of a composite or the condition of a partial.

## Write cost — the budget

Every index is updated on every INSERT and on every UPDATE that touches its columns. Guidelines:
- Typical transactional table: 3-6 indexes is normal; 10+ demands query-by-query justification.
- Bulk-ingest table (events, logs, messages): minimal indexes; complex analysis goes to a replica/warehouse.
- Before adding an index to a hot table, list what it already has (`\d table` / `SHOW INDEX FROM table`) and ask: does an existing one already serve as a prefix? are two existing ones redundant with each other?

## Verification with EXPLAIN (the gate)

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ... ;  -- Postgres; MySQL: EXPLAIN ANALYZE
```
Read the plan and confirm:
- An `Index Scan`/`Index Only Scan` using YOUR index appears — not a `Seq Scan` over the big table.
- The `ORDER BY ... LIMIT` is resolved by the index (no `Sort` node over thousands of rows).
- Estimated vs actual `rows` don't differ by orders of magnitude (if they do: run `ANALYZE table;`).

Verification traps:
- On a DB with 100 rows the planner picks Seq Scan even if your index is perfect (it's correct with 100 rows). Verify against realistic volume or at least reason out the expected plan and write it down.
- `EXPLAIN` without `ANALYZE` is the estimate; with `ANALYZE` it actually runs — be careful running it on UPDATE/DELETE in prod (wrap it in `BEGIN; ... ROLLBACK;`).

## In production: create without blocking

`CREATE INDEX CONCURRENTLY` always on live tables (outside a transaction; if it fails it's left `INVALID`: `DROP INDEX` and retry). MySQL/InnoDB: `ALGORITHM=INPLACE, LOCK=NONE`. Details and checklist in `migrations.md`.

## Maintenance — indexes nobody uses anymore

When reviewing performance, besides adding, drop:
```sql
SELECT indexrelname, idx_scan FROM pg_stat_user_indexes
WHERE schemaname='public' ORDER BY idx_scan ASC;  -- idx_scan=0 after weeks = DROP candidate
```
Don't drop the ones backing UNIQUE/PK even with 0 scans: those are constraints, not accelerators.
