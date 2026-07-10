# The questions before planning

Five layers, in order. Each question carries its why and the error it prevents — if you already know the answer when you read it (from the code, the docs, or the conversation), do NOT ask it: asking something already answered burns the user's patience and dilutes the questions that do matter.

**Mechanics:** one question per message. Multiple choice whenever the answer space is enumerable (2-4 options + your recommendation marked). Open-ended only when you genuinely can't anticipate the answer. Stop when one more answer would no longer change the plan: the goal isn't to complete the questionnaire, it's to eliminate the unknowns that change decisions.

---

## Layer A — Real intent (what problem it is, not what feature it is)

The user usually arrives with a solution ("I want a button that exports to Excel"), not with the problem ("accounting asks me for this data every Friday"). Planning the literal solution without understanding the problem produces correct features that don't help.

| Question | Why | Error it prevents |
|---|---|---|
| What problem does this solve, and for whom? | The proposed solution may not be the best one for that problem. | Building what was asked instead of what was needed. |
| What does success look like? What will someone be able to do that they can't today? | Turns a wish into a verifiable criterion. | "Done" with no definition → argument at delivery. |
| What happens if it's NOT done? What hurts today? | Calibrates urgency and the justifiable size of the solution. | Over-engineering for a minor pain (or the reverse). |
| Is this for production, an experiment, or a demo? | The standard for quality, tests and robustness changes radically. | Applying production rigor to a throwaway prototype, or vice versa. |

## Layer B — Scope (the edge matters more than the center)

| Question | Why | Error it prevents |
|---|---|---|
| What is explicitly OUT of this version? | The unsaid is assumed in; scope only grows if it isn't bounded in writing. | Silent scope creep; phantom features the user assumed were included. |
| What is the minimum version that already does something useful? | There's almost always an intermediate deliverable cut that validates the direction. | A weeks-long big-bang with no intermediate feedback. |
| Which decisions here are reversible and which aren't? | The irreversible ones (public schema, API format, IDs, external contracts) deserve 10x more analysis; the reversible ones are decided fast and fixed cheap. | Spending the analysis on the reversible and deciding the irreversible carelessly. |
| Are there edge cases you already know exist? (users without X, old data, concurrency) | The user usually knows the rare cases in their domain and doesn't mention them until they break. | Designing only for the happy path. |

## Layer C — Existing context (the plan lives in a repo, not in a vacuum)

This layer is answered mostly by EXPLORING, not asking. Ask only what the repo can't tell you.

| Question / check | Why | Error it prevents |
|---|---|---|
| What existing code does this touch or replace? (explore) | Every change lands on top of existing patterns, conventions and debt. | Reinventing something that already exists; breaking an established pattern. |
| Is there a spec, tracker or backlog to reconcile against BEFORE writing the plan? | If a prior list of tasks/requirements exists, every item is either included or explicitly re-planned with the user — never silently dropped. | Agreed features that vanish from the plan because nobody cross-checked the lists (a real and costly error). |
| What existing pattern should this follow? (layers, validation, tests, naming) | Consistency is worth more than the personal preference of whoever's writing. | An "island" module with its own conventions no one else understands. |
| Does this change alter contracts others consume? (API, schema, events, jobs) | Broken consumers show up in production, not in your tests. | Accidental breaking changes. |

## Layer D — Risks (find the assumption that topples everything)

| Question | Why | Error it prevents |
|---|---|---|
| What is THE assumption that, if false, invalidates the whole plan? | Every plan rests on 1-2 structural assumptions (the external API allows it, the data exists, the volume fits). Verify them FIRST, with a spike if needed. | Discovering at task 8 of 10 that the premise of task 1 was false. |
| Which part has the most technical uncertainty? | That part goes at the start of the execution order, not the end. | Leaving the hard thing for last and crashing with everything else already built on top. |
| Are there legacy data/states that don't meet the new rules? | New code assumes invariants that old data violates (nulls, duplicates, impossible states). | Migrations that blow up in production with real data. |
| What can fail at the boundary with external systems? (timeouts, duplicate webhooks, rate limits) | External things fail in ways local ones don't; idempotency and retries are designed, not patched. | Duplicate effects and inconsistent states under partial failures. |
| Does this touch security, personal data or money? | If so, the plan needs its controls and adversarial tests as first-class tasks, not as "pending". | Security as an afterthought. |

## Layer E — Constraints (the reality of the project)

| Question | Why | Error it prevents |
|---|---|---|
| Is there a deadline or event that fixes a date? | With a hard date, scope is the variable; with no date, quality rules. | Cutting quality when scope was what needed cutting. |
| Infra/services budget? (do we optimize cost or availability?) | Changes entire architecture decisions. | Designing for a scale/budget that isn't the real one. |
| What compatibility must be preserved? (versions, old clients, public URLs) | The public surface is a contract even if it isn't documented. | Breaking unknown consumers. |
| Can the migration take downtime or must it be live? | Defines whether the plan needs an expand/contract strategy and its own deploy steps. | Planning as if the DB were empty. |

---

## Anti-patterns when asking

- **Full interrogation:** asking all 20 questions. Ask only the ones that change THIS project's plan; 4-6 usually suffices.
- **Filler question:** asking what the code already answers. Explore first.
- **Asking without recommending:** in multiple choice, always mark your recommendation and its why — the user decides better against a stance than against a neutral menu.
- **Accepting vague answers on the irreversible:** if the answer to an irreversible decision is "whatever you think", present the options with concrete consequences and ask for an explicit decision. On the reversible, "whatever you think" is a green light: decide yourself and document it.
