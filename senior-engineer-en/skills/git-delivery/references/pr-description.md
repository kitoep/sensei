# PR description — auditable, not a diff narration

## Principle

The reviewer ALREADY SEES the diff. The description exists to give them what the diff doesn't contain: the problem context, the decisions made, the evidence it works, and the risks. **Restating the diff ("file X was modified to add function Y") is anti-pattern #1: it takes up space, informs nothing, and buries what matters.**

Test for every sentence in your description: can the reviewer deduce it by reading the diff? If yes, delete it.

## Required template (the 5 sections, in this order)

```markdown
## Context / Problem

<What was broken or missing and who it affects. The state of the world
BEFORE the PR. Reference to the ticket/issue/report. If it's a bug: the
observable symptom and how to reproduce it. 2-5 lines.>

## What changes and why

<The decision, not the diff. What approach you took and WHY that one and
not another. Rejected alternatives and their reason, if you weighed any.
If there's a change in the diff that would surprise the reviewer ("what's
this doing here?"), explain it here before they ask. 3-8 lines.>

## How it was verified

<EXECUTED EVIDENCE, not intentions. For each claim, the real output:
- Suite: command + pasted summary (`142 passed, 0 failed`).
- New test: what it captures, and that you saw it FAIL without the change.
- Manual verification: exact steps + observed result (not "tested and it
  works": which request/action, which response/state resulted).
- For UI: before/after screenshot.
Anything NOT verified is stated: "not tested on <env/case> because X".>

## Risks and what it does NOT cover

<What could break and where they'd look first if something fails. Debt
this PR leaves on purpose. Out-of-scope cases. Required migrations or
deploy steps. "Risks: none" is forbidden — a change with no identifiable
risk didn't warrant a PR; identify at least the area of greatest exposure.>

## Notes for the reviewer

<Where to start reading the diff. Which file is the heart of the change
and which are mechanical/generated. Where you want a second opinion.
If stacked: on which PR and which range to review.>
```

## Hard rules

- **The 5 sections are mandatory.** Without evidence in "How it was verified" the PR isn't opened: finish verification first. "CI verifies it" doesn't count — CI verifies the suite passes, not that the change does what the PR promises.
- "How it was verified" must contain REAL pasted output, from commands run in this session after the last commit. Pasting output from memory or from a run before the last change is forbidden.
- The description is written by reading `git diff main...HEAD` (the full PR range), not from your memory of what you did.
- Target length: the reviewer reads it in ≤2 minutes. If it exceeds ~40 lines, the PR probably exceeds its scope (see `branches-prs.md`), not the description its brevity.

## Calibrated example

❌ **Diff narration (forbidden):**

> This PR modifies `discount.py` to change the `calculate_discount` function so it uses `price_with_tax` instead of `base_price`. It also updates `test_discount.py` by adding the `test_discount_with_tax` test and modifies `models.py` to expose the property.

All of that is in the diff. Zero new information.

✅ **Auditable:**

> **Context:** Coupon orders were overcharging: the discount was applied to the pre-tax price (report #482, ~3% of June's orders).
>
> **What changes and why:** The discount is now computed on the final tax-inclusive price, which is what the customer sees and what Legal confirmed as correct. I rejected recomputing tax post-discount: it shifts historical totals and needs a migration; this approach only affects new orders.
>
> **How it was verified:** `pytest tests/orders/ -q` → `87 passed, 0 failed`. The new test `test_discount_with_tax` fails without the fix (`AssertionError: 116.0 != 100.0` — output pasted below). I reproduced the order from report #482 in staging: total goes from $116.00 to $100.00.
>
> **Risks:** any financial report that assumes the old calculation will show a discontinuity from the deploy onward. Doesn't cover historical orders (on purpose, see above).
>
> **Notes for the reviewer:** the heart is `discount.py:42-58`; `models.py` only exposes an already-computed property.
