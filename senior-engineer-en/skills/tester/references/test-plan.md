# How to derive a feature's test plan

## Step 1 — Map the invariants (before writing a single case)

Answer these questions about the feature in writing. Each answer generates cases:

1. **What must be IMPOSSIBLE?** (double booking, seeing another tenant's/user's data, charging twice, sending the message twice, bypassing a permission). → priority 1–2 cases.
2. **Who must NOT be able to do this?** (role without permission, anonymous user, user from another account, expired token). → adversarial cases.
3. **What happens if N arrive at once?** (same slot, same submit, same webhook retried). → concurrency/idempotency cases.
4. **What enforces the invariant: application logic or a platform constraint/mechanism?** If it's app logic, the case must try to dodge it; if it's a constraint (unique, RLS, FK), the case must actually exercise it (hence: real DB).
5. **What state is left in the system after the operation?** The case verifies that state (row created/absent, job enqueued, message NOT sent), not just the HTTP response.

If you can't state a single invariant, you don't understand the feature yet: read the spec/code before planning.

## Step 2 — Order by fixed priority

| P | Category | What it tests |
|---|----------|---------------|
| 1 | Adversarial / security | The forbidden thing FAILS: no permission → rejected; other tenant → 404 (not 403, so you don't leak existence); malicious input → rejected by the server, not by the UI. |
| 2 | Business invariants | Uniqueness, data isolation, no double booking, valid states. Verifying the EFFECT in the DB, not the status code. |
| 3 | Concurrency and idempotency | N simultaneous requests to the same resource → exactly 1 wins. Retry of the same job/webhook → no duplicated effects. |
| 4 | Happy path | The normal full flow works end to end. |
| 5 | Edge cases | Empties, nulls, length limits, unicode/emoji, timezones and DST, pagination at the boundary, legacy data in old states. |

Not every feature has cases in every category — but the ABSENCE of a category is declared explicitly in the plan ("no concurrency surface because X"), never silently dropped.

## Step 3 — Plan template (exact format)

```markdown
## Test plan — <feature>

**Invariants identified:**
1. <invariant and what enforces it (constraint / guard / logic)>
2. ...

| # | P | Case | Type | How it's run | Expected evidence |
|---|---|------|------|--------------|-------------------|
| 1 | 1 | User without permission X cannot <action> | integration | <test command / concrete request> | 403 for permission (check body.message) AND effect absent in DB |
| 2 | 3 | 10 concurrent requests to the same <resource> | integration | <script/test with Promise.all> | exactly 1 with 201; 9 with 409; a single row in DB |
| ... |

**Categories with no cases and why:** <or "none">
**Litmus test:** for each P1–P2 case, state which enforcement is being exercised
(if <guard/constraint> were deleted, case #N would fail because <reason>).
```

Fill-in rules:

- **Type** ∈ unit / integration / E2E. Anything exercising a constraint, RLS, permissions, or queues is integration or E2E against real dependencies — never unit with mocks.
- **How it's run** is a command or reproducible steps, not "check that it works."
- **Expected evidence** always includes the real effect (state in DB, message sent/not sent, file created), not just the response code.
- A P1/P2 case whose test would stay green without the enforcement is a badly designed case: redesign it before running.

## Step 4 — Decide execution

- Plan ≤ ~8 cases and a single area: run it yourself directly.
- Plan with independent areas (API / UI / concurrency) or > ~8 cases: dispatch subagents with briefs from `subagent-briefs.md`, one per area, never two touching the same state.
- If the feature has a UI: the UI cases come from `ui-testing.md` and are part of THIS plan, not an optional appendix.

## Common planning mistakes

- Deriving cases from the list of endpoints/screens → yields only happy paths. Derive from invariants.
- Testing that a flag/signal appears in the response instead of attempting the forbidden action and demanding it fail.
- Writing the concurrency case as sequential (one after another) — that never exercises the race; use genuinely simultaneous requests.
- Trusting that "the framework validates" — the case must send the raw invalid input (curl/HTTP client), bypassing the UI.
- Forgetting the retry case: every job/webhook/handler with retries needs its idempotency case.
