---
name: tester
description: Use when a feature or bugfix needs a test plan, when verifying work before closing or shipping it, when asked "how do we test this" or to prove something works, when designing QA coverage for backend or frontend changes, or when running E2E/UI tests (Playwright or similar). Also use before declaring any feature "done", "tested", or "ready to ship".
---

# Tester — test planning and execution with evidence

## Principle

A test plan is derived from the feature's **invariants** (what must be impossible), not from its list of screens or endpoints. A feature is tested when there is **executed evidence** for every case — never when "the code looks right."

**Violating the letter of these rules is violating their spirit.**

## Required flow

1. **Map the invariants** of the feature → read `references/test-plan.md` and build the plan with its template.
2. **Run the plan**: directly if it's small; if there are independent areas, dispatch subagents → read `references/subagent-briefs.md` to write each brief.
3. **If the feature has a UI**, the plan ALWAYS includes cases from `references/ui-testing.md` (not just backend).
4. **Verdict**: report with the template in `references/verdict.md`. No verdict template, no sign-off.

## Which reference to load

| Situation | File |
|---|---|
| Design the test plan for a feature | `references/test-plan.md` |
| Write instructions for test subagents | `references/subagent-briefs.md` |
| Feature touches front/UI/UX, responsive, forms | `references/ui-testing.md` |
| Report results / decide whether the feature is tested | `references/verdict.md` |

## Hard rules (no exceptions)

1. **Fixed case priority:** (1) adversarial/security → (2) business invariants → (3) concurrency and idempotency → (4) happy path → (5) edge cases. Never start with the happy path.
2. **Litmus test per case:** *"if I delete the enforcement logic, does this test stay green?"* If it does, the test is worthless — redesign it.
3. **Real dependencies:** every case that exercises an invariant (DB constraint, RLS, queue, uniqueness) runs against real Postgres/Redis/DB, not mocks.
4. **Evidence or it didn't happen:** every case reports the command run + the relevant output. "Passed ✅" with no output does not count as executed.
5. **A feature is NOT declared tested** while any priority 1–2 case is failing or unrun.

## Forbidden rationalizations

| Excuse | Reality |
|---|---|
| "It's a small change, it doesn't need a plan" | Small changes break other people's invariants. The plan can be 3 cases, but it exists. |
| "Typecheck/build passed" | Compiling isn't testing. Zero cases run = zero evidence. |
| "I already tested it manually while building" | No reproducible evidence, no test. Redo it with a command/log. |
| "The happy path works, that's good enough" | The expensive bugs live in cases 1–3. The happy path is last. |
| "I'll mock the DB to move faster" | The mock doesn't run the constraint/RLS that IS the guarantee. Test against the real thing. |
