# Verification (Phase 5) — the "fixed" standard

**A fix without this full checklist is NOT declared fixed.** It's declared "change applied, verification pending" — which is a different thing, and the user deserves to know the difference.

## Mandatory checklist (the 5 points, with evidence)

### (a) The original reproduction now passes
Run EXACTLY the Phase 1 reproduction (same command, same data). Paste the real output in your report. "I ran the repro and it now responds 200 with the expected body: [output]". If you changed the repro to make it pass, you violated prohibition #3 in diagnosis.

### (b) A regression test that fails without the fix
1. Write the test that captures the bug.
2. **Test it against the bug:** temporarily revert the fix (`git stash` the fix, or comment out the lines), run the test, confirm it FAILS with the original error. Paste that output.
3. Restore the fix, run the test, confirm it PASSES. Paste that output.

A regression test you never saw fail proves nothing: it might be testing something else, or nothing (fake green).

### (c) The full suite passes
Not just your new test: the whole suite of the affected area (ideally the entire project). Fixes break neighboring things. Paste the summary (`X passed, 0 failed`). If something fails, your fix isn't ready — no matter that it "seems unrelated": prove it or fix it.

### (d) The real effect verified in the state
Exit code 0 and HTTP 200 are not the effect. Verify the real state the bug was corrupting:
- Data bug → query the row in the DB and paste the result.
- Duplicate-send bug → count the messages/jobs actually created (1, not 2).
- File bug → show the content of the generated file.
- Concurrency bug → run the scenario N times / N parallel requests and paste the count (exactly 1 won, 20/20 clean runs).

### (e) Symmetric pattern searched
Result of the pattern search (see root-cause.md): search command run + occurrences found + what was done with each. "I searched `<pattern>` across the whole repo: 3 hits; fixed 2, the third doesn't apply because X".

## Phrases forbidden without attached evidence

| Phrase | You can only say it if… |
|---|---|
| "It's fixed now" | …all 5 checklist points have pasted output. |
| "Should work" | Never. Either you verified and "it works", or you didn't and you say so: "I applied the change, X still needs verifying". |
| "The problem was X" | …you have the experiment that confirms X and rules out the rivals (Phase 2). If not, say "my hypothesis is X". |
| "Tests pass" | …you ran them in THIS session, after the last change, and you paste the summary. |
| "Shouldn't affect anything else" | …you ran the full suite (point c). |

Honesty about what's NOT verified is not weakness: "fixed and verified (a)-(d); (e) pending because the repo is huge, pattern described to search for it" is a professional report. "Done ✅" with no evidence is the report that produces the "still failing" message.

## "Still failing" protocol

When the user reports that something you declared fixed is still broken:

1. **The user's report is correct by default.** Forbidden to reply "works on my machine", "I already fixed it, try again", or to re-assert the fix without new evidence.
2. **Go back to Phase 1 with THE USER'S CASE**, not your previous repro. If your repro passes and the user's case fails, it means your repro wasn't capturing the real bug (or there's a second bug). Ask for/extract the exact details: what they did, with what data, what they saw, in what environment/commit.
3. **Audit your previous verification:** which of the 5 checklist points did you skip or do weakly? That gap is your first hypothesis. Did you verify the status but not the state (d)? Did you never see the regression test fail (b)?
4. Consider second-order hypotheses: fix correct but not deployed (stale build, cache, wrong branch), fix correct for ONE cause of a symptom with TWO causes, fix that introduced a new regression with the same symptom.
5. When re-declaring fixed, the checklist runs in FULL again, now including the user's exact case as the reproduction.
