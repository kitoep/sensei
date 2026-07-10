# Attack surface — OWASP, operationalized

Not a theoretical list. For each vector: **how to find it** (concrete pattern) and **what the fix looks like**. Walk the ones that apply to the detected stack.

## A01 — Broken Access Control (the #1 in SaaS: IDOR / BOLA)

**What it is:** a user reaches data/actions that aren't theirs by changing an identifier.

**How to find it:**
- Grep routes with IDs: `:id`, `/{id}`, `req.params`, `req.query.*id`.
- For each, check: is it verified that the resource belongs to the token's tenant/user BEFORE operating? Look for `findFirst({ where: { id, tenantId } })` vs. the dangerous `findUnique({ where: { id } })` without tenant.
- Grep write actions (`update`, `delete`, `destroy`) that take an ID from the client.

**Fix:** every resource access by ID also filters by the tenant/owner derived from the token. Return 404 (not 403) if it doesn't belong. See saas-multitenant.md.

## A03 — Injection (SQL / NoSQL / Command / Path)

**How to find it:**
- Raw SQL with interpolation: grep `` `SELECT ${ `` , `"SELECT " +`, `query(` with concatenation, `raw(`, `$queryRawUnsafe`.
- NoSQL: client objects passed straight into filters (`find(req.body)`), `$where`/`$regex` operators with raw input.
- Command: `exec(`, `execSync(`, `spawn(` with strings built from input.
- Path traversal: `readFile`/`join` with a client-supplied filename that isn't sanitized (`../`).

**Fix:** parameters/bindings (`$1`, `?`), never interpolation. For NoSQL, whitelist fields and cast types. For commands, arg arrays and avoid the shell. For paths, resolve and verify it stays inside the allowed directory.

## A02/A04 — Crypto and secrets

**How to find it:**
- Hardcoded secrets: grep `api[_-]?key`, `secret`, `password`, `token`, `private[_-]?key`, strings like `sk_live`, `AKIA`, literal JWTs. Check git history too (`git log -p | grep`).
- Weak algorithms: `md5`, `sha1` for passwords, `Math.random()` for security tokens/IDs, `createCipher` (deprecated) instead of `createCipheriv`.
- Third-party tokens stored in cleartext in the DB.

**Fix:** secrets from env/vault; rotate anything exposed. Passwords with bcrypt/argon2. Tokens/nonces with a CSPRNG (`crypto.randomBytes`). Field encryption with AES-256-GCM (`createCipheriv`). See guard-rules.md §5.

## A05 — Security misconfiguration

**How to find it:**
- **CORS:** grep `cors(`, `Access-Control-Allow-Origin`. Red flag: `origin: '*'` or `origin: true` with `credentials: true`. It must be an origin allowlist.
- **Headers:** is there `helmet` or equivalent? Check CSP, HSTS, X-Content-Type-Options, X-Frame-Options.
- **Verbose errors:** stack traces or DB messages returned to the client in production.
- **Debug/flags:** `debug: true`, misconfigured `NODE_ENV`, introspection endpoints (GraphQL introspection, `/actuator`, `/debug`) left open.
- **Defaults:** default credentials, exposed admin ports.

**Fix:** CORS with an explicit allowlist; helmet configured; generic errors to the client + detail only in logs; disable debug/introspection in prod.

## A07 — Authentication failures

**How to find it:**
- Is the auth guard global (default-closed) or do you have to remember to add it per route? The latter is fragile.
- Grep routes marked public/`@Public`/`skipAuth` — is each one justified?
- Rate limit on login/signup/reset/OTP: grep `throttle`, `rateLimit`, `RateLimiter`. Missing on login = brute force.
- Session handling: long tokens with no expiry? Refresh doesn't rotate? Logout doesn't revoke?

**Fix:** global guard + justified exceptions; rate limit on auth surfaces; short sessions + rotating revocable refresh; 2FA for admins. See saas-multitenant.md (segregation).

## CSRF

**How to find it:** apps with cookie sessions. Is there CSRF protection (double-submit token, SameSite)? APIs with a Bearer token in a header don't need CSRF (the token doesn't travel automatically), but those that accept a session cookie do.

**Fix:** CSRF double-submit with timing-safe comparison, or SameSite=Strict/Lax + origin verification. Note for tests: a CSRF 403 can mask a permissions test — use Bearer to isolate (see adversarial-testing.md).

## XSS

**How to find it:** grep `dangerouslySetInnerHTML`, `innerHTML`, `v-html`, `bypassSecurityTrust`, template strings that inject input into HTML without escaping.

**Fix:** escape by default (modern frameworks do); sanitize allowed HTML with a library (DOMPurify); CSP as a second barrier.

## SSRF

**How to find it:** grep `fetch(`, `axios(`, `http.get(`, `request(` where the URL comes (fully or partly) from the client. Common in: configurable webhooks, "import from URL", thumbnail generation, integrations.

**Fix:** allowlist domains/schemes; block private/loopback/metadata IPs (`169.254.169.254`, `10.*`, `127.*`, `localhost`); resolve DNS and validate the resulting IP; no automatic redirects to unvalidated destinations.

## A06 — Supply chain (dependencies)

**How to find it:**
- Run `npm audit` / `pip-audit` / `govulncheck`. Critical/High = finding.
- Is there a committed lockfile (`package-lock.json`, `poetry.lock`)? No lockfile = non-reproducible builds.
- Typosquatting: check for suspicious or little-known package names with few downloads.
- Postinstall scripts in untrusted dependencies.

**Fix:** `npm audit` in CI that blocks on critical; committed lockfile; pin versions; review new dependencies before adding them.

## LLM / bot risks (if the project uses AI)

Specific, often-ignored vectors:

1. **Prompt injection:** user input (or retrieved content: emails, docs, messages) that contains instructions hijacking the model. Look for where untrusted text enters the prompt.
   - **Fix:** separate system instructions from user data; treat ALL input and retrieved content as untrusted; don't give the model capabilities it shouldn't run without confirmation; validate/whitelist outputs that trigger actions.
2. **Exfiltration via tool-calling:** a model with tool access can be induced to read one tenant's data and send it to another, or to an external channel.
   - **Fix:** tools apply the SAME tenant/permission scoping as the rest of the backend (the model is not a privileged actor); parameterize tools by the context's tenant, never by whatever the model "decides"; log and limit tools that send data out.
3. **The bot doesn't make things up (grounding):** the model answers ONLY with retrieved evidence (KB, tool-calling to the DB). Data like prices/services/availability ALWAYS via tool-calling to the source of truth, never generated by the model. No evidence → "I don't have that information" + offer a handoff.
4. **Human mode turns the bot off:** when a conversation is with a human, the bot does NOT respond. Verify that state exists and is respected (a test that in HUMAN mode the bot emits no message).
5. **Topic guardrails:** the bot only discusses the business's topics; it refuses to go out of scope.
6. **Spend/abuse limits:** per-tenant rate limit on LLM calls; client input doesn't control the model, tokens, or cost without a cap.

## Output of this dimension

Each vector found is recorded as a finding with the audit-playbook.md template, with its grep pattern as part of the evidence and its concrete fix.
