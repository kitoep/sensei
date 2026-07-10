# Walking skeleton — deploy before features

## What it is

The **walking skeleton** is the thinnest possible version of the system that runs the ENTIRE path end to end and is DEPLOYED: request → auth → minimal logic → migrated database → response, with CI/CD that ships it automatically. It does almost nothing — but it does it in the real environment.

**Why it works:** integration problems (config, secrets, migrations, networking, hosting permissions) are the ones that wreck schedules, and they only show up when you integrate. The skeleton forces them to appear in week 1, when fixing them takes minutes, instead of week 12, when there are 40 features on top.

## Mandatory Milestone 0 (before any feature)

Checklist — milestone 0 is done when ALL of this is true:

1. [ ] The repo has green CI: typecheck + tests + build on every PR.
2. [ ] `docker compose up` (or equivalent) brings up the full local environment with real dependencies.
3. [ ] There is ONE applied migration (even a trivial table) and CI migrates from scratch.
4. [ ] There is ONE end-to-end endpoint working: it hits the DB and responds (with auth if the system has it — and it has it, default-closed, from now on).
5. [ ] That endpoint is DEPLOYED to a URL-accessible environment, shipped by the pipeline (not by hand).
6. [ ] The deployed environment's secrets live in the hosting provider's secret manager, not in the repo.
7. [ ] There is ONE integration test exercising that path against the real DB, running in CI.

If point 5 fails, the milestone isn't done. "Runs on my machine" isn't a walking skeleton; it's a skeleton lying down.

## How to order the first 5 milestones: by risk, not by ease

Golden rule: **whatever can invalidate the project gets tested first, in its cheapest version.** The natural (and wrong) order is to start with the easy, known stuff; the correct order is to start with what you don't know will work.

Procedure:

1. Take the answer to intake question 12 ("what would invalidate the project?") and the integrations from question 10.
2. For each risk, define the **cheapest spike that confirms or kills it** (e.g. "send and receive ONE real message through the WhatsApp API with an approved account", "charge a real $1 through the gateway", "the model extracts the right field on 20 real cases").
3. Order it: Milestone 0 = walking skeleton → Milestones 1–2 = spikes for the risks that invalidate → Milestones 3–5 = the heart of the product on top of what's already validated.
4. The reversible and known stuff (CRUDs, admin screens, visual polish) goes LAST. You can always do it; it never invalidates anything.

Sanity check on the plan: if your milestone 1 is something you know for certain you can build, the order is wrong — you're starting with the comfortable, not the risky.

## Anti-patterns (with their real consequence)

| Anti-pattern | Why it's tempting | How it ends |
|---|---|---|
| Start with the easy CRUD | Fast visible progress | Week 8: the perfect CRUD orbits a heart that was never validated and doesn't work |
| Deploy "at the end" | "Get it working locally first" | The deploy surfaces broken config/secrets/migrations with the whole system on top; days of fixes under pressure |
| Features with no tests "for now" | "Moving fast, I'll add them later" | Nobody adds them; the first refactor breaks 3 things silently and trust in the code is gone |
| "We'll wire it up later" (deferred integration) | Each piece moves in parallel | The pieces don't fit: different contracts, different assumptions, weeks of glue |
| Validate the risk with mocks | The mock always works | The real API has quotas, approvals, and latencies the mock never showed — and the project depended on it |
| Build for imaginary scale | "Better to leave it ready to grow" | Months on infra for thousands of users who don't exist yet, zero product validation |

## Phase output

Milestone 0 completed (checklist above, with evidence: deployed URL + green CI) and a tracker with milestones 1–5 ordered by risk, each with its own measurable done criterion.
