# Written plan template

Use this exact structure. A plan that omits a section must say why it omits it (one line), not skip it silently. Save it as a file in the repo (`docs/plans/YYYY-MM-DD-<topic>.md` or wherever the project already keeps plans) — a plan that only lives in the chat is lost.

```markdown
# Plan: <short name>

**Date:** YYYY-MM-DD · **Status:** draft | approved | in progress | completed

## 1. Context and goal
<2-5 sentences: what problem is being solved, for whom, and what success
looks like in verifiable terms. No jargon invented during the conversation.>

## 2. Decisions made
<Table. Only REAL decisions (there were alternatives), with their why.
Mark the irreversible ones.>

| Decision | Alternatives discarded | Why | Irreversible? |
|---|---|---|---|

## 3. Out of scope
<Explicit list of what this version does NOT include. If something was
deferred from a prior list/tracker, name it and say where it was deferred to.>

## 4. Assumptions and risks
<The structural assumption and how/when it's verified. Risks with their
mitigation or "accepted". Relevant legacy data and external boundaries.>

## 5. Tasks
<Numbered, in execution order (risk first, dependencies respected).>

### Task N: <title with deliverable>
- **What:** <2-3 sentences; the expected diff described>
- **Likely files:** <paths>
- **Verification:** <concrete command/test/observation that proves it's done>
- **Depends on:** <prior tasks or "—">
- **Risk:** <what can go wrong here, or "low">

## 6. Done criteria for the whole
<Final checklist: what must be true to declare EVERYTHING done.
Includes: all the plan's tests green, typecheck/build/CI green,
docs updated if the WHAT changed, and the end-to-end demo/flow the
user can run to see it working.>
```

## Self-review checklist (before presenting the plan)

Walk ALL the points; fix inline and only then present:

1. **Placeholders:** is there any TBD, "to be defined", empty section or "etc." hiding work? → resolve it or turn it into an explicit question to the user.
2. **Contradictions:** does any task contradict a decision from section 2? Does out-of-scope contradict the goal?
3. **Verifiability:** does EVERY task have a concrete verification? Hard test: could another agent, without this conversation, execute the task and know it's done? If it needs context from the chat, that context is missing from the plan.
4. **Size:** any task with a structural "and" or a diff not describable in 2-3 sentences? → split it.
5. **Order:** is the highest uncertainty at the start? Is the structural assumption verified in task 1 or before?
6. **Reconciliation:** if a prior tracker/spec/backlog existed, is every item from it in the tasks, in out-of-scope, or flagged for re-planning? None can simply not appear.
7. **Ambiguity:** any phrase interpretable two ways? Pick one and write it.
8. **YAGNI:** are there tasks nobody asked for and no success criterion needs? → out (to section 3 if worth recording).

## Presenting to the user

Present the plan by sections (not a wall of text), ask for approval, and incorporate changes where they belong (a new decision can reorder tasks). **The plan is not executed without explicit approval.** Once approved, scope changes during execution are negotiated against the plan — "while we're at it, let's add X" reopens section 3, it doesn't sneak in as task 7b.
