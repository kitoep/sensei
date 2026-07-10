# AUDIT — Security audit playbook

Deliverable: a prioritized findings report using the exact template in the "Report template" section. It's not a chat; it's an actionable document.

## Phase 1 — Stack recon (10-15 min of exploration)

Before hunting for vulnerabilities, map the terrain. Run and note:

1. **Languages and frameworks:** read `package.json`/`requirements.txt`/`go.mod`/`pom.xml`. Note versions (an outdated version IS a potential finding).
2. **Topology:** monolith, monorepo, microservices? Which apps exist (api, web, worker, jobs)? Fixes must be symmetric across apps.
3. **Entry surface:** HTTP routes/controllers, webhooks, queues/jobs, uploads, WebSockets, CLI. Every point where external data enters is a vector.
4. **Data:** which DB? Multi-tenancy? What PII is stored? Payments? Third-party secrets?
5. **Auth:** how does it authenticate (JWT/session/OAuth)? Global guard or per-route? RBAC? 2FA?
6. **Existing security config:** look for `helmet`, `cors`, `csrf`, `throttler`/rate-limit, RLS, encryption. Note what exists and what's missing.

Record the map. If the user scoped the work (only the diff, only one module), respect it but note what was left out.

## Phase 2 — Quick threat model

Answer each in 3 lines:

- **Assets:** what's the most valuable thing that can be stolen/corrupted? (customer PII, credentials, money, isolation between tenants).
- **Actors:** who attacks? Malicious authenticated user (the most important one in SaaS), neighboring tenant, anonymous attacker, insider, compromised third parties (supply chain).
- **Surfaces:** the list from Phase 1, item 3.

This focuses the sweep: prioritize the vectors that touch the highest-value assets.

## Phase 3 — Sweep by dimension

Walk each applicable dimension (per the detected stack). For each, use the grep patterns and checks in the referenced file:

| Dimension | Reference | Guiding question |
|---|---|---|
| Multi-tenant isolation / RLS | `saas-multitenant.md` | Can tenant A see/touch tenant B's data? |
| Access control / RBAC / IDOR | `attack-surface.md` (Broken Access Control) | Can a user reach what isn't theirs by changing an ID? |
| Injection (SQL/NoSQL/command) | `attack-surface.md` | Does input reach a query/shell unparameterized? |
| Auth and sessions | `guard-rules.md` (auth) + `attack-surface.md` | Default-closed? Tokens/sessions handled correctly? |
| Secrets and encryption | `guard-rules.md` (secrets) | Any secrets in cleartext, in logs, in the repo? |
| Data protection / compliance | `data-protection.md` | PII in logs? Retention? Data-subject rights? |
| Web surface (XSS/CSRF/CORS/headers) | `attack-surface.md` | Headers, CORS, and CSRF configured correctly? |
| Supply chain | `attack-surface.md` (supply chain) | Vulnerable deps, lockfile, typosquatting? |
| LLM/bot (if applicable) | `attack-surface.md` (LLM) | Prompt injection, exfiltration via tools, does the bot make things up? |
| Security test quality | `adversarial-testing.md` | Do the tests exercise what's denied, or only confirm the happy path? |

## Phase 4 — Adversarial verification of findings (hard rule)

Before writing the report, every candidate **Critical or High** finding must pass verification:

1. **Write the concrete exploitation scenario:** exact request/input, precondition (role, state), and the observable harmful effect. If you can't write it, it isn't Critical/High — downgrade or drop it.
2. **Trace the code:** confirm with `file:line` that there is NO control mitigating it upstream or downstream (a guard, a DB constraint, a middleware). False positives come from missing the defense that's actually there.
3. **If the project is large or there are many findings, deploy verification subagents:** for each High/Critical finding, launch a subagent with the brief *"Try to REFUTE this finding: <finding + file:line>. Look for any control that mitigates it. Return CONFIRMED or REFUTED with evidence."* A finding survives only if the subagent fails to refute it. Launch the subagents in parallel (one message, multiple calls).

This avoids the most common failure mode of an audit: reporting plausible-but-false findings that burn the credibility of the whole report.

## Severity rubric (operational definitions)

Assign the highest severity that applies:

- **Critical:** exploitable by a remote attacker with low or no privileges, and the impact is cross-tenant data access, RCE, mass theft of credentials/PII, or loss of money. Verified exploitation scenario mandatory.
- **High:** exploitable by an authenticated user against data/actions that aren't theirs (IDOR, privilege escalation, auth bypass on a route), or a reusable exposed secret. Verified scenario mandatory.
- **Medium:** requires uncommon conditions, or the impact is bounded (metadata leak, missing rate-limit on a sensitive endpoint, CSRF on a non-critical action, vulnerable dependency with no confirmed exploitation path).
- **Low:** hardening/defense-in-depth with no exploitation path (missing header, verbose errors, suboptimal practice). Group them; don't enumerate one by one if trivial.

Hard rule: **without a concrete, verified exploitation scenario, a finding cannot be Critical or High.** Medium at most.

## Report template (exact output format)

```
# Security audit report — <project/diff> — <date>

## Executive summary
- Scope audited: <what was reviewed / what was left out>
- Stack: <languages, frameworks, DB, relevant versions>
- Findings: <N> Critical, <N> High, <N> Medium, <N> Low
- Top risk in 1 sentence: <...>

## Findings (ordered by severity, Critical first)

### [CRITICAL-01] <short, specific title>
- **Severity:** Critical
- **Dimension:** <e.g. Multi-tenant isolation>
- **Evidence:** `apps/api/src/x/y.ts:142` (and all relevant lines)
- **Exploitation scenario:** <exact request/input → precondition → observable harmful effect, step by step>
- **Verification:** CONFIRMED — <how it was confirmed / refuter subagent result>
- **Proposed fix:** <concrete change; snippet if applicable>
- **Symmetric fixes:** <other locations/apps with the same pattern: file:line, or "none">
- **Effort:** <S/M/L>

[repeat per finding]

## Hardening (grouped Lows)
- <compact list of defense-in-depth improvements>

## Compliance gap analysis (if applicable)
- GDPR/PCI/LFPDPPP: <missing controls, see data-protection.md>

## Checks performed (for an audit with no findings, THIS is the deliverable)
- <list of what was searched, with which pattern, in which files, and why it came back clean>
```

## Final rule of the report

If you finish with no Critical/High findings, do NOT write "the code is secure." Write "I found no Critical/High vulnerabilities in the audited scope" + the "Checks performed" section. Absence of evidence is not evidence of absence; document the effort so it's auditable.
