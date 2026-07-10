# Briefs for test subagents

A subagent without a strict brief drifts: it tests the wrong thing, softens failures, or "fixes" code it shouldn't touch. The brief is a contract — the subagent executes and reports, it does not interpret.

## Required brief structure (the 6 sections, in this order)

```markdown
1. OBJECTIVE (one line): what is being verified and for which feature.
2. SCOPE:
   - IN: exact files/flows/cases it covers.
   - OUT: what NOT to touch or test (other areas are covered by other agents).
   - FORBIDDEN: modifying production code. If a test needs a fix, REPORT it, don't apply it.
3. SETUP: exact setup commands (bring up services, migrate, seed, credentials/env to use).
4. STEPS: numbered, each runnable as-is (concrete command or interaction).
5. PASS/FAIL PER STEP: what output/state = PASS and what = FAIL. Include the real effect to verify (row in DB, message not sent), not just the status.
6. OUTPUT FORMAT: the exact results table (below) + instruction to report failures AS-IS,
   with the full error output, without softening them ("almost passed", "would only fail if...") and without fixing them.
```

Output format the brief requires from the subagent:

```markdown
| # | Case | Result | Evidence |
|---|------|--------|----------|
| 1 | <case> | PASS / FAIL / NOT RUNNABLE | <command + relevant output snippet> |

Failures (exact repro):
- Case #N: <command> → <full error output> → <what state was left in DB/system>
Not runnable: <case and concrete reason (missing env, service down, etc.)>
```

## Parallelization rule

- One subagent per **independent area** (API, UI, concurrency, permissions).
- Never two subagents touching the same state (same mutable test DB, same seed user, same port). If they share state: run sequentially, or give each agent separate data/tenants.
- Concurrency cases go in ONE agent (the race is orchestrated inside the test, not across agents).

## Example 1 — API integration brief

```markdown
OBJECTIVE: Verify the no-double-booking invariant of POST /bookings (scheduling feature).
SCOPE:
- IN: cases 2, 3, and 5 of the plan (slot uniqueness, concurrency, idempotent retry).
- OUT: UI or permission cases (another agent covers those).
- FORBIDDEN: modifying code in src/. If an existing test is wrong, report it.
SETUP:
- docker compose up -d db redis
- npm run db:migrate:test && npm run seed:test
- Env: DATABASE_URL=<...> REDIS_URL=<...>
STEPS:
1. Run: npm test -- bookings.integration
2. Concurrency case: run the test that fires 10 simultaneous POST /bookings (Promise.all)
   to seed slot "slot-A" with 10 distinct clients.
3. Check DB: SELECT count(*) FROM bookings WHERE slot_id='slot-A' AND status='CONFIRMED';
PASS/FAIL:
- Step 2: exactly 1 response 201 and 9 responses 409. Any other distribution = FAIL.
- Step 3: count = 1. count > 1 = CRITICAL FAIL (real double booking), report it first.
OUTPUT: standard table + failures with exact repro.
```

## Example 2 — UI E2E brief

```markdown
OBJECTIVE: Verify the appointment-creation flow from the UI (scheduling feature) with Playwright.
SCOPE:
- IN: full create-appointment flow, form error states, double-click on submit.
- OUT: backend availability logic (covered by the API agent). Don't test other modules.
- FORBIDDEN: modifying components. If a selector doesn't exist, report "NOT RUNNABLE + missing selector".
SETUP:
- npm run dev (wait until http://localhost:3000/health responds)
- Test user: <email>/<pass from seed>. Default viewport 1280x720.
STEPS:
1. Log in → go to /agenda → create an appointment in a visible slot → verify the success toast AND that the
   appointment appears in the calendar (reload the page and confirm it persists).
2. Submit the form with an empty date → verify a VISIBLE error message next to the field
   (not just console) and that the appointment was NOT created (calendar doesn't show it after reload).
3. Rapid double-click on "Book" → verify a SINGLE appointment is created (count cards after reload).
PASS/FAIL:
- Use getByRole/getByTestId, never CSS class selectors. Wait on state (expect(locator)),
  never waitForTimeout. If you need a sleep to make it pass, the result is FAIL (flaky), not PASS.
- Each step: the assertion of the persisted effect (after reload) decides PASS/FAIL.
OUTPUT: standard table + screenshot of each failure (save under test-results/ and cite the path).
```

## Example 3 — Adversarial permissions brief

```markdown
OBJECTIVE: Verify that the RECEPTIONIST role cannot delete services (permission services.delete).
SCOPE:
- IN: attempt the forbidden action via ALL paths: direct API (Bearer), not just the UI.
- OUT: other permissions or roles.
- FORBIDDEN: modifying guards or permission seeds.
SETUP: get a receptionist token from the seed; have an existing service with a known id.
STEPS:
1. DELETE /services/:id with the receptionist token (direct Bearer, not through the UI).
2. Verify in the DB that the service STILL exists.
3. Verify the REASON for the rejection: body.message must indicate permission denied —
   if the 403 comes from CSRF/throttler/nonexistent route, the case is FAIL (the permission wasn't tested).
4. Repeat with an ADMIN token: it must be 200 and the row must disappear (positive control:
   proves step 1 failed because of the permission and not because the endpoint is broken).
PASS/FAIL: PASS only if (1) rejected for the right reason, (2) row intact, (3) positive control OK.
OUTPUT: standard table; if step 1 returns 200, it's a CRITICAL FAIL — report it first with evidence.
```

## Common mistakes when writing briefs

- Brief with no OUT/FORBIDDEN section → the subagent "helps" by fixing production code.
- "Test that module X works" with no numbered steps → the agent invents its own plan.
- Not requiring an output format → you get optimistic prose that's impossible to audit.
- Not asking for the positive control in adversarial cases → a 403 from a broken route gets reported as "permission works".
- Splitting cases from the same table/state across two parallel agents → contaminated results.
