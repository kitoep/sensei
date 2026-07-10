# Output format — the recommendation template

The final recommendation is delivered EXACTLY with these 7 sections, in this order. None is optional. If a section would be empty, that's a signal the intake is incomplete — go back and ask.

**Traceability rule (applied before delivering):** walk through each choice in section 3; if you can't cite the intake condition that justifies it, it's fashion — cut it or move it to section 5 (not yet).

---

```markdown
## 1. Context understood
[3-6 lines: real users at 6 months, budget, who operates it, consequence
of a 1h outage, data sensitivity, stack the team knows, what's most
uncertain about the business. Mark with "(assumption)" any data you assumed
due to a missing answer — the user corrects here, not after building.]

## 2. Dominant force
[ONE: cost | availability | development speed | scale.
1-2 lines on why that one and what gets subordinated to it.]

## 3. Recommendation by layers
| Layer | Choice | Why (user's condition) |
|---|---|---|
| Hosting | ... | ... |
| Database | ... | ... |
| Backend | ... | ... |
| Frontend | ... | ... |
| Jobs/queues | ... | (or "not applicable: <condition>") |
| Storage | ... | ... |
| Observability | ... | ... |
[Column 3 cites the condition, not generics: "because you operate solo with
no devops" ✔ — "because it's scalable" ✘]

## 4. Estimated monthly cost
[Breakdown per piece + total as a range: "$X–Y/month". If it exceeds the
intake budget, say so and offer what to cut.]

## 5. Not yet — and the signal to add it
[Table: piece cut today → concrete metric/threshold that triggers adding it
→ which seam is already prepared for that day.
E.g.: "Redis cache → the same query dominates p95 >500ms → repos already
encapsulate data access".]

## 6. Risks of this recommendation
[2-4 REAL risks of what's recommended, with their mitigation or explicit
acceptance. E.g.: "single-instance: deploy with ~10s of blip — accepted for
budget". A recommendation with no declared risks is incomplete.]

## 7. Main alternative ruled out
[The second serious option you considered and WHY it lost against the
conditions (not against your taste). 2-4 lines. Give the user the door:
"if [condition] changes to [X], this alternative wins".]
```

---

## Final checks before sending

1. Does each row of section 3 cite an intake condition? (traceability)
2. Does the section 4 total fit the declared budget?
3. Does section 5 have at least 2 pieces? (if everything went in today, you over-provisioned)
4. Are the assumptions marked "(assumption)" in section 1?
5. Is the biggest irreversible decision (DB / multi-tenant model / language) explained in more depth than the reversible ones?
6. Did you tell the user something they didn't want to hear? (if the recommendation matches 100% what they already thought, check whether you're pleasing instead of analyzing)
