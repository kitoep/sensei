# Skill design: from named failure to loadable protocol

## Baseline failures: the reason the skill exists

Before writing a line, answer in writing:

1. **What does the model do wrong today, concretely?** Not "it could plan better": "it declares work done without running anything", "it invents SDK parameter names", "it commits .env files with `git add .`".
2. **Where did you see it?** A real session, a real incident, a documented tendency. One real example per failure. If you cannot produce one, you are designing for an imaginary problem.
3. **Is a skill the right fix?** Automations that must ALWAYS run belong in hooks or settings, not skills. Knowledge the repo already documents belongs in the repo. A skill fixes judgment and behavior, not missing configuration.

The list of baseline failures becomes your test plan later: each failure should map to a trap in the RED/GREEN test.

## Anatomy of a SKILL.md (the orchestrator stays light)

| Section | Job | Size |
|---|---|---|
| Frontmatter `name` | Identifier, kebab-case, native to the suite's language | 1 line |
| Frontmatter `description` | The ONLY thing the model sees when choosing skills. Conditions first, phrases as examples | 1 long line |
| Principle | Why the doctrine exists, in two or three sentences the model can internalize | ~5 lines |
| Hard rule | The one gate that is never negotiable | ~3 lines |
| Protocol table | Phases with a GATE each, pointing to references | 1 table |
| Red flags | What the model LITERALLY writes when it is about to fail | 5-7 bullets |
| Rationalizations | Excuse next to reality, closing each exit | 1 table |

Total target: 50-70 lines. Everything detailed (procedures, templates, catalogs, examples) goes into `references/` files that the model loads only for the phase it is in. A 200-line SKILL.md gets skimmed; a 60-line one gets followed.

## Descriptions: conditions, not phrases

The description is the trigger. Two rules:

1. **Lead with the condition.** "Use when anything about to be built is not 100% defined" fires on every wording. "Use when the user says 'it would be nice if'" fires on one.
2. **Phrases only as examples**, marked as such ("e.g. ..."). They help matching; they must never be the definition.

Include self-detection when it applies: the model catching ITSELF about to commit the failure ("about to write 'should work'", "about to fill a gap with an assumption") is often a stronger trigger than anything the user says.

## Red flags and rationalizations that actually work

- Red flags quote what the model writes, verbatim, in the language it will write it: "Done ✅", "Should work", "I'll assume you mean...". The model pattern-matches its own draft against them.
- Rationalizations are pre-refutations. List the excuses the model will generate to skip the rule ("it's a small change", "no time for tests") and put the reality next to each one. An excuse that is already answered in the skill loses its power.
- Both lists come from the baseline failures. If a red flag does not trace to a failure you named, cut it.

## Scope: one skill, one doctrine

If the protocol table needs more than ~6 phases, or the references cover two unrelated crafts, it is two skills. Cross-reference between skills is fine but always conditional ("use the `fixer` skill if installed; if not, this still applies"): each skill must work standalone because users install them one at a time.
