---
name: project-bootstrap
description: Use when starting a new project or repository from scratch, setting up a greenfield codebase, scaffolding a new app or service, or when the user says "how do I start", "start the project", "spin up a new repo", "new project", or wants to begin building something that doesn't exist yet. Also use before writing the first line of code in an empty or near-empty repo.
---

# Project Bootstrap — starting projects that don't collapse later

## Relay with `shaping`

If what arrives is still fuzzy (open decisions, more than one possible reading), the `shaping` skill goes first if installed: this skill works on ideas already defined. And the other way around: if the idea arrives shaped and approved, do not repeat that intake; build on what was already answered.

## Principle

Almost every project that dies at the 3–6 month mark was already dead in week one: it skipped the questions, the foundations, or the walking skeleton. This skill exists so you do NOT write code on day 1. The phase order is mandatory and each phase has a gate: you don't advance without clearing it.

**Skipping the phase order "because the project is simple" is the #1 cause of rework.** A simple project clears the phases in an hour; a complex one, in a day. Neither skips them.

## Phases (in order, with gates)

| Phase | What you do | Reference to load | Gate to advance |
|---|---|---|---|
| 0. Intake | Ask the user BEFORE deciding anything | `references/intake.md` | Answers recorded in `docs/00_intake.md` and confirmed by the user |
| 1. Foundations | Decisions that are cheap today, brutally expensive later | `references/foundations.md` | Foundations checklist applied or dismissed item by item with a written reason |
| 2. Walking skeleton | End-to-end path deployed before any feature | `references/walking-skeleton.md` | The skeleton runs in a deployed environment and CI is green |
| 3. Governance | CLAUDE.md + docs so agents don't degrade the project | `references/docs-setup.md` | CLAUDE.md and docs structure committed |

## Hard rules

1. **Zero code before Phase 0 is done.** No scaffold, no `npm init`, no "let me get the repo started." If the user asks for code up front, respond: "First I need 5–10 answers that change the architecture; without them I'll build the wrong thing" and start the intake.
2. **Intake questions are asked one at a time** (or in groups of at most 3 with options). Don't dump the whole questionnaire.
3. **Every foundations item is decided explicitly.** "Apply" or "dismiss with a written reason." Never skipped silently — what's skipped silently is what blows up in month 4.
4. **The first deploy comes before the first feature.** If there's no deploy pipeline when Phase 2 ends, the phase isn't done.
5. **No phase is declared done without its evidence**: written intake, checklist with decisions, deployed and verified skeleton, committed docs.

## Signs you're violating the skill (stop)

- "It's a small project, I'll start with the CRUD and clean up later" → the easy-CRUD-first move is the #1 anti-pattern (see walking-skeleton.md).
- "We'll deal with the deploy at the end" → deploying at the end ALWAYS surfaces config/secrets/migration problems once they're expensive to fix.
- "I'll add tests / CI later" → a project without CI from commit 1 accrues invisible debt nobody volunteers to pay down.
- "We'll add multi-tenancy if we get more customers" → multi-tenancy isn't retrofittable; it's decided in the intake (see foundations.md).
- "I already know what the user wants, I'll skip the intake" → what you think you know and what the user confirms differ in 100% of cases on at least one point that changes the architecture.

## Expected output of the full skill

When the 4 phases are done the repo has: `docs/00_intake.md` (answers), `docs/01_decisions.md` (foundations applied/dismissed with reasons), a deployed walking skeleton with green CI, `CLAUDE.md` with the project's non-negotiable rules, and a tracker with the first 5 milestones ordered by risk.
