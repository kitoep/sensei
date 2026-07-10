# Root cause (Phases 3–4)

## The point where it blows up is NOT the cause

The stack trace tells you where it BLEW UP, not why. A `NullPointerException` on line 40 is almost never fixed on line 40: it's fixed where the null became possible. Fixing at the point of explosion (adding an `if (x != null)`) is treating the symptom — the null keeps traveling and will blow up somewhere else.

**Rule:** trace the bad value/state BACKWARD to the point where it originated, and fix there.

## 5 whys applied to code — full example

Symptom: a customer received two reminders for the same appointment.

1. **Why did two messages arrive?** Because there are two reminder jobs for the same appointment in the queue.
2. **Why are there two jobs?** Because rescheduling the appointment enqueues a new job without canceling the previous one.
3. **Why isn't the previous one canceled?** Because the `jobId` includes the appointment's timestamp, so the old and new jobs have different IDs and the queue's dedupe doesn't see them as duplicates.
4. **Why does the jobId include the timestamp?** Because it was designed for "one job per time slot", not "one job per appointment".
5. **Why was it designed that way?** Because there was no written idempotency rule: each feature invents its own jobId scheme.

- Symptom fix (bad): manually delete the duplicate jobs.
- Cause fix (good, level 3–4): deterministic `jobId` per appointment (`reminder:<appointmentId>`), and rescheduling replaces the job.
- Class fix (better, level 5): documented convention + a single helper to generate jobIds, and a mandatory idempotency test for every new job.

Stop at the level where the fix is actionable and proportional; it's almost never level 1.

## The symmetric-pattern rule — bugs live in families

When you confirm the root cause, it is MANDATORY to search for the SAME defective pattern across the rest of the repo before considering the fix closed:

1. Describe the pattern in one greppable sentence. E.g.: "unique-error catch inside a transaction", "third-party token stored unencrypted", "date built with local TZ".
2. Search it mechanically: `grep -rn "<pattern>" apps/ packages/ src/` — across ALL modules/services/workers in the monorepo, not just where it blew up. Shared code (`packages/`, `lib/`, `utils/`) counts double: a bug there is a bug in all its consumers.
3. Each occurrence found: either you fix it in the same fix, or you explicitly document why that instance isn't affected. There's no third option ("I'll look at it later" = the same bug reported in 2 weeks in the other module).
4. Report the search result as part of the fix: "pattern searched in X, Y, Z; found and fixed in Y; Z doesn't apply because...".

## Classify: one-off bug or class of bug?

Mandatory question before writing the fix: **does the architecture ALLOW this bug to be written again tomorrow?**

| Signal | Classification | What the fix requires |
|---|---|---|
| Typo, inverted condition, edge case forgotten in ONE place | One-off | Fix + regression test |
| The same error can be written in any new module (forget tenant_id, forget to encrypt, forget idempotency) | Class | Fix + make the class IMPOSSIBLE or detectable |
| Already appeared 2+ times in the repo history | Class (confirmed) | Same as above, no excuses |

Making the class impossible, in order of preference:
1. **Database constraint** (unique, check, FK, RLS): the guarantee lives where it can't be bypassed.
2. **The type system**: types that can't represent the invalid state (e.g. an `EncryptedToken` type distinct from `string`).
3. **Lint rule / CI check**: detects the forbidden pattern on every PR.
4. **A single mandatory helper**: one correct function everyone uses (and lint that forbids the manual path).
5. **Documentation/checklist**: the weakest; use it only if 1–4 are impossible, and say so.

If you choose not to close the class (due to cost/scope), declare it explicitly to the user as debt: "this fix corrects the case; the class stays open because X".

## Phase 4 — The fix

- The fix attacks the identified cause, at the point of origin, not at the point of explosion.
- One fix per cause. If you spotted another bug along the way, report it or fix it in a separate change.
- Zero opportunistic refactors mixed into the fix: the fix's diff should read as "this and only this fixes the bug".
- If the fix is a conscious workaround (doesn't attack the cause), label it as such to the user and record what's left to do.
