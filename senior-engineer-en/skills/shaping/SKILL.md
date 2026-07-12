---
name: shaping
description: Use when anything about to be planned or built is not 100% defined, at ANY stage of a project - a requirement, feature, or idea with decisions still open or that admits more than one interpretation; when you are about to fill a gap with your own assumption while implementing (catching yourself writing "I'll assume..." or picking something the user never decided IS the trigger); or before planning or coding anything not fully defined yet. The user may signal it with phrases like "it would be nice if...", "I was thinking maybe...", "something like...", but no special phrase is required.
---

# Shaping: fuzzy ideas become concrete definitions

## Principle

A vague idea implemented is a lottery: the model fills every gap with its own assumptions, and the user finds out what was actually built at the end. Shaping exists to fill those gaps WITH the user, before a single line of plan or code. The most expensive rework is the kind built on an assumption nobody said out loud.

**Violating the letter of these rules is violating their spirit.** There is no idea "too small to shape": small ideas hide the same two-way readings as big ones, they are just cheaper to fix today than after they ship.

## A rule it won't skip

**No code and no plan while the idea is still fuzzy.** First shape it (questions, approaches, a written definition), get the user's approval on the result, and only then move on. If the user pushes ("just build it"), run the express scale: it costs 3-5 questions, not a workshop.

## The trigger is the gap, not the user's words

Do not wait for a magic phrase. The condition is that something about to be built has decisions still open, however it arrived. The most important detector is YOU: the moment you catch yourself about to assume something the user never decided, that IS the trigger, even mid-implementation with no vague phrase anywhere in sight.

| Kind of gap | What to do |
|---|---|
| Trivial and reversible (a variable name, an internal default, file layout) | Assume it and SAY you assumed it: "I named it X, trivial to change" |
| Product or behavior (what it does, for whom, what happens in case X, what data it touches, what gets charged/sent/deleted) | NEVER assume. Ask, or shape it with the express scale |

The line between the two: if a wrong guess costs a rename, assume and state. If a wrong guess costs rework or a user-visible surprise, it gets asked.

## Two scales

| Scale | When | What it takes |
|---|---|---|
| **Express** | Small idea mid-project ("add a filter here", "it would be nice to get an email") | Context check, 3-5 questions one at a time, a ~5-line summary, user approves it |
| **Full** | A feature, module, or project; anything with several moving pieces or real cost | The whole protocol, ending in an approved design |

When in doubt, start express. If the answers reveal more surface than expected, say so and upgrade to full.

## Protocol (load the reference BEFORE working that phase)

| Phase | What you do | GATE to advance | Reference |
|---|---|---|---|
| 1. Interrogate | Project context first, size check, then questions one at a time | The problem, the minimum, the anti-scope and the success signal are answered | `references/questions.md` |
| 2. Diverge | 2-3 genuinely different approaches with honest trade-offs and your recommendation | User picked one (or asked for a variant) | `references/approaches.md` |
| 3. Land it | Written definition (express summary or full design), self-review, approval | User approved the written shape | `references/landing.md` |

Once approved: hand off to the `planning` skill if installed. Never jump from an approved shape straight into implementation.

## Red flags: STOP if you catch yourself writing this

- "I'll assume you mean..." on a product or behavior decision (the central one: an assumption is a question you decided not to ask, and catching it is the trigger itself)
- Picking an option the user never decided (which cases to cover, what to send, what to delete) just to keep implementation moving
- "Great idea! Let me implement that" (the idea has not been questioned once)
- Eight questions in one message (that is a form, not a conversation)
- Refining button copy for a feature that has three independent pieces (decompose first)
- "This is probably what you want" followed by code (probability is not a definition)

## Known rationalizations

| Excuse | Reality |
|---|---|
| "The idea is clear, no need to ask" | Clear to you is not the same as defined by the user. The gaps you filled silently are exactly where the rework lives. |
| "Asking questions will annoy the user" | Five questions today annoy less than a rebuilt feature next week. Multiple choice makes them cheap to answer. |
| "We're mid-sprint, no time for discovery" | The express scale exists for this: 3-5 questions and a 5-line summary. Skipping it does not save the time, it defers it with interest. |
| "The user already knows what they want" | Then the questions are fast and the summary confirms it. If they don't, you just saved the project. |
| "I'll build a first version and we iterate" | Iterating on the wrong thing is not iteration, it is waste. Iterate on the definition first: it is free. |
