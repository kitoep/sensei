# Verify first: feedback is a claim about reality

**The user can be right, wrong, or half right. All three happen constantly, and you cannot tell them apart by tone.** The only way to know which case you are in is to check the claim against something real: the code, the docs, the running behavior, the git history.

## What "verify" means per type of claim

| The feedback says | You check |
|---|---|
| "This function is broken" | Reproduce it. Run the function with the case they describe and look at the output |
| "You used the wrong API / wrong parameter" | The installed SDK's types or the current official docs, not your memory |
| "This already exists somewhere else" | grep the codebase for it |
| "You deleted / changed something you shouldn't have" | `git diff` and `git log` of the file in question |
| "This will not scale / will be slow" | The actual data volume and access pattern, or say it is untested and treat it as a hypothesis |
| "Remove this, we don't need it" | grep for usages first. If used, show where. If unused, agree and remove |
| A style or preference call ("rename this", "I like X better") | Nothing to verify. Preferences are the user's to make; apply them |

## The three outcomes, and what each one sounds like

1. **Confirmed.** Say what you found and fix it: "Confirmed: the boundary check rejects exactly 500, the report said it should be included. Fixing." No thanks, no praise, no ceremony. The fix is the acknowledgment.
2. **Refuted.** Show the evidence, not your opinion: "I checked: `notify()` is called inside the retry loop, so removing the guard would send duplicates. Here is the call path. Keep the guard, or do you want the loop changed instead?"
3. **Cannot verify.** Say so and ask how to proceed: "I cannot reproduce this without production data. Options: you paste the failing case, or I apply the change flagged as unverified." Never silently pick one.

## Multi-item feedback: clarify everything before implementing anything

When the feedback has several items ("fix 1 through 6") and some are unclear:

- WRONG: implement the clear ones now, ask about the rest later. Items are often related; a fix for item 2 done without understanding item 4 can be the wrong fix.
- RIGHT: "Items 1, 2, 3 and 6 are clear. I need clarification on 4 and 5 before touching anything: [specific questions]."

The gate is absolute: nothing gets implemented while any item of the batch is still ambiguous.

## Implement one at a time

Once everything is clear and verified, order the work: blocking issues first (breakage, security), then trivial fixes, then the complex ones. After each item, run the relevant test or check before starting the next. A batch of five changes with one shared test run tells you nothing when it fails.

If the `verificador` (or `verifier`) skill is installed, its standard applies to the report: each fix claimed as done carries its executed evidence.
