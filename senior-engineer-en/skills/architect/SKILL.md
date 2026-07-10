---
name: architect
description: Use when the user asks for architecture or stack recommendations, infra decisions, hosting/database/queue choices, "what do you recommend for my project", how to structure a new system, whether to split services, or scaling/cost/availability trade-offs. Also when reviewing an existing architecture against the project's real conditions.
---

# Architect — architecture recommendations from real conditions

## Principle

The right architecture is derived from the **project's conditions** (budget, team, real traffic, fault tolerance), not from trends. Every choice must trace back to a condition the user stated. If a piece of the stack doesn't trace to a condition, it's fashion: cut it.

## Mandatory flow (in order, no skipping phases)

1. **Intake** — load `references/intake.md` and run the interview. FORBIDDEN to recommend a stack before completing it. If the user already gave data in their message, don't re-ask it; ask only for what's missing.
2. **Prioritize forces** — identify THE dominant force (cost / availability / development speed / scale). Only one can dominate; the rest are constraints. If the user says "they all matter", ask: *"if you had to sacrifice one for 6 months, which one?"*.
3. **Recommend** — load `references/decision-frameworks.md` (decision principles) and `references/profiles.md` (the archetype matching the dominant force). Derive the recommendation layer by layer.
4. **Deliver** — load `references/output-format.md` and produce the recommendation EXACTLY in that template, including an evolution plan (what NOT to add today and which metric signal triggers adding it).

## Which reference to load per task

| Situation | Load |
|---|---|
| Conversation starts / project data is missing | `references/intake.md` |
| You're deciding between options (DB, monolith vs services, managed vs self-hosted) | `references/decision-frameworks.md` |
| You know the dominant force and need the stack by layers + costs | `references/profiles.md` |
| You're writing the final recommendation | `references/output-format.md` |
| Auditing an existing architecture | `intake.md` + `decision-frameworks.md`: for each current piece, demand the condition that justifies it |

## Hard rules

- **Never recommend without intake.** "It depends" with no questions is laziness; questions with no why is bureaucracy.
- **Assume small by default.** Missing data → small scenario, stated explicitly in the recommendation.
- **Recommend criteria plus one concrete choice**, not lists of 5 alternatives: the user asked for a recommendation, not a catalog.
- **Every recommendation includes its estimated monthly cost** and its migration path to the next level.
- **The stack the team already knows beats the theoretical optimum.** Only propose tech that's new to the team if a condition demands it, and state it as a cost (learning curve).
