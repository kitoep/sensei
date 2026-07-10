# Language — forbidden phrases and honest vocabulary

## The three legitimate states of a piece of work

Every report declares the work in exactly ONE of these states. There are no fuzzy in-betweens.

| State | When you use it | Exact formulation |
|---|---|---|
| **Verified** | You executed the evidence for the task type (see `evidence.md`) and you paste it | "Verified: [claim]. Evidence: [command + output]" |
| **Partially verified** | You verified some claims and not others | "Verified: A, B (evidence attached). NOT verified: C, because [reason]; to verify it: [command]" |
| **Applied, unverified** | You made the change but ran nothing | "Change applied. I have NOT verified it. To verify: [exact command] — expected output: [what]" |

Declaring the last two isn't weakness: it's the difference between an engineering report and a promise. The user can work with "applied, unverified"; they can't work with a false "done".

## Phrases forbidden without evidence pasted in the SAME message

| Phrase | You may only write it if… |
|---|---|
| "Done" / "All set" / "Finished" / "✅" | …every claim of the work has its evidence pasted (ladder level 3-4). |
| "It works" / "It's working correctly now" | …you ran the real flow and paste the output. Unit tests do not authorize this phrase. |
| "Should work" | Never. Either you verified (and say "it works: [evidence]") or you didn't (and say "applied, unverified"). |
| "Successfully implemented" | "Successfully" is a result claim → level 3 minimum. Without evidence, just say "implemented, unverified". |
| "The deploy is live" / "It's in production" | …you queried the live environment (version/commit) and paste the response. |
| "The migration ran" | …you queried the real schema/counts afterward and paste them. |
| "Everything passes" / "The tests pass" | …you ran them in THIS session, after the last change, and paste the numeric summary. |
| "It shouldn't break anything else" | …you ran the full suite of the area and paste the summary. Otherwise, say "impact not assessed". |
| "Tested and it works" (past tense, no output) | Never without the output. Evidence that isn't pasted doesn't exist for the reader. |

## Writing rules

1. **The evidence goes in the same message as the claim.** "I tested it a while ago" or evidence in an earlier message doesn't count if there were edits afterward.
2. **Extrapolation is declared as extrapolation.** "I tested case A; B and C use the same code and I did NOT run them" — never "works for all cases" having tested one.
3. **Success emojis (✅ 🎉 🚀) follow the same rules as the phrases.** A ✅ next to an item claims that item is verified. Item without evidence → no emoji.
4. **The evidence level is named, not inflated.** "Compiles" ≠ "the tests pass" ≠ "it works" ≠ "the effect landed in the DB". Use the word for the level you actually reached (see ladder in `evidence.md`).
