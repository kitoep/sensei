---
name: security-compliance
description: Use when auditing a codebase or diff for security vulnerabilities, before writing code that touches auth/tenancy/PII/payments/secrets, when reviewing data-protection or compliance posture (GDPR, PCI-DSS, LFPDPPP), when designing multi-tenant isolation or RLS, when writing security tests, or when the user mentions hacking risk, vulnerabilities, permission segregation, data leaks, or "security review".
---

# Security & Compliance

Expert-level security playbook. You (the model reading this) do NOT improvise security judgment: **you run this playbook.** Where it says "hard rule," there are no exceptions and no nuance.

## Step 0 — Detect the mode (mandatory, before anything else)

| The user asks for... | Mode | Load |
|---|---|---|
| "audit", "review the security", "find vulnerabilities", "risk analysis", "is this safe?", reviewing an existing diff/PR/project | **AUDIT** | `references/audit-playbook.md` |
| Writing/modifying new code (feature, endpoint, table, job, integration) | **GUARD** | `references/guard-rules.md` |
| Ambiguous or both | Both | both files |

In GUARD mode the primary work is the feature; the security rules apply as you write. In AUDIT mode the work IS the security report.

## Step 1 — Detect the stack and load conditional references

Run this detection with Glob/Grep BEFORE auditing or writing code. Each signal activates an extra reference:

| Signal in the repo | How to detect it | Extra load |
|---|---|---|
| Multi-tenant SaaS (multiple accounts/businesses/orgs in one DB) | Grep: `tenant_id\|business_id\|organization_id\|account_id\|workspace_id` in schema/models | `references/saas-multitenant.md` |
| Postgres/Prisma/ORM with a relational DB | `schema.prisma`, `*.sql`, `knexfile`, `typeorm`, migrations | `references/saas-multitenant.md` (RLS section) |
| Handles personal data (end users, customers, patients, phones, emails) | Grep: `email\|phone\|address\|birth\|ssn\|tax_id` in schema/models | `references/data-protection.md` |
| Payments or card data | Grep: `stripe\|card\|payment\|pan\|cvv\|checkout` | `references/data-protection.md` (PCI section) |
| Bot, LLM, agent, tool-calling, chat | Grep: `openai\|anthropic\|llm\|prompt\|langchain\|tool_call\|whatsapp.*bot` | `references/attack-surface.md` (LLM section) |
| Public API / webhooks / web frontend | controllers, routes, `webhook`, `cors` | `references/attack-surface.md` |
| Security tests will be written or reviewed | whenever a test suite exists | `references/adversarial-testing.md` |

If you can't determine a signal, assume it DOES apply (default-closed: over-loading a reference is cheap; skipping a vector is an incident).

## Step 2 — Run the mode

- **AUDIT** → follow the phases in `audit-playbook.md` in order. The deliverable is the report using that file's exact template. Hard rule: no Critical/High finding without a concrete exploitation scenario verified in the code.
- **GUARD** → apply the rules in `guard-rules.md` to every line you write. Before calling the feature done, run that file's "Definition of done checklist" and the applicable tests from `adversarial-testing.md`.

## Cross-cutting hard rules (both modes, always)

1. **Identity and tenant NEVER come from the client.** Not in body, query, params, or custom headers. Only from a verified token or a server-side mapping.
2. **The UI is never enforcement.** Every access-control/validation check runs on the server; the frontend only hides.
3. **Default-closed.** Global auth by default, exceptions explicit and annotated. When unsure whether to allow or deny: deny.
4. **Secrets encrypted at rest, never in logs, never in the repo.** Third-party tokens, API keys, credentials: AES-256-GCM or a vault; no PII or secrets in logs.
5. **Never call anything "secure" or "done" without the matching adversarial test run** (see `adversarial-testing.md`).

## Common operator mistakes (yours)

| Temptation | Correction |
|---|---|
| "The diff is small, no need for the playbook" | Incidents live in small diffs. Run the playbook proportional to the diff, but run it. |
| "I'll report everything I found" (20 Low lint findings) | The value is in prioritizing: 3 actionable Criticals > 30 trivial ones. Use the severity rubric. |
| "This framework already protects it by default" | Verify the real config (version, flags, registered middleware), not the framework's reputation. |
| "The test passes, so it's protected" | Apply the litmus test: if you remove the enforcement, does the test stay green? If yes, there's no test. |
| "I found nothing, the code is fine" | An audit with no findings needs evidence of what you looked for and where. List the checks you ran. |
