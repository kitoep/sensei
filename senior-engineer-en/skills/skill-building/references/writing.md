# Writing and validating: suite conventions

## Style rules (hard)

- **Prescriptive, not advisory.** Checklists, gates, tables and exact rules. "Consider adding tests" is advice; "no fix ships without the regression test seen failing" is a rule. The operating model must not depend on its own judgment.
- **Zero em-dashes** (the long dash character) in any content. Use periods, commas, colons or parentheses. Verify with a search for the character before finishing, not by eye.
- **Native language, never literal translation.** A Spanish suite is written the way a Spanish-speaking senior talks (no calques like "puerta dura" for hard gate); an English suite the way an English-speaking senior talks. Forbidden-phrase tables adapt to what the model actually writes in THAT language ("Done ✅" vs "Listo ✅"), because they match against the model's own draft.
- **Signature doctrine** in every skill: "Violating the letter of these rules is violating their spirit." It closes the "this case is special" exit.
- **Names**: kebab-case, native to the suite's language, no accents or special characters in folder names (portability across filesystems).

## Cross-references between skills

- Always conditional: "use the `fixer` skill if installed (its phase 5 covers this); if not installed, this rule still applies as follows: ...". Never assume the other skill exists.
- Never reference by file path across skills except inside a conditional mention.
- When a skill is renamed, grep the whole suite for the old name: SKILL.md bodies, references, marketplace entries, README tables, install examples. Renames rot in five places at once.

## Delegating the writing (subagents or another model)

Delegation works if the brief carries three things:

1. **A domain-expert role**: "act as a senior DBA who has run Postgres in production for years", not "write a database skill".
2. **The package's purpose**: why the suite exists and who operates it, so tone and assumptions land right.
3. **The baseline failures**: the concrete list of what the model does wrong, so every section attacks something real.

Constrain the output shape (file names, section list, size targets) in the brief. Vague briefs produce generic documents.

## Personal validation: read everything, trust nothing

The writer's report is not evidence; it is written by the party being graded, and it always says the work went great.

After any delegated writing:

1. Read EVERY file end to end. Not a sample, not the summary.
2. Check against the design: baseline failures all covered? description condition-based? red flags verbatim and in the right language? references match the protocol table's file names?
3. Check facts you can check: SQL that claims to avoid locks, commands, API names. If the skill teaches something, the teaching must be correct, and you verify it with your own knowledge or against docs.
4. Run the mechanical checks: search for em-dashes, search for leftover source-language words in a translated suite, validate the plugin package if there is one.
5. Fix or send back with the specific defect. "Looks good" after skimming is the exact false-green this suite exists to kill.
