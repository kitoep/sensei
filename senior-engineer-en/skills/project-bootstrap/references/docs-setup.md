# Governance — leaving the project operable by agents without it degrading

A project maintained with AI agents degrades if the rules live in the memory of a conversation. You govern it with three committed pieces: **CLAUDE.md** (non-negotiable rules; or `AGENTS.md` if the tool uses that convention), **docs/** (the spec: the WHAT), and **the tracker** (plan and progress: the HOW MUCH). This phase creates them.

## 1. CLAUDE.md — the rules no agent can violate

Create it at the root BEFORE the first feature. It's not general documentation: it's the short list of rules that, if violated, break the project. Template (adapt with the decisions from intake and foundations):

```markdown
# CLAUDE.md — Project rules

[1 line: what the project is and for whom.]
[Repo structure: apps/packages and what each one does.]

## Rule #1 — [THE project invariant] (NON-NEGOTIABLE)
[The rule that's never violated, with the exact HOW. If it's multi-tenant SaaS:
data isolation between tenants, where tenant_id is derived from,
the double barrier (app + RLS), and the checklist for every new table/endpoint.]

## Rules #2–#N
[At most 3–4 more domain invariants, e.g.: "no double booking",
"the bot doesn't make things up", "idempotent jobs". Only what breaks the product.]

## Code conventions
- Layers: [controller → service → repository, or your chosen stack's].
- Validation: never trust client input; the UI is not enforcement.
- [Allowed UI stack / no introducing new libraries.]

## Established security (maintain)
[List of what's already implemented that no change may regress:
default-closed auth, encrypted secrets, rate limiting, no PII in logs...]

## Test priority
1. [Invariant #1], 2. [invariant #2], ... — and the rule: invariant tests
are ADVERSARIAL (they prove the forbidden thing FAILS), not confirmatory.
Litmus test: "if I remove the enforcement, does the test still pass?" — if
yes, the test is worthless.

## Working conventions
- No task is declared done without its tests run and documented. If a test
  fails, report it as-is.
- Docs = spec (the WHAT). Tracker = plan and progress (the HOW MUCH). If a
  task changes the WHAT, updating the doc is part of its definition of done.
- Commands: [dev, migrate, typecheck, test, build — the repo's real ones].

## MVP scope — DO NOT implement
[The exclusion list from the intake, question 3. Explicit.]
```

Rules for writing it:
- **Short and dense.** Every line must change the behavior of whoever reads it. If it's general information, it goes in `docs/`, not here.
- Only rules already decided (intake + foundations) go in. No aspirations.
- When a real bug reveals a new rule ("never X inside a transaction"), add it to CLAUDE.md as part of the fix. That way the file accumulates the project's scars and no agent repeats the mistake.

## 2. docs/ — the spec (the WHAT)

Minimum structure:

```
docs/
├── README.md          ← reading order and precedence rule
├── 00_intake.md       ← intake answers (already exists from Phase 0)
├── 01_decisions.md    ← foundations applied/dismissed with reasons (Phase 1)
├── 02_product.md      ← what the product does: features, flows, business rules
├── 03_db_schema.md    ← data model and its invariants
└── 04_stack.md        ← stack, hosting, deploy
```

Rules:
- `docs/README.md` declares the **reading order** and the **precedence on contradiction** (typically: product > db schema > stack). Without declared precedence, two contradicting docs paralyze any agent.
- Docs describe the DESIRED state (spec), never the progress. Forbidden to write "X is already done" in a doc — that belongs to the tracker.
- If a task changes the WHAT (product, schema, permissions, stack), updating the corresponding doc is part of its definition of done, not a "later."

## 3. Tracker — the plan and the progress (the HOW MUCH)

It can be a file (`TRACKER.md`), a board, or an artifact — but it's ONE, and it rules the order of work. Minimum content per milestone: tasks with status, a measurable done criterion, and **test evidence** (what was run and what came out).

Rules:
- **Golden rule: no task is declared done without its tests run and documented in the tracker.** "I finished the code" is not a status; "tests green: [evidence]" is.
- On a contradiction about plan or order, the tracker wins (docs win on the WHAT).
- When designing a new milestone, cross-check its scope against the tasks already listed in the tracker BEFORE writing the spec: every prior task is either executed or explicitly re-planned with the user — never dropped silently.

## Phase output

`CLAUDE.md`, `docs/` with a README + the minimum docs, and the tracker with the Phase 2 milestones — all committed. The project is now ready for any agent, on any model, to work on it without degrading it.
