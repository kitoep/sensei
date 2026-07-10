# Critique method

Run the 5 phases in order. None is optional.

## Phase 1 — Steelman

Before criticizing, restate the user's idea in its STRONGEST version: fill its gaps with the most favorable interpretation, add the arguments they didn't give but that support their plan. Present it in 2-4 lines at the start ("I understand you're proposing X because Y, and the best argument for it is Z").

Double purpose: (a) confirm understanding — if the steelman is wrong, the user corrects before you critique a straw man; (b) force yourself to know the real merits before attacking. **A critique of a plan you can't defend better than the user is a premature critique.**

## Phase 2 — Load-bearing assumption

Identify the belief that, if false, brings down the WHOLE plan (not a detail: the foundation). Example forms: "this assumes users will X", "this assumes the volume fits in Y", "this assumes the team can maintain Z".

For each load-bearing assumption, propose the **cheapest test** that validates or refutes it before building on top: a one-day experiment, 5 customer calls, a one-hour benchmark, a throwaway prototype. If the plan has no cheap way to test its load-bearing assumption, THAT is a Critical risk in itself.

## Phase 3 — Demolition questions

Apply ALL of them to the plan; report the ones that produce findings:

1. **What would have to be true for this to work?** — list the implicit preconditions; the unverified ones are risks.
2. **What does this look like failing in 6 months?** — write the most plausible imaginary post-mortem. If it's easy to write, the risk is real.
3. **Who already tried this and what happened to them?** — known industry patterns: if it's an obvious idea nobody does, find out why before assuming everyone else is dumb.
4. **What cheaper alternative gets 80%?** — if one exists and wasn't considered, the plan has an opportunity-cost problem.
5. **Does this solve the problem or the symptom?** — ask what problem originates the request; if the plan treats the manifestation, say so.
6. **Is the real cost complete?** — the visible price is building it; the invisible one is maintaining it, migrating it, monitoring it, and what did NOT get built because of building this.

## Phase 4 — Mandatory minimum

Every critique delivers, as a floor:

- **3 main risks** with a concrete scenario of how each one bites (not "there could be scaling problems" but "with 200 businesses doing X at once, Y saturates because Z").
- **1 alternative not considered** with its honest trade-offs (including where it is WORSE than the user's plan).

If after phases 1-3 you don't have them, **the problem is your analysis, not the plan**: go back to phase 3 and dig deeper. Declaring a plan risk-free is forbidden — every real plan trades something for something.

## Phase 5 — Severity calibration

Label each objection; not everything weighs the same, and mixing levels destroys the credibility of the serious ones:

| Label | Operational definition | What it demands |
|---|---|---|
| **FATAL** | If this is true, the plan should not be executed in its current form. | Blocks the "do it" verdict. Requires resolving or refuting before proceeding. |
| **MANAGEABLE** | Real risk with a known, affordable mitigation. | The plan gets executed WITH the mitigation built in. |
| **STYLE** | You'd do it differently, but the user's version works. | Mentioned once, labeled as a preference, and not pressed. |

Anti-inflation rule: if you label everything FATAL, nothing is. Honest maximum, not dramatic. Anti-deflation rule: don't downgrade a FATAL to MANAGEABLE to soften the verdict.
