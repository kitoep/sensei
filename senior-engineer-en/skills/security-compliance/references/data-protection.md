# Data protection and compliance

Operational framework. Not legal theory: each control translates into something verifiable in the code. Apply the one that matches the data the project handles.

## Step 1 — Classify the data (do this first)

Before deciding controls, classify each field the system stores:

| Class | Examples | Minimum control |
|---|---|---|
| **Public** | business trade name, catalog | nothing special |
| **Internal** | aggregate metrics, non-sensitive config | authenticated access |
| **PII** (personal data) | name, email, phone, address, IP, customer ID | encryption in transit, role-based access, out of logs, subject to data-subject rights |
| **Sensitive** | health, biometrics, orientation, religion, national/tax IDs, precise geolocation, credentials | everything for PII + encryption at rest + audited access + explicit consent |
| **Secrets** | third-party tokens, API keys, passwords | encryption at rest (AES-256-GCM) or hash; never cleartext, logs, or repo |

Rule: if you don't know which class a field falls into, treat it as Sensitive until confirmed otherwise.

## GDPR (EU) — if you have or will have European users (or as the stricter standard)

1. **Legal basis:** every processing activity needs one (consent, contract, legitimate interest…). Document it per purpose.
2. **DSAR (Data Subject Access Request):** right of access, rectification, erasure ("right to be forgotten"), portability, and objection. In code: mechanisms to export a subject's data (Access), edit it (Rectification), delete/anonymize it (Erasure), stop processing it (Objection), plus **export in a structured, portable format** (JSON/CSV).
3. **Minimization and purpose limitation:** don't collect more than needed for the declared purpose. Question each new field: does the stated purpose justify it?
4. **Retention:** define and enforce retention periods. In code: a process that deletes or anonymizes data past the period. Don't keep data indefinitely "just in case."
5. **Erasure / anonymization:** when exercised, personal data is deleted or irreversibly anonymized (not reversible via a mapping table). Distinguish deletion (removes the row) from anonymization (breaks the link to the person but keeps aggregates).
6. **Privacy by design & by default:** the most protective configuration is the factory setting.
7. **Breach notification:** a procedure to notify within 72h. In code: logging/alerting that lets you detect and scope a breach.

## PCI-DSS basics — ONLY if the project touches card data

Rule number one: **do not store card data.** The correct way to comply with PCI is to stay out of its scope.

1. **Never store the full PAN or the CVV.** The CVV is never stored, not even encrypted.
2. **Delegate to a certified processor** (Stripe, Adyen, etc.): the card goes from the client to the processor (client-side tokenization), your backend only handles the token/reference. That way the PAN never touches your servers.
3. If for any reason the PAN passes through your system, you enter full PCI scope — avoid it by design.
4. In code: look for `card`, `pan`, `cvv`, `cardNumber` being stored in the DB or logs → Critical finding.

## LFPDPPP (Mexico's data protection law) — applies to any SaaS handling data of people in Mexico

Verifiable controls:

1. **Privacy notice:** must exist and be accessible to the data subject at collection time. In code: the signup/onboarding flow links or presents the notice; consent is recorded (timestamp, notice version).
2. **Consent:** for sensitive data, express consent (unchecked checkbox, affirmative action). Keep evidence.
3. **ARCO rights** (Access, Rectification, Cancellation, Objection): a mechanism must exist for the subject to exercise each. In code: an endpoint/process to export the subject's data (Access), edit it (Rectification), delete/anonymize it (Cancellation), and stop processing it (Objection).
4. **Purpose and minimization:** don't collect more data than needed for the declared purpose. Question each new field against the notice.
5. **Transfers:** if data is shared with third parties (providers, subprocessors like hosting or messaging), the notice must declare it. Document the subprocessors.
6. **Security:** proportional security measures (encryption, access control) — cross-reference guard-rules.md and saas-multitenant.md.

## Cross-cutting data controls (always)

- **PII out of logs (hard rule):** no emails, phones, names, or tokens in logs. Log opaque identifiers (`user_id`, `trace_id`) and the `tenant_id`. Also review error messages returned to the client and reports sent to third-party services (Sentry, etc.) — configure them for PII scrubbing.
- **Encryption in transit:** TLS everywhere (API, DB, queues, internal services). No plain HTTP and no DB connections without SSL in production.
- **Encryption at rest:** Sensitive and Secret data encrypted at the field level (AES-256-GCM); the rest, at least provider disk/volume encryption.
- **Retention and deletion:** period defined per data type; an automated process that deletes/anonymizes at expiry. The "anonymize button" must be irreversible.
- **Audited data access:** for Sensitive data, record who accessed what (access log, without exposing the data in the log).

## Gap analysis (format for the audit report)

When auditing compliance, deliver a table:

| Control | Framework | Exists? | Evidence / Gap |
|---|---|---|---|
| Privacy notice + consent | LFPDPPP | ✅/❌ | file:line or "missing" |
| ARCO / DSAR mechanism | LFPDPPP/GDPR | ✅/❌ | ... |
| PII out of logs | both | ✅/❌ | ... |
| Automated retention/deletion | GDPR | ✅/❌ | ... |
| No PAN/CVV stored | PCI | ✅/❌ | ... |
| Encryption at rest for sensitive/secrets | both | ✅/❌ | ... |
