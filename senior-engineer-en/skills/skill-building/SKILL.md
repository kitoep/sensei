---
name: skill-building
description: Use when creating a new skill, editing an existing one, or reviewing a skill before publishing; when a repeated model failure suggests a new skill should exist; and before adding any skill to a suite without a RED/GREEN test proving it changes behavior.
---

# Skill-building: skills that change behavior, not decoration

## Principle

A skill is not documentation. It is a behavioral patch for a model failure you can name. If you cannot describe what the model does WRONG without the skill, with a real example, there is nothing to fix and the skill is decoration that costs context. And a skill nobody tested is a hypothesis: the only proof that it works is watching the same model behave differently with it and without it.

**Violating the letter of these rules is violating their spirit.** There are no skills "too obvious to test" and no drafts "good enough to publish as is".

## A rule it won't skip

**No skill ships without (1) a named baseline failure and (2) a RED/GREEN test.** RED: a fresh model gets a neutral request without the skill. GREEN: same request with it. If the two runs behave the same, the skill failed, however nice it reads.

## Protocol (load the reference BEFORE working that phase)

| Phase | What you do | GATE to advance | Reference |
|---|---|---|---|
| 1. Justify | Name the baseline failures: what the model concretely does wrong today | Each failure has a real example, not a vibe | `references/skill-design.md` |
| 2. Design | Description triggered by conditions, light SKILL.md, references on demand | Anatomy checklist passes | `references/skill-design.md` |
| 3. Write | Suite conventions: red flags with what the model literally writes, rationalization tables | Style checklist passes, zero em-dashes | `references/writing.md` |
| 4. Validate | If writing was delegated, read EVERYTHING yourself; never trust the writer's report | You read every file end to end | `references/writing.md` |
| 5. Test | RED/GREEN with a neutral prompt and planted traps; verify results on disk | Behavior changed in the right direction, evidence recorded | `references/red-green-testing.md` |
| 6. Publish | Marketplace entry, README row, version bump, package validation | Validator passes and docs match reality | `references/red-green-testing.md` |

## Red flags: STOP if you catch yourself writing this

- "This skill will help the model with..." and no example of the failure it fixes (help is not a failure; name what goes wrong without it)
- A description listing user phrases instead of conditions ("when the user says X" misses every case where they phrase it differently)
- "The skill reads well, ready to publish" (reading well proves nothing; only RED vs GREEN proves anything)
- Accepting a delegated draft based on the writer's summary (the summary always says it went great; read the files)
- "RED also did fine, but the skill adds structure" (then the test proved nothing; say so instead of shipping on faith)
- A 200-line SKILL.md (the model won't hold it; push detail into references and keep the orchestrator light)

## Known rationalizations

| Excuse | Reality |
|---|---|
| "It's obviously useful, testing is overkill" | Half of obviously useful skills change nothing because the model already behaved that way. The test costs two runs; shipping dead weight costs context forever. |
| "I'll test it later with real use" | Real use has no control group. You will never know if the skill works or the task was easy. |
| "The subagent's report says everything is in place" | The report is written by the party being graded. Read the files. |
| "One good example phrase in the description is enough" | Phrases are one signal. The model must recognize the CONDITION, or it will miss every rewording. |
| "More rules make it stronger" | Every rule the model can't recall is noise. Fewer rules, harder gates, tables it can pattern-match against. |
