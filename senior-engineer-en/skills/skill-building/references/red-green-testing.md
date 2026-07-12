# RED/GREEN testing: the only proof a skill works

A skill that reads well proves nothing. The proof is behavioral: the same model, the same request, with and without the skill, behaving differently in the direction you designed.

## The method

1. **RED**: a fresh agent gets a neutral request WITHOUT the skill. This is the baseline.
2. **GREEN**: another fresh agent gets the SAME request, with one added instruction: read and follow the skill before acting.
3. Compare the two behaviors against the baseline failures the skill was designed to fix.

Rules that keep the test honest:

- **The prompt is neutral and identical.** Written the way the real user writes (casual, no hints, no keywords from the skill). If the prompt telegraphs the expected behavior, both runs will behave and the test is void.
- **Isolate the RED.** If the agent can wander into the folder that contains the skill (or a CLAUDE.md that teaches the same doctrine), it is contaminated and is not a baseline. Anchor the task to a sandbox path away from the skill source. If contamination happens, rerun; note the accident (a model voluntarily adopting a skill it found is evidence of discoverability, not a baseline).
- **Fresh agents, not your own session.** Your context already contains the skill; you cannot play RED.

## Planted traps

Design the sandbox so skipping the doctrine produces a visible failure. Each baseline failure gets a trap:

| Failure the skill fixes | Trap |
|---|---|
| Declares done without running anything | A test file the prompt never mentions, with an edge case the obvious implementation gets wrong |
| Blind `git add .` | A working tree with a fake secret in `.env`, a debug log, and unrelated reformatting mixed in |
| Invents API/SDK details | A task requiring current model or parameter names |
| Builds on assumptions | A request whose requirements legitimately read two ways |

The trap must be discoverable by a diligent agent (reading the project, running the tests) and missable by a hasty one. That gap is what the test measures.

## Verify on disk, not on claims

The agents' final messages are claims. Ground truth is the artifact:

- Run the planted tests yourself over each agent's output.
- Inspect `git log --stat` and diffs: how many commits, what got committed, did the secret enter history.
- Grep the generated code for the failure (the unscoped query, the hardcoded model name, the missing layer).

If GREEN claims something you can check, check it. A GREEN that follows the ritual but produces the same defect as RED is a failed skill with good manners.

## Scoring and honesty

- **Changed behavior in the designed direction**: the skill works. Record RED behavior, GREEN behavior, and the on-disk evidence.
- **RED also behaved well**: the test proved nothing about the skill. Say so. Check whether the harness or project context gave RED the doctrine for free, and note that the delta would be larger without it. Do not inflate the result.
- **GREEN ignored the skill or behaved the same**: the skill failed. The usual suspects: description did not fire (conditions too narrow), SKILL.md too long to follow, rules advisory instead of gated. Fix and retest; do not publish on faith.
- One run per pair validates direction, not statistics. Say that too.

Record the evidence (prompts, both behaviors, on-disk checks) in a test log kept OUT of the published package.

## Publishing checklist

- [ ] Marketplace entry added (and the suite auto-includes the new folder).
- [ ] README table row in every language of the package, counts updated.
- [ ] Plugin version bumped (users on a pinned version never see the update otherwise).
- [ ] Package validator passes (`claude plugin validate`).
- [ ] Final greps clean: em-dashes, old names after renames, leftover source-language text.
