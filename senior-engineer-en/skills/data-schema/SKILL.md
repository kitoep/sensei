---
name: data-schema
description: Use when designing or modifying database schemas, creating tables or models, writing or reviewing migrations, adding columns or indexes, choosing column types or primary keys, or when the user says "create the table", "add a column", "I need a migration", "change the column type"; also before running any ALTER TABLE against a production database.
---

# Data-schema — schemas and migrations that don't break prod

## Principle

The schema is the most expensive contract in the whole system to change. A code bug is fixed with a deploy; a badly designed schema is fixed with data migrations, downtime, and weeks of work. So: **the DB is the final guarantee of integrity (not the app), every migration assumes the old app is still running while you migrate, and no design starts without knowing the queries that will read it.**

**Violating the letter of these rules is violating their spirit.** There are no tables "too simple" nor migrations "too obvious" for the process.

## Protocol (load the reference BEFORE acting on its phase)

| Situation | What you do | GATE to proceed | Reference |
|---|---|---|---|
| Design a new table/model | Intake of queries, volume, tenancy, and lifecycle; then types and naming by rule | You answered the intake (or asked it) and every column type is justified by the type table | `references/design.md` |
| Any ALTER/migration | Classify the operation (safe vs dangerous) and apply the expand/contract procedure | The migration runs with the old app alive, no long lock, and has a written rollback plan | `references/migrations.md` |
| Define columns/relationships | Declare constraints in the DB: NOT NULL, FK, UNIQUE, CHECK | Every inviolable business rule has its constraint in the DB, not just app validation | `references/constraints.md` |
| Slow queries or new table with reads | Derive indexes from the real query pattern | Every index traces to a concrete query and you verified it with EXPLAIN | `references/indexes.md` |

A typical change touches several rows: new table = design + constraints + indexes; new column in prod = design (type) + migration + constraint.

## Red flags — STOP if you catch yourself thinking this

- "I'll create the table and figure out the queries later" → the schema is designed FROM the queries. Intake first.
- "I'll just rename the column in the migration" → an in-place rename breaks the deployed old app. Expand/contract.
- "The app already validates that, I don't need the constraint" → the app has bugs, runs concurrently, and isn't the DB's only client. The constraint goes in.
- "`FLOAT` for the price is fine" → money is `NUMERIC`/integer cents. Always.
- "I'll just slap `NOT NULL` on the table" → on a large table that's a full lock/scan. Phased procedure.
- "I'll add an index on every column just in case" → every index costs on every write. Indexes trace to queries.
- "It's a small table, doesn't matter" → today's small tables are tomorrow's 50M-row tables nobody can migrate.

## Known rationalizations

| Excuse | Reality |
|---|---|
| "I don't know the future volume, I'll design it generic" | Not knowing is an intake answer: ask, or write down the assumption. Generic = implicit decisions nobody reviewed. |
| "Expand/contract is a lot of work for a rename" | A direct rename with the old app running = 500s in prod for the whole deploy. Expand/contract is 2 migrations; the incident is your night. |
| "Constraints slow the DB down" | A FK/CHECK costs microseconds per write. Corrupt data costs a cleanup script, a customer incident, and never being able to trust your own data again. |
| "I'll add the indexes later when it's slow" | "Slow" in prod = customers already affected. Indexes for known queries go in from day 1; speculative ones, never. |
| "The ORM generates the migration, that's enough" | The ORM generates the naive ALTER (lock, rewrite, destructive rename). You're responsible for reading the generated SQL and classifying it with `references/migrations.md`. |
| "I'll just dump it all in a JSON" | JSON is for data without a stable schema that you don't filter or join on. Any field you query, validate, or relate is a column. |
