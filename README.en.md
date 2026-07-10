# sensei 🥋

**[Español](README.md)** | English

**Senior-engineer discipline for Claude Code.**

12 skills that make any model work like a seasoned senior engineer: hard gates before acting, executed evidence before declaring anything done, honest critique instead of flattery, and zero improvisation on expensive decisions (DB schemas, architecture, security, AI features).

These are not "tips" — they are prescriptive protocols: checklists, templates and exact rules, written so the model cannot rely on its own judgment or rationalize exceptions. Every skill ships with its table of known rationalizations ("it's a trivial change", "no time for tests"...) and the reality next to each one, closing the exits.

> This suite comes in two fully native versions — install `senior-engineer-en` for English or `senior-engineer` for Spanish. Each is authored in its language (not auto-translated) so the forbidden-phrase tables match what the model actually writes.

## Install

Inside Claude Code:

```
/plugin marketplace add kitoep/sensei
/plugin install senior-engineer-en@sensei     # English
```

The suite ships in **two languages** — install one or the other (not both):

- `senior-engineer-en@sensei` — **English** (skills `senior-engineer-en:*`)
- `senior-engineer@sensei` — **Spanish** (skills `senior-engineer:*`)

Why two versions instead of one auto-translated? Because each skill's "forbidden-phrase" tables literally match what the model is about to write; if you code in English your agent replies in English and needs the English tables to fire. In `/plugin` → **Discover** you'll see both and pick one.

The wizard asks for scope: **User** (all your sessions) or **Project** (current project only). Then `/reload-plugins` or restart the session.

### À la carte

Don't want all 12? Each skill can be installed separately (same file copy; pick suite **or** individual skills, not both):

```
/plugin install senior-engineer-fixer@sensei
/plugin install senior-engineer-esquema-datos@sensei
```

Or visually: `/plugin` → **Discover** tab → pick from the list. Individual skills work standalone — cross-references are conditional and don't require the rest of the suite.

### Update / uninstall

```
/plugin marketplace update sensei
/reload-plugins
/plugin uninstall senior-engineer@sensei
```

## The 12 skills

| Skill | What it enforces |
|---|---|
| `planning` | Five layers of questions before writing any plan; reconcile against the project's real state FIRST (never rebuild what already exists); decomposition into verifiable tasks ordered by risk. |
| `architect` | No stack recommendations without a 9-question intake about real conditions; deflates scale fantasies; every choice traces to a condition ("if it doesn't trace, it's fashion"). |
| `project-bootstrap` | Zero code before intake; foundations that are brutally expensive to retrofit (multi-tenancy, auth, deletion) are decided on day 1; walking skeleton before features. |
| `designer` | Hard gate: no UI code without an approved design system; catalog of the 12 tells of AI-made design; 4 mandatory states per screen; acts as a process gate BEFORE any aesthetic skill. |
| `critic` | Critique without flattery: the first sentence is the verdict; steelman → load-bearing assumption → minimum 3 risks + 1 alternative; holds position under pressure. |
| `fixer` | Debugging with gates: no reproduction, no fix; falsifiable hypotheses; root cause and symmetric-pattern search across the repo; the regression test must be SEEN failing; "still failing" protocol. |
| `tester` | Test plans derived from invariants (adversarial first, happy path last); UI is always included; nothing is declared tested with P1-P2 cases pending. |
| `security-compliance` | Audit (AUDIT) and prevention (GUARD): OWASP with grep patterns, multi-tenant isolation/RLS, data protection (GDPR/LFPDPPP/PCI), severity only with a verified scenario, adversarial tests. |
| `verificador` | Anti-"fake green": nothing is declared done without executed, pasted evidence; evidence ladder per task type; "verified" ≠ "applied, unverified"; the ✅ is earned, not given. |
| `esquema-datos` | Schemas and migrations that don't break production: query/volume intake before designing; never float for money; expand/contract for every incompatible change; DB constraints as the final guarantee; indexes verified with EXPLAIN. |
| `features-ia` | LLM features without making things up: model names/params must be verified against current docs, never from memory; one provider-agnostic layer; tool-calling with strict contracts and per-user scoping; injection guardrails; a golden eval set gates every prompt change. |
| `entrega-git` | Atomic commits with the diff reviewed file by file (never blind `git add .`); messages that carry the why; single-topic PRs with auditable descriptions; destructive operations forbidden without an explicit request. |

## How they're built

- Each skill is a lightweight `SKILL.md` (protocol, gates, red flags) + `references/` with detailed knowledge the model loads only when needed — context cost is minimal until the skill activates.
- Content in **Spanish**, frontmatter descriptions in English (for Claude Code's matching).
- Signature doctrine: **"Violating the letter of these rules is violating their spirit"** — no task is "too simple" for the process.
- All 12 were validated with RED/GREEN tests (same neutral prompt, with and without the skill, on executable sandboxes with planted traps): 12/12 changed the model's behavior in the right direction — from forcing evidence where there was an empty "done ✅", to preventing a production-breaking migration and a cross-customer data leak in a bot.

## Manual install (no plugin)

Copy the folders under `senior-engineer/skills/` into your personal skills directory (`~/.claude/skills/`) and restart the session. You lose automatic marketplace updates.

## License

MIT — [kitoep](https://github.com/kitoep)
