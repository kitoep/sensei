# sensei 🥋

[Español](README.md) | English

**Skills for Claude Code that make the model work like a senior, not a junior in a hurry.**

By default the model improvises, tells you it's "done" without having run it, and agrees with everything you say. These 15 skills put a stop to that. They make it ask before dumping code, run things before claiming victory, and tell you when something's wrong even if you don't want to hear it. And on the stuff that's expensive to undo later (databases, architecture, security, AI bots) they don't let it take the easy way out.

Each skill comes with concrete rules: checklists, templates, and the list of excuses the model tends to reach for ("it's a small change", "I'll add tests later") so it can't get away with them.

## Install

Inside Claude Code:

```
/plugin marketplace add kitoep/sensei
/plugin install senior-engineer-en@sensei
```

There are two versions, English and Spanish. Install one:

- `senior-engineer-en@sensei` for English (skills `senior-engineer-en:*`)
- `senior-engineer-es@sensei` for Spanish (skills `senior-engineer-es:*`)

Why two versions instead of one that translates itself? Because each skill has forbidden-phrase tables that catch exactly what the model is about to write. If you code in English your agent replies in English, so it needs the English version for those tables to work. In `/plugin` open **Discover**, see both and pick one.

The wizard asks for the scope: **User** for all your sessions, or **Project** for the current project only. Then run `/reload-plugins` or restart the session.

### One skill at a time

If you don't want all 15, each skill installs on its own (it's the same copy of files; install the full suite or the individual ones, not both):

```
/plugin install fixer@sensei
/plugin install data-schema@sensei
/plugin install reparador@sensei          # Spanish-version skills go by their Spanish names
```

You can also do it visually: in `/plugin` open **Discover** and pick from the list. Individual skills work on their own; when one mentions another, that reference is optional and won't break if the other isn't installed.

### Update or remove

```
/plugin marketplace update sensei
/reload-plugins
/plugin uninstall senior-engineer-en@sensei
```

## The 15 skills

| Skill | What it makes the model do |
|---|---|
| `shaping` | When an idea or requirement isn't fully defined (at any point in the project, not just the start), it stops before building on assumptions. It asks one thing at a time, proposes 2 or 3 approaches with their downsides, and doesn't let go until the idea is shaped and approved by you. Catching itself about to fill a gap with its own assumption is itself a trigger. |
| `planning` | Before writing a plan it asks the questions that matter (the five layers) and checks what already exists in the project, so it doesn't rebuild something that was already there. It splits the work into tasks you can verify and orders them by risk. |
| `architect` | It won't recommend a stack without first asking you the nine things that actually drive the decision. It brings scale fantasies back down to earth and ties every choice to a real condition of yours. If it can't tie it to one, it's fashion, and it says so. |
| `project-bootstrap` | No code until the intake is done. The decisions that get brutally expensive to change later (multi-tenancy, auth, data deletion) are made on day one. First a skeleton that runs end to end, then the features. |
| `designer` | A rule it won't skip: it won't build a single screen until there's a design system you've approved. It brings the catalog of the 12 signs that give away AI-made design, requires the 4 states of every screen (loading, empty, error, with data), and runs before any other design or aesthetic skill. |
| `critic` | It criticizes you straight, without buttering you up. It opens with the verdict on the first line, first builds the strongest version of your idea and then attacks it, and gives you at least 3 risks and 1 alternative. If you push back, it doesn't cave. |
| `fixer` | To fix a bug it reproduces it first; no repro, no fix. It makes hypotheses you can rule out, goes for the root cause, and looks for the same pattern across the whole repo. It watches the test that proves the fix fail before trusting it. And it has a protocol for when something "still fails". |
| `tester` | It builds the test plan from what must not break, starting with the cases that break things and leaving the happy path for last. It always includes the UI. It calls nothing tested if important cases are still unrun. |
| `security-compliance` | Two modes: audit code for holes, and warn you while you build. It checks OWASP with concrete searches, isolation between customers (multi-tenant and RLS), and data protection (GDPR, PCI, and Mexico's law). It only flags something as serious if it builds the scenario that proves it. |
| `verifier` | Nothing is called done without the proof that it runs, pasted in the same message. It tells apart "verified" from "I applied it but didn't test it", and it doesn't hand out the ✅ for free. |
| `data-schema` | Databases and migrations that don't take production down. It asks about queries and volume before designing, never uses float for money, makes dangerous changes in two steps (expand/contract), puts the guarantees in the database, and checks indexes with EXPLAIN. |
| `ai-features` | LLM features without making things up. It's not allowed to write model names or parameters from memory (it verifies them against the docs), puts everything behind a single provider-agnostic layer, builds tools with strict contracts and per-user scoping, adds defenses against prompt injection, and won't change a prompt without running its test set. |
| `git-delivery` | Commits that do one thing, with the diff reviewed file by file (never a blind `git add .`). Messages explain the why, not the what. PRs cover a single topic and come with a description you can audit. Nothing destructive unless you ask for it, and when the work is done it presents the options (merge, PR, keep, discard) instead of deciding alone. |
| `receiving-feedback` | When you correct it, it checks against the code before agreeing (you can be right, wrong, or half right). It doesn't cave to pure insistence: it asks for the new argument. When it's the one that's wrong, it says so dry and fixes it, no paragraphs of apologies. And if it finds a mistake of its own you haven't seen, it tells you first. |
| `skill-building` | The method for building new skills to this standard: first the concrete model failure it fixes (no named failure, no skill), the suite's conventions, and the mandatory test before publishing: the same request with and without the skill on a project with planted traps, verifying the outcome on disk. |

## How they're built

- Each skill has a short `SKILL.md` (the protocol, the rules, the red flags) and a `references/` folder with the detail, which the model opens only when it needs it. Until it's used, it barely takes up context.
- The content is in English and the descriptions too (that's how Claude Code matches them). The forbidden-phrase tables are in English on purpose, because they catch exactly what the model is about to write.
- The line that runs through all of them: **"Violating the letter of these rules is violating their spirit."** No task is "too simple" to skip the process.
- The original 12 were tested with a simple method: the same request, once without the skill and once with it, over test projects with traps set on purpose. In all 12 the skill changed the model's behavior for the better. From demanding proof where there used to be an empty "done ✅", to stopping a migration that took production down and plugging a cross-customer data leak in a bot.

## Manual install (no plugin)

Copy the folders under `senior-engineer-en/skills/` (or the ones under `senior-engineer-es/skills/`) into your skills folder (`~/.claude/skills/`) and restart the session. That way you lose the marketplace's automatic updates.

## License

MIT. [kitoep](https://github.com/kitoep)
