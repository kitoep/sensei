# Systematic diagnosis (Phases 1–2)

## Phase 1 — REPRODUCE FIRST (hard gate)

**Rule:** if you can't reproduce the bug deterministically, you do NOT have permission to propose fixes. None. Not even "preventive fixes just in case".

You clear the gate when you have both things:
1. **Exact command** that triggers the bug (or a script/test that triggers it), runnable by anyone.
2. **Real error output pasted** (stack trace, HTTP response, wrong state in DB) — not a description from memory.

Procedure:
1. Run the case the user reports EXACTLY as they describe it, with their data if they gave it.
2. If it reproduces: freeze the reproduction as a script or test (`repro.sh`, a `.skip`-ed test, a saved curl). This reproduction is your success criterion in Phase 5.
3. If it does NOT reproduce: the difference between your environment and the user's IS the first clue. Compare: version/commit, environment variables, data, user permissions/role, timezone, prior state. Don't move on until you close that gap or ask the user for the exact missing detail.

### Reproducing the hard stuff

| Bug type | Technique |
|---|---|
| Concurrency (double booking, race) | Fire N simultaneous requests at the same resource (`Promise.all` of N calls, or `xargs -P`); the bug shows up when >1 wins. Repeat 20+ times: one clean run proves nothing. |
| Timing/flaky | Run the test in a loop until it fails (`for i in $(seq 50); do npm test -- -t "name" || break; done`). Add artificial latency (a sleep at the suspicious point) to widen the race window. |
| State-dependent | Capture the exact prior state (dump of the involved rows, fixture). The bug that "only happens sometimes" is usually "only happens with this state". |
| Prod-only | Replicate the data (anonymized) and prod config locally. If impossible, add targeted logging (see bisection) and wait for the next occurrence — but say so explicitly, don't guess. |
| Date/time-dependent | Pin the clock (mock `Date.now`, `jest.useFakeTimers`, TZ variable). Bugs that "only fail at month-end / DST change" reproduce by pinning that date. |

## Phase 1.5 — Actually read the error

Before hypothesizing:

1. **Read the full message**, word for word. Not what you expect it to say: what it says. 30% of bugs are solved right here.
2. **Read the full stack**, not just the first frame. Find the deepest frame that is YOUR code (not framework).
3. **Look for the FIRST cause, not the last.** In logs with multiple errors, the last is usually a consequence (connection closed, transaction aborted). Scroll up to the first error in the chain; that's the one you diagnose.
4. **Chained errors:** `caused by`, `errno`, internal codes (`P2002`, `ECONNRESET`, `EADDRINUSE`) — look up the exact code in the tool's docs before theorizing.

## Phase 2 — Single falsifiable hypothesis

**Rule:** ONE hypothesis at a time, written before touching code, with the experiment that kills it.

Mandatory format (write it out literally in your reasoning or a progress comment):

```
HYPOTHESIS: the expired hold isn't released because the expiration job uses
            the server's date and not UTC.
PREDICTS:   if I create a hold with expiration in a past UTC-6, the job does NOT release it.
EXPERIMENT: create a hold with expiresAt = now()-1h in UTC, run the job, check the row.
IF THE PREDICTION FAILS: the hypothesis is dead; do NOT patch it with sub-clauses.
```

Rules:
- The experiment must be able to **kill** the hypothesis. If any result "confirms" it, it's not an experiment.
- Minimal experiment: the smallest change/query that distinguishes your hypothesis from the rivals. A `SELECT`, a log, a unit test — not a refactor.
- Dead hypothesis = dead. Note it in a discarded list (with the evidence) and form the next one. This list is gold when you escalate or get stuck.
- **Confirmed ≠ done.** Confirmed moves you to Phase 3 (root cause), not to the fix.

## Bisection — when there's no clear hypothesis

Cut the problem space in half repeatedly:

- **In time:** `git bisect start; git bisect bad HEAD; git bisect good <last-good-commit>` + your reproduction script as the criterion. Finds the culprit commit in log₂(n) steps.
- **In code:** comment out/disable half the flow (middleware, hooks, pipeline steps). Bug still there? The cause is in the active half. Repeat.
- **In data:** does it fail with the full dataset? Try with half. Find the exact record that triggers the bug.
- **With targeted logging:** log the suspicious value at 3 points in the flow (input, transformation, output). The point where the value gets corrupted bounds the guilty half. Delete these logs when done.

## Absolute prohibitions in diagnosis

1. **Don't change things "to see if it works".** Every change without a hypothesis contaminates the state and can mask the bug.
2. **Don't add try/catch so it "doesn't crash".** Silencing the symptom is not fixing; it's hiding the body.
3. **Don't touch the failing test to make it pass** (not the assert, not the timeout, not `.skip`). The test is the evidence.
4. **Don't declare a root cause without evidence that distinguishes it from the rivals.** "It's definitely X" with two live hypotheses = you don't know which it is.
5. **Don't fix two things at once.** One fix per cause, verified separately.
