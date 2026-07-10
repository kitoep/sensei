# Archetypes by dominant condition

Four profiles. Choose ONE based on the intake's dominant force; the rest serve as the evolution path. Each profile gives the reference stack per layer with selection criteria (not closed brands: the concrete provider is chosen with the criteria at the end). Costs in USD/month, 2025-2026 ranges.

---

## A. Minimum spend / validation (~$0–25/month)

**When it applies:** an idea with no paying users, ~$0 budget, a single developer, the dominant thing is discovering whether anyone wants the product.
**When NOT:** there are already paying customers or real third-party sensitive data — move up to profile B; the free tier gives no serious backups or SLA.

| Layer | Reference | Criterion |
|---|---|---|
| Hosting | 1 PaaS service with free/hobby tier (web+API together) | Deploy by git push, HTTPS and domain included, zero server config |
| DB | Small managed Postgres (free tier or ~$5) | Automatic backup even if only daily; NEVER SQLite on the PaaS's ephemeral disk |
| Backend | Monolith in the language the team knows | Iteration speed; no separate services |
| Front | Server-rendered or SPA served by the same service | A single deploy |
| Jobs | PaaS cron or a task table + tick every minute | No dedicated queue; the seams (logic outside the controller) prepare the future queue |
| Storage | S3-compatible bucket free tier | Never on the PaaS disk (ephemeral) |
| Observability | PaaS logs + free uptime monitor | Enough for validation |

**Accepted risk (state it):** minutes of downtime without warning, cold starts, no support.
**Migration to B:** the code doesn't change (same monolith); the PaaS plan changes, the DB to a plan with point-in-time recovery, and the worker gets split out. ~100% reused.

---

## B. Small SaaS in production (~$25–150/month)

**When it applies:** real paying customers, third-party data (PII), 1-3 developers, the dominant thing is shipping features without fires. **The default profile when in doubt.**
**When NOT:** if nobody pays yet (use A); if one hour of downtime loses critical transactions (use C).

| Layer | Reference | Criterion |
|---|---|---|
| Hosting | Railway/Render/Fly-style PaaS: web service + separate worker | Separate API/worker processes; automatic restart; deploy by push with rollback |
| DB | Managed Postgres ~$10-30 with point-in-time recovery | PITR is the line that separates "hobby" from "production"; verify retention ≥7 days |
| Backend | Modular monolith (layers: controller→service→repo) | One deployable; module boundaries are in code, not on the network |
| Front | Same repo (monorepo), own deploy or on the PaaS | Share types/contracts between front and API |
| Jobs | Queue backed by managed Redis (~$5-10) or by the DB itself | Mandatory if there are webhooks/retries/sends; idempotent jobs from day 1 |
| Storage | S3-compatible bucket with versioning | Customer data: versioning + retention policy |
| Observability | Structured logs (JSON + trace_id) + error tracking (free tier) + uptime + failed-backup alert | The alert that matters most: failed backup and 5xx errors |

**Accepted risk:** single-instance per service → deploys with seconds of blip, and a region outage = your outage (rare; state it).
**Migration to C:** duplicate instances behind the PaaS load balancer, DB replica. Everything is reused if the seams exist (sessions out of memory, idempotent jobs, no local disk state).

---

## C. High availability required (~$150–800/month)

**When it applies:** one hour of downtime loses money or customers measurably (intake #4 with a concrete consequence), or there's a contractual SLA.
**When NOT (common paranoia):** "it'd look bad if it went down" is not HA; profile B with a good restart gives ~99.5% which covers most cases. Demand the measurable consequence before accepting this profile.

| Layer | Reference | Criterion |
|---|---|---|
| Hosting | ≥2 instances per service behind a load balancer, real health checks (that touch the DB, not just `200 OK`) | Rolling deploy with no downtime; one instance dies and nobody notices |
| DB | Managed Postgres with replica and automatic failover | Test the failover BEFORE you need it (game day) |
| Backend/Front | Same as B — HA doesn't change the code, it changes the topology | Stateless mandatory: sessions in DB/Redis, zero state on local disk |
| Jobs | Queue with managed Redis with replica; workers ≥2 | Idempotency stops being a best practice and becomes correctness |
| Storage | Bucket with provider replication | — |
| Observability | Metrics with alerts (p95, error rate, queue depth), defined on-call, status page | HA without alerts is decorative HA: nobody finds out the failover happened |

**Multi-region: almost always paranoia.** It's only justified with users on different continents suffering measured latency, or a contractual/regulatory residency requirement. A full-region outage at a large provider is so rare that a documented restore plan (passive DR: backups in another region + runbook) covers the risk at 1/10 the cost of active multi-region.
**Migration to D:** almost there already — D adds elasticity, not redundancy.

---

## D. Traffic spikes (~variable, $100–1000+/month)

**When it applies:** traffic with spikes of ≥10× normal, predictable or sudden (campaigns, seasons, events, virality). The dominant force is absorbing the spike without paying for the spike 24h a day.
**When NOT:** sustained gradual growth is not "spikes" — it's profile B/C scaling the plan.

| Layer | Reference | Criterion |
|---|---|---|
| Hosting | Horizontal autoscaling (PaaS with autoscale or managed containers) | Scale by metric (CPU/RPS/queue depth), with a low minimum and a bounded maximum (protects the wallet) |
| DB | The piece that does NOT autoscale: connection pooler (pgbouncer or the provider's) mandatory + plan sized to the write peak | Connections kill Postgres before CPU does; the pooler is the first thing |
| Backend | Same, but EVERYTHING deferrable is decoupled to the queue | The synchronous request does the minimum; the queue absorbs the spike and workers drain at their own pace |
| Front | Assets behind a CDN, public pages cached/static | The spike of curious visitors must not touch your API |
| Jobs | The queue IS the buffer: workers autoscale by queue depth | Explicit backpressure: what degrades first (e.g. reminders get delayed, bookings don't) |
| Observability | Same as C + live saturation dashboards + spend limits/billing alerts | Autoscaling with no cap turns an attack/bug into an invoice |

**Key decision — what to decouple:** classify each operation as (1) synchronous untouchable (login, booking, payment), (2) deferrable by seconds (notifications, outbound webhooks), (3) deferrable by minutes (reports, emails). Only (1) sizes the API; (2) and (3) live in the queue.

---

## Criteria for choosing the concrete provider (any profile)

1. **Predictable pricing** — an estimable monthly bill; beware per-request serverless for constant loads (cheap to start, expensive sustained).
2. **Cheap exit** — standard Postgres + containers + S3-compatible = you move in a weekend. Proprietary services (exclusive DB, functions with their own SDK) = lock-in that's paid for with a condition that justifies it.
3. **PITR and verifiable backups** on the managed DB — restore a test one before trusting it.
4. **Region close to your users** (and compatible with data residency if intake #5 requires it).
5. **Track record** — public status page with history; without a history, don't host production there.
6. **The one the team already knows wins ties.**
