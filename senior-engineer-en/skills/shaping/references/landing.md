# Landing it: the written shape, reviewed and approved

## Express summary (small ideas, or gaps caught mid-implementation)

Five to six lines, written in the conversation, approved before anything else happens:

```
SHAPED: <name of the idea>
What: <one sentence, what gets built>
For: <who uses it and for what problem>
In: <the 2-4 things v1 includes>
OUT: <what it explicitly does not do; the neighbor features that were cut>
Done when: <the observable signal that it worked>
```

Ask for approval explicitly ("does this match what you had in mind?"). Silence is not approval. If the user corrects anything, update the summary and show it again.

When the skill fired mid-implementation (you caught yourself about to assume a product decision), the express summary can shrink to the open decision itself: state the gap, the options, your recommendation, and wait. Then resume the work with the answer written into the task, not into your head.

## Full design (features and projects)

Present the design in sections, each scaled to its complexity (a few sentences if simple, a couple of paragraphs if not), and ask after EACH section whether it looks right before moving on. Do not dump the full design in one message and ask "thoughts?".

Cover, in this order:

1. Problem and success signal (from the questions phase, now in final form)
2. Chosen approach and why (with the discarded ones in one line each)
3. Scope of v1 and anti-scope
4. The pieces: components, data, integrations, each with one clear purpose
5. Risks and open edges (what could force a redesign, what was left explicitly unresolved)

If the project has independent sub-parts, each gets its own shaping and its own plan. Do not write one mega-design for three projects.

## Self-review before delivering

Read your own summary or design with fresh eyes and fix inline:

- [ ] Any "TBD", "we'll see", or placeholder left? Resolve it or move it to open edges explicitly.
- [ ] Does any section contradict another?
- [ ] Can any requirement still be read two ways? Pick one reading and write it down.
- [ ] Does the scope fit ONE implementation plan? If not, split before approval, not after.

## The handoff

Once the user approves the written shape:

- If the `planning` skill is installed, hand off to it: the approved shape is its input.
- If it is not installed, tell the user the definition is ready to plan against, and that the next step is an implementation plan, not code.
- If the skill fired mid-implementation for a single decision, go back to the task you paused, carrying the approved answer.
- NEVER jump from a freshly approved shape straight into implementation. Approval of the idea is not approval of a plan that does not exist yet.

## What this phase is not

- Not a contract: if implementation later reveals the shape was wrong somewhere, come back and re-shape that part with the user. Do not silently drift.
- Not documentation theater: the summary or design exists so two people agree on the same thing, not to decorate the repo. Keep it as short as agreement allows.
