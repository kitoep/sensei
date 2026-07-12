---
name: planning
description: Use when planning a new feature, project, or significant change before writing code — when the user says "how do we do this", "plan this", "I want to build X", asks for an implementation plan, or is about to start anything non-trivial without a written plan. Also use when a task keeps growing mid-implementation, a symptom that planning was skipped.
---

# Planning — plan like a senior engineer, not like a list generator

## Relay with `shaping`

If what arrives is still fuzzy (open decisions, more than one possible reading), the `shaping` skill goes first if installed: this skill works on ideas already defined. And the other way around: if the idea arrives shaped and approved, do not repeat that intake; build on what was already answered.

## Principle

A plan is not a list of steps: it is a set of **decisions made with information that is cheap today and expensive tomorrow**. 80% of the value of planning is in the questions you ask BEFORE writing the plan; a plan written without asking only documents assumptions.

**Golden rule:** if you can't name which question you asked the user and which answer changed the plan, you didn't plan — you guessed.

## Mandatory flow

1. **Understand the existing context** — explore the project's code, docs and tracker BEFORE asking anything. Never ask something the repo already answers.
2. **Ask** — load `references/questions.md` and walk the 5 layers. ONE question per message, multiple choice where possible. Stop when answers stop changing the plan.
3. **Decompose and order** — load `references/decomposition.md`. Tasks with a verifiable deliverable, uncertainty first.
4. **Write the plan** — load `references/plan-template.md` and fill the exact template. Pass the self-review checklist before presenting it.
5. **User approval** — the plan is not executed until the user approves it. If they request changes, go back to the relevant step.

## Which reference to load

| Moment | File |
|---|---|
| Before asking the first question | `references/questions.md` |
| When splitting the work into tasks | `references/decomposition.md` |
| When writing the final plan | `references/plan-template.md` |

## Signs you're failing

- You wrote a plan without asking a single question → delete the plan, go back to step 2.
- The plan has tasks like "make progress on X" or "improve Y" → not verifiable, re-decompose.
- You ask 6 things in a single message → the user answers 2 and you lose the other 4.
- The user says "do whatever you think is best" to everything → present 2-3 options with your recommendation and trade-offs; a concrete recommendation gets better information than an open question.
- You discover mid-implementation something a layer (c) or (d) question would have revealed → record the missing question; the cost of that omission is the argument for not skipping this flow next time.
