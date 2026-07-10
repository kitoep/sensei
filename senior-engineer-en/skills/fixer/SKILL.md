---
name: fixer
description: Use when debugging any bug, error, test failure, crash, or flaky behavior; when the user reports "it's broken", "still failing", or that something previously claimed as fixed is still broken; before proposing any fix; or when about to declare a bug resolved.
---

# Fixer — systematic debugging with hard gates

## Principle

A bug is not fixed when the code "looks right". It's fixed when the original reproduction passes, a regression test fails without the fix, and the defective pattern was searched across the whole repo. Everything else is an opinion.

**Violating the letter of these rules is violating their spirit.** There are no bugs "too simple" for the process.

## Phase flow (with hard gates)

Each phase has a GATE. You do not advance to the next phase without clearing it. If you skipped it, go back.

| Phase | What you do | GATE to advance | Reference |
|---|---|---|---|
| 1. Reproduce | Reproduce the bug deterministically | You have the exact command + the error output pasted | `references/diagnosis.md` |
| 2. Diagnose | Single falsifiable hypothesis + minimal experiment | Evidence that confirms YOUR hypothesis and rules out rivals | `references/diagnosis.md` |
| 3. Root cause | From symptom to the real cause; classify one-off vs class | You can explain WHY it happens, not just WHERE it blows up | `references/root-cause.md` |
| 4. Fix | Fix the cause, not the symptom | The fix attacks the root cause identified in phase 3 | `references/root-cause.md` |
| 5. Verify | Full "fixed" checklist | The 5 checklist points with evidence pasted | `references/verification.md` |

**Load the reference for the phase you're in BEFORE acting in it.**

Special cases:
- User reports "still failing" after a fix of yours → go straight to `references/verification.md`, section "Still-failing protocol".
- 3+ dead hypotheses or >30 min with no progress → load `references/stuck.md`.

## Red flags — STOP if you catch yourself thinking this

- "It's probably X, let me change it and see" → no reproduction, no falsifiable hypothesis. Phase 1.
- "It's fixed now" (with no command output pasted) → it's not fixed. Phase 5.
- "Should work now" → "should" = you didn't verify. Phase 5.
- "I'll just add a try/catch here" → you're hiding the symptom, not fixing the cause.
- "Let me tweak the test so it passes" → you're erasing the evidence of the bug.
- "It's an edge case, not worth reproducing" → if it doesn't reproduce, you can't know you fixed it.
- "Works on my machine" → the user's report is correct until proven otherwise.

## Known rationalizations

| Excuse | Reality |
|---|---|
| "The fix is obvious, I don't need to reproduce" | "Obvious" fixes that don't attack the real cause create the "still failing" report. Reproducing takes minutes; the false fix costs hours. |
| "No time for a regression test" | Without a regression test the bug comes back in the next refactor and you pay for all of it again. |
| "The error is gone, done" | Gone because you fixed it, or because you changed the conditions? Without running the original reproduction, you don't know. |
| "I changed 3 things and now it works" | You don't know which one fixed it or what the other two broke. Revert and apply one at a time. |
| "It's flaky, must've been the environment" | Flaky = a concurrency/timing/state bug until proven otherwise. |
