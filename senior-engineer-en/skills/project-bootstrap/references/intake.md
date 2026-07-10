# Intake — the questions before the first line of code

Goal: get the answers that CHANGE the architecture. This isn't a bureaucratic form: every question unblocks a concrete decision. If an answer wouldn't change anything you're about to build, don't ask it.

## How to run it

1. Ask the questions **one at a time** or in groups of at most 3 with multiple-choice options. Users bail on long questionnaires.
2. Always give your recommendation with the why ("I recommend X because..."), not a neutral menu.
3. Record each answer in `docs/00_intake.md` as it comes in. At the end, show the full document to the user and ask for explicit confirmation.
4. If the user says "I don't know," propose the default that's cheapest to change later and mark it `[ASSUMPTION]` in the doc — assumptions get revisited at the first milestone.
5. If you spot a contradiction between answers (e.g. "$0 budget" + "high availability"), flag it on the spot, don't resolve it silently.

## Questionnaire

### Block A — Purpose (defines the scope)

**1. What problem does it solve and for whom, in one paragraph?**
Why: if it doesn't fit in a paragraph, the project isn't defined and everything else is speculation.
Unblocks: the criterion for saying NO to features (anything that doesn't serve that paragraph gets cut).

**2. What does success look like in 3 months, in numbers?** (active users, paying businesses, hours saved — something measurable)
Why: "it works" isn't a criterion; without a number there's no way to prioritize.
Unblocks: which milestone comes first and what's v2.

**3. What will NOT be built in v1?** (explicit negative scope — ask for at least 5 items)
Why: negative scope is the only defense against silent creep. Anything not on the exclusion list, someone will ask for.
Unblocks: the "DO NOT implement" section of CLAUDE.md.

### Block B — Constraints (define the stack)

**4. Monthly infrastructure budget?** (ranges: $0 / <$25 / <$100 / <$500 / doesn't matter)
Why: budget decides hosting and architecture more than any technical preference. At <$25 there's no Kubernetes and no multi-region, and that's fine.
Unblocks: hosting, managed vs self-run database, and how much redundancy is honest.

**5. Who's going to maintain this?** (just the user + agents / a small team / a team that will grow)
Why: the complexity ceiling is set by the maintainer, not the ambition. A stack the maintainer doesn't know is debt from day 1.
Unblocks: how many moving parts are allowed (modular monolith or services?) and how much documentation is mandatory.

**6. Deadline? What happens if it ships a month late?** (nothing / lose a customer / project dies)
Why: the answer calibrates how much technical risk you can take. "Project dies" = zero experiments, all boring proven stack.
Unblocks: how aggressive the milestone plan can be.

**7. Realistic concurrent users in year 1?** (dozens / hundreds / thousands / no idea)
Why: over-engineering for imaginary scale kills more projects than lack of scale. Hundreds of concurrent users a monolith on Postgres handles without breaking a sweat.
Unblocks: explicit permission to NOT build for scale that doesn't exist.

### Block C — Nature of the system (irreversible decisions)

**8. Is it multi-tenant?** — will more than one business/organization use the same instance with data that must NEVER cross?
Why: multi-tenancy is NOT retrofittable. Adding it later means touching every table, every query, and every test. It's the most irreversible decision of all.
Unblocks: if yes → `tenant_id` across the whole schema + DB-level isolation from the first table (see foundations.md #10).

**9. Does it handle personal or sensitive data? What jurisdiction are the data subjects in?** (Mexico/LFPDPPP, EU/GDPR, health, payments)
Why: compliance isn't a feature you bolt on: it defines what gets encrypted, what gets logged, what can be deleted, and where it's hosted.
Unblocks: encryption at rest, PII-free logging policy, deletion/anonymization mechanism, and whether the `security-compliance` skill applies from day 1 (if installed).

**10. What external integrations does the heart of the product depend on?** (third-party APIs, payment gateways, WhatsApp, platform approvals)
Why: the external integration is the risk you don't control — quotas, approvals, API changes. If the product depends on one, that integration gets validated BEFORE you build around it.
Unblocks: the first spike of the plan (see walking-skeleton.md, ordering by risk).

**11. Are there users with login? Are there distinct roles?**
Why: auth and permissions are foundation, not feature. Adding roles after "everyone can do everything" means auditing every existing endpoint.
Unblocks: default-closed auth + role model from the skeleton onward (foundations.md #9).

### Block D — Risk

**12. What's the one thing that, if it turns out false, invalidates the entire project?**
Why: every project has a central bet (the API allows it / people will pay / the model can do it). Naming it forces you to test it first, cheaply.
Unblocks: the real milestone 1 — which is almost never the one the user had in mind.

## Phase output

`docs/00_intake.md` with the 12 answers (assumptions marked), confirmed by the user. Only then move to `references/foundations.md`.
