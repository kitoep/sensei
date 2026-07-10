# Delivering results — template and rules for the final message

## Core rule: 1 claim → 1 piece of evidence

Every behavior or effect claim in your final message maps to exactly one piece of executed evidence in the same message. Claim without evidence → either you get the evidence, or you move the claim to "Not verified". There is no third option.

## Final message template

```markdown
## What changed
[1-3 lines: files/systems touched and the expected effect. No process narrative.]

## Verified (evidence executed in this session, after the last change)
- [Claim 1] — level [3: flow executed / 4: state queried]
  ```
  $ [exact command]
  [relevant lines of the real output]
  ```
- [Claim 2] — ...

## Not verified
- [Claim X] — why not: [no access to prod / missing credential / environment unavailable]
  To verify: `[exact command]` — expected output: [what should appear]

## Residual risks (if any)
- [What could fail even with what's verified, e.g. "only tested with datasets < 1k rows"]
```

The "Not verified" and "Residual risks" sections are omitted ONLY if they are genuinely empty — not because they look bad. A delivery with "Not verified" populated is more trustworthy than one without the section.

## Rules about pasted evidence

1. **Real output, not reconstructed.** Paste what the terminal printed. If you write "it responded 200" from memory, that's a claim, not evidence.
2. **Truncating is fine; the lines that prove it are not.** From a 200-line output, paste the 5-10 that demonstrate the claim (the test summary, the DB row, the status + relevant body) and mark the cut with `[...]`.
3. **Include the command alongside the output.** Output without a command is neither auditable nor reproducible by the user.
4. **Logical timestamp:** the evidence is valid only if it was executed AFTER the last edit. If you edited something after verifying, the affected verification goes back to "Not verified" until you re-run it.
5. **For UI:** the evidence is the screenshot or Playwright output, with the state it shows named ("error state with visible message").

## Delivery anti-patterns

| Anti-pattern | Correction |
|---|---|
| List of ✅ checkmarks per task, zero output | Each ✅ becomes a "Verified" entry with evidence, or loses the checkmark |
| 30-line "summary of everything I did", evidence in 0 | The delivery is organized by verifiable claims, not by chronology of edits |
| Level 1-2 evidence presented under "it works" | Rename it to what it is: "compiles", "the unit tests pass". See ladder in `evidence.md` |
| "Verified manually" without saying what command or what output | "Manually" is not a method. Command + output, or it goes to "Not verified" |
| Promising future verification ("I'll test it later") and closing as done | The state is "applied, unverified" until the verification happens |

## Relationship with `fixer`

If the work was a bugfix and the `fixer` skill is installed, the evidence standard is its 5-point checklist (`fixer/references/verification.md`: original reproduction, regression test seen failing, full suite, effect on state, symmetric pattern) — it's a superset of this template. If `fixer` is not installed, use this template requiring at minimum the original reproduction executed and passing, plus the area's suite. This template applies to everything else: features, refactors, config, deploys, migrations, scripts, UI.
