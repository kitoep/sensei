# Decomposition: from design to executable tasks

## Rule 1 — Every task has a verifiable deliverable

A task is well-defined only if you can answer: **"what command do I run or what do I observe to know it's done?"**. If there's no answer, it's not a task, it's an intention.

| ❌ Bad (intention) | ✅ Good (verifiable deliverable) |
|---|---|
| "Make progress on the reports module" | "Endpoint `GET /reports/monthly` returns the correct aggregate for the month with seed data; integration test green" |
| "Improve validation" | "Requests with an invalid `email` get 400 with a field message; test proves it with 3 invalid payloads" |
| "Set up the database" | "Migration applied; `npm run db:migrate` runs clean locally and in CI; table visible with a unique constraint on (a, b)" |
| "Integrate service X" | "Real call to X in sandbox returns a parsed response; the timeout and retry are tested by simulating an outage" |

Every task in the plan carries its verification WRITTEN in the plan itself (command, test, or concrete observation). Verification is defined at planning time, not at execution — defining it afterward invites tailoring it to whatever came out.

## Rule 2 — Order by risk, then by dependencies

1. **First, what can invalidate the plan:** the structural assumption detected in question layer D is verified in task 1 or with a spike before the plan (spike = throwaway code with a concrete question and timebox; its output is an answer, not code).
2. **Then the highest technical uncertainty:** what you don't know how to do goes before what you do, while the cost of pivoting is low.
3. **Last, the mechanical stuff:** CRUD, forms, styles, wiring — the stuff that will surely come out fine, once nothing can force a redo.
4. **Dependencies are a constraint, not the ordering:** within what dependencies allow, risk wins. "Schema first because everything depends on it" is valid; "CRUD first because it's easy" is not.

## Rule 3 — Maximum task size

A task must fit in one work session with full context: **if you can't describe its expected diff in 2-3 sentences, split it**. Signs a task is bloated:

- Its description has a structural "and": "create the endpoint AND the job AND the screen" → 3 tasks.
- It touches more than ~2 layers of the system at once (schema + API + worker + UI) → split by layer, with the integration as its own task.
- Its verification needs paragraphs → each verification sentence is usually a task.

The first task of a feature with UI + backend is almost always the **minimal vertical slice**: the narrowest path that crosses every layer end to end (one field, one button, one real datum). Validate the integration before fattening each layer.

## Rule 4 — When to split into sub-specs

If the described work contains **independent subsystems** (e.g., "chat + billing + analytics"), don't force it into a single plan:

1. Identify the independent pieces and their interfaces to each other.
2. Define the order between pieces (by dependency and by value).
3. Each piece gets its own question → spec → plan → execution cycle.

Hard signal: if the plan exceeds ~10-12 tasks or mixes two different definitions of success, it's two specs.

## Rule 5 — Every task maps to its done evidence

The plan closes with a task → evidence table. When executing, a task without its executed and documented evidence is NOT marked done — "it compiled" is not evidence, "the task's test ran and passed" is. If the evidence fails, the failure is reported as-is; never dressed up.
