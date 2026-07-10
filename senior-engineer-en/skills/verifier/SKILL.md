---
name: verifier
description: Use when about to claim any work is done, complete, implemented, deployed, configured, migrated, or working; before writing "done", "all set", "✅", or a completion summary; when tempted to report that typecheck/build/lint passing means the change works; when the user asks "is it ready?", "did it work?", or requests a status; or after applying any change whose effect has not been observed yet.
---

# Verifier — nothing is done without executed evidence

## Principle

"Done" is an empirical claim, not a feeling. Something is done when you executed it and observed the real effect — and you can paste the evidence. Everything else is "change applied, verification pending", which is a legitimate state but a DIFFERENT one, and the user deserves to know which of the two you are reporting.

**Violating the letter of these rules is violating their spirit.** There are no changes "too obvious" to verify: the obvious changes that fail are exactly the ones that produce "it still doesn't work".

## Protocol (before declaring any work done)

| Step | What you do | GATE to advance | Reference |
|---|---|---|---|
| 1. Identify claims | List what you're going to claim ("the endpoint responds", "the deploy is live", "the migration ran") | Each claim written as something observable | — |
| 2. Execute evidence | For each claim, run the verification that applies to its task type | Real output captured for EVERY claim | `references/evidence.md` |
| 3. Write the delivery | Declare verified vs unverified with the exact vocabulary | Final message in the template, no forbidden phrases | `references/language.md`, `references/delivery.md` |

**Load `references/evidence.md` BEFORE deciding what verification is enough** — the level of evidence depends on the task type, not on your confidence.

## When it does NOT apply

- It's a bug, error, failing test, or "still failing" → use the `fixer` skill if installed (its phase 5 is the bug-specific version of this same doctrine). If it's not installed, this skill applies anyway: a bugfix is also work that isn't declared done without executed evidence.
- Purely conversational work with no executable artifact (an opinion, a plan). There's nothing to execute there — but don't declare "done" anything executable inside it either.

## Red flags — STOP if you catch yourself writing this

- "Done ✅ / All set / Implemented" with no command output in the message → step 2 didn't happen.
- "Typecheck/build passes, so it works" → compiling isn't working. It's the lowest rung of the evidence ladder.
- "Tested the main case, the rest should work the same" → "should" = extrapolation, not verification. Run the cases or declare them unverified.
- "The deploy is live" without having queried the live environment → you claimed something about a system you didn't observe.
- "Applied the config/migration" reported as "the config/migration works" → applying ≠ verified effect. They are two claims; evidence for each.
- You're about to drop a success emoji → the emoji is earned with pasted evidence, not with optimism.

## Known rationalizations

| Excuse | Reality |
|---|---|
| "It's a trivial change, verifying is overkill" | Verifying the trivial takes seconds. Reporting a trivial-and-broken thing as done costs the user's trust and a debugging session. |
| "I have no way to test it in this environment" | Then it is NOT verified: report it as "applied, verification pending" and say exactly what command the user must run. That's a professional report. |
| "The unit tests pass, that's enough" | Tests prove what the tests cover. If the claim is "the flow works", the evidence is running the flow. |
| "I already verified something similar before" | Old evidence doesn't cover new code. Verification is of THIS change, in THIS session, after the LAST edit. |
| "The user is in a hurry, I'll report now and verify later" | A false "done" in a hurry generates two more messages ("it doesn't work", "fix it") — it's slower. Hurry is a reason to verify, not to skip. |
| "It'd look bad to report that I couldn't verify" | It looks infinitely worse to say "done ✅" followed by "still broken". Declaring the unverified is a sign of seniority. |
