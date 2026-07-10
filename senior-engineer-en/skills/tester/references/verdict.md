# Verdict — the final test report

The verdict is the only thing the user reads. Without this format, the testing work doesn't exist.

## Sign-off rule (no exceptions)

**Forbidden to declare the feature "tested" if any priority 1 or 2 case failed or wasn't run.** In that state the verdict is **NOT TESTED** with the list of what's pending/failing — even if the other 20 cases passed, even if there's a rush, even if it'll "surely pass." A P1–P2 case that's not runnable due to the environment is reported as a blocker, not dropped.

## Template (exact format)

```markdown
## Test verdict — <feature>

**Overall result:** TESTED / NOT TESTED / TESTED WITH RISKS (only P3–P5 pending)
**Summary:** X passed / Y failed / Z not runnable

| # | P | Case | Result | Evidence |
|---|---|------|--------|----------|
| 1 | 1 | <case> | PASS | `npm test -- auth.spec` → "12 passed" + SELECT shows 0 cross-tenant rows |
| 2 | 3 | <case> | FAIL | see Failures |
| 3 | 5 | <case> | NOT RUNNABLE | STRIPE_KEY missing in test env |

### Failures (as-is, with repro)
- **Case #2:** <exact command> → <full error output, not summarized or softened>
  → state left behind: <e.g. 2 rows in bookings for the same slot>
  → repro: <minimal steps>

### Honest coverage — what was NOT tested and what risk it leaves open
- <area/case not covered> → risk: <what could fail in production without us knowing>
- <or: "the plan ran in full; no known gaps">
```

## Fill-in rules

1. **Evidence = command + relevant output snippet.** A bare "PASS ✅" is not a result — it's an unsupported claim and it voids the verdict.
2. **Failures are reported as-is:** full error output, no "almost passed", "minor detail", "would only fail in a rare case." Grading severity is the reader's job; yours is the evidence.
3. **"Not runnable" carries a concrete reason** (missing env, service down, nonexistent selector) and what's needed to unblock it.
4. **The honest-coverage section is mandatory** even if it ends up "no known gaps" — writing it forces you to think it through.
5. If, during testing, a bug is found OUTSIDE the plan's scope, it's listed under coverage/failures with a note "out of scope, not fixed" — never fixed silently or dropped.

## Forbidden rationalizations at sign-off

| Excuse | Reality |
|---|---|
| "2 fail but they're edge cases" | If they're P1–P2, the feature is NOT TESTED. If they're P4–P5, the verdict is TESTED WITH RISKS and lists them. |
| "Couldn't run the concurrency one, but the logic looks right" | Concurrency is P3 and its guarantee is a constraint: without running it there's no positive verdict on that invariant. |
| "The test fails because of the environment, it'll pass in CI" | Then the result is NOT RUNNABLE + blocker, not a preemptive PASS. |
| "I already reported the failures in chat, the template isn't needed" | The template IS the auditable deliverable. Without it, the sign-off didn't happen. |
