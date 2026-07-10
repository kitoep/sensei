# Adversarial testing — anti "fake green"

Rules born from real bugs where a test passed but tested the wrong thing. A security test that can't fail when the protection disappears is worse than no test: it gives false confidence.

## The litmus test (apply it to EVERY security test)

> **"If I delete the enforcement logic from the backend, would this test stay green?"**

If the answer is yes, the test is useless. Delete it or fix it. A test for a security invariant MUST go red when the defense is removed.

## Rule 1 — Adversarial, not confirmatory

- **Confirmatory (bad):** verifies the happy path works, or that a flag/signal shows up. Useless example: "the response has `canEdit: false`."
- **Adversarial (good):** attempts the FORBIDDEN action with a client that ignores the UI, and requires that it FAILS and that the real effect does NOT happen.
- A flag for the frontend is NOT enforcement. The test must ignore the flag and attempt the action anyway.

## Rule 2 — Verify the real effect, not the HTTP status

- After a denied mutation, receiving 403/404 isn't enough. **Query the real state:** the row was NOT created/modified/deleted in the DB; the message was NOT sent; the slot stayed free.
- A 200 can hide that nothing happened, and a 403 can arrive without the permission ever being evaluated. Check the effect.

## Rule 3 — The 403/404/409 must arrive for the right reason

A rejection can come for the wrong reason and produce a fake green:
- **CSRF:** if you test permissions with a request that also fails CSRF, the 403 is from CSRF, not permissions. Use a Bearer token (which is exempt from CSRF) to isolate the control you want to test.
- **Throttler:** a 429 in disguise, or a 403 after rate limiting.
- **Format validation:** a `ParseUUIDPipe`/schema that rejects the malformed ID before it reaches the permission check. Use a valid, well-formed ID.
- **Nonexistent route:** a 404 because the route doesn't exist, not because the resource belongs to another tenant.
- Verify the reason: inspect the error body, or construct the case so the ONLY possible rejection reason is the one you're testing.

## Rule 4 — Against real dependencies, not mocks

- Security tests run against real Postgres/Redis (not mocks), because what you're testing is the real behavior of RLS, uniqueness constraints, transactions, and queues. A mock has no RLS and no constraints.
- If you mock the DB, you're not testing isolation; you're testing your mock.

## Rule 5 — Every critical invariant has its behavior test

Before closing a feature, map it to its invariants and make sure the adversarial test for each one it touches exists:

1. **Multi-tenant isolation:** tenant A doesn't see/affect B's data (404, and null effect in the DB). See saas-multitenant.md.
2. **No double-booking / uniqueness:** concurrency tested (N simultaneous requests to the same resource → exactly 1 wins, verified in the DB).
3. **Job idempotency:** retrying the same job doesn't duplicate the effect (send/charge/reservation).
4. **Auth default-closed:** a route with no token is rejected; a public exception is intentional.
5. **Server-side enforcement:** a client that ignores the UI and sends the forbidden request directly, fails.
6. **Human mode (if there's a bot):** in HUMAN mode the bot emits no message (verify no message was created/sent).

## Rule 6 — Concurrency is actually tested

For uniqueness invariants (no double-booking, a single winner):
- Fire N concurrent requests at the same slot/resource.
- Verify in the DB that exactly 1 succeeded and N-1 failed.
- This validates that the guarantee lives in a DB constraint, not in app logic that loses the race. If the test passes with 1 request but you never run it with N, you didn't test concurrency.

## Rule 7 — Symmetric fixes

When you fix a defective pattern (dedupe with catch-in-tx, unencrypted token, query without tenant filter, secret in a log), the SAME pattern usually lives in other apps of the monorepo (api, worker, shared) and in other modules.
- Grep the pattern across the whole repo.
- Fix it everywhere.
- Add/extend the test to cover each location.
- Fixing only one leaves the hole open and gives a false sense of being resolved.

## Rule 8 — No `as any` that hides types in security logic

A cast that silences the compiler can hide the bug (a value that isn't what you think, a field that doesn't exist). If the type is in the way, that's a signal something is wrong, not that the type is unnecessary. `as any`/`# type: ignore` forbidden in enforcement code.

## Checklist before declaring "tested and secure"

1. [ ] Every security test passes the litmus test (red if enforcement is removed).
2. [ ] The tests verify the effect in the DB/state, not just the status.
3. [ ] The rejection arrives for the right reason (isolated from CSRF/throttler/validation/route).
4. [ ] They run against real dependencies.
5. [ ] An adversarial test exists for every critical invariant the feature touches.
6. [ ] Concurrency is tested with N requests where applicable.
7. [ ] Fixes were applied symmetrically and there's a test at each location.
