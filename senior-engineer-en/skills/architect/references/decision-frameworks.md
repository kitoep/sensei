# Decision frameworks — principles with operating rules

These principles apply in order when two options compete. Each has its executable rule and its legitimate exception. If you recommend against a principle, state which one and why the user's condition warrants it.

## 1. Boring first

**Rule:** technology with ≥5 years in mass production > new technology, as long as it covers the requirement. Innovation is spent on the PRODUCT, not on the infra.
**Test:** can you find 3 postmortems/operations guides from other teams using this component? If not, it's too new for a small team.
**Legitimate exception:** the new tech eliminates an entire category of work (e.g. a BaaS that saves building auth+API+realtime for a validation MVP).

## 2. Modular monolith first

**Rule:** a single deployable with clearly-bounded modules (layers, domains separated in the code). Microservices only with DEMONSTRATED pain: teams blocking each other, a module with a radically different load profile, or a real independent-scaling requirement — never for anticipated pain.
**Nuance:** separating **API** and **jobs worker** (same codebase, two processes) is NOT microservices; it's hygiene: it protects API latency from heavy jobs and it's cheap from day 1 if there are jobs (reminders, webhooks, sends).
**Test for splitting something out:** can you name the metric or the incident that demands it? Without a metric, nothing gets split.

## 3. Managed > self-hosted (as long as cost allows)

**Rule:** DB, queues, storage, email — managed by default. Ops time is the most expensive resource for a small team: 4 hours/month of maintenance is worth more than the price difference.
**Breaking point:** when the managed service's cost exceeds ~3-5× the equivalent self-hosted AND the team already has real ops capacity (someone whose job includes operating infra), re-evaluate piece by piece.
**Never self-host** (small team): the production database, transactional email, anything with data you can't lose.

## 4. Postgres as default

**Rule:** the relational database (Postgres is the default for RLS, JSONB, extensions and ecosystem) until a specific, measurable requirement rules it out. It covers: relational, documents (JSONB), basic full-text search, simple queues, geodata (PostGIS).
**Valid rule-outs:** deep graphs as a core operation, time series at millions of points/day scale, sub-millisecond cache (Redis complements, doesn't replace), advanced search with relevance (when native full-text falls short — not before).
**Anti-pattern:** adding a second database "just in case". Each additional engine is duplicated backups, monitoring and expertise.

## 5. Don't add pieces until a metric asks for it — but leave the seams

**Rule:** queues, cache, CDN, read replicas: they do NOT enter the initial architecture unless the intake shows the need (e.g. integrations with retries → queue from day 1). Each piece is added when a metric crosses a threshold, not when "it feels slow".
**The seams ARE paid for today (they're almost free):** data access behind repositories (enables cache later), domain logic outside the controllers (enables moving to a worker later), IDs and contracts that don't assume a single server, simple feature flags, dates in UTC.
**Reference thresholds:** cache → when the same heavy query dominates p95; queue → when a request makes >1 external call or >2s of work; CDN → when assets/geographic traffic show it; replicas → when reads saturate the primary (visible in metrics, not in fear).

## 6. Optimize for cost of change, not for the end state

**Rule:** the central design question: *"how expensive is it to change this decision in 6 months?"* — not *"does it hold 1M users?"*. The answer to intake #9 (what's most uncertain about the business) signals which parts must be cheap to change; invest flexibility THERE and nowhere else.
**Anti-pattern:** abstracting everything "just in case" (the ORM-agnostic layer, the preventive multi-cloud). Generic flexibility is cost without benefit; flexibility directed by the declared uncertainty is investment.

## 7. Reversibility rule (how much analysis each decision deserves)

**Irreversible or very expensive to reverse** — deserve deep analysis, written comparison and consulting the user:
- The primary database and its model (single-tenant vs multi-tenant with RLS vs DB-per-tenant)
- The backend language/runtime
- The identity/auth model (own vs provider)
- Region/data residency (if there's compliance)

**Reversible** — decide fast with the reasonable default and move on; changing them later costs days, not months:
- UI library, email provider, logging/monitoring tool, CDN, CSS framework, hosting provider (if the code doesn't use proprietary services)

**Rule:** analysis time must be proportional to the cost of reversing. Debating the email provider for 2 days is as bad as choosing the DB in 5 minutes.

## Resolving conflicts between principles

When two principles clash (e.g. managed [3] vs minimum budget [intake #2]): **the dominant force identified in the flow wins**. E.g.: dominant force = minimum cost → more operational risk is accepted and documented ("no replica: a DB outage means restoring from backup, ~1h max loss — accepted for budget").
