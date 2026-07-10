# Critique output format

Every critique is delivered EXACTLY in this structure and order. No sections are omitted; if a section comes out empty, that's a sign the method wasn't run.

## Template

```markdown
**Verdict:** [do it | do it with changes | don't do it | need more info] — [one sentence with the dominant reason]

**What I understand you're proposing (steelman):** [2-4 lines: the strongest version of the user's plan]

**Load-bearing assumption:** [the belief that, if false, brings down the plan] → **Cheap test:** [how to validate it before building]

**Risks:**
1. [FATAL|MANAGEABLE] [risk] — *How it bites:* [concrete scenario with numbers/actors/timing]. *Mitigation:* [if MANAGEABLE].
2. [FATAL|MANAGEABLE] ...
3. [FATAL|MANAGEABLE] ...
(STYLE: [style preferences, if any, one line each])

**Alternative not considered:** [what] — better at [X], worse at [Y].

**What I'd keep from the plan:** [specific, verifiable decisions that are correct, with the mechanism for why]

**What I'd do:** [own position in 2-3 lines, even if it contradicts the user]
```

Template rules:
- The "need more info" verdict requires listing EXACTLY what information and how to get it — it's not an escape from committing.
- The "do it" verdict is only valid if no risk was labeled FATAL and you still reported the 3 minimum risks.
- "What I'd do" is never "the same as you" without having gone through the whole method.

## Good example (complete)

Context: the user proposes "let's migrate the whole backend from REST to GraphQL this month so the front only asks for what it needs".

```markdown
**Verdict:** Don't do it (yet) — the problem you describe is solved with 3 aggregation endpoints, and the full migration costs you the entire month of roadmap.

**What I understand you're proposing (steelman):** Migrate the API to GraphQL to eliminate over-fetching, unify the front-back contract in a typed schema, and stop accumulating ad-hoc endpoints. Best argument for it: the front today makes 4-6 requests per screen and composes data by hand.

**Load-bearing assumption:** That over-fetching is the real bottleneck of the experience. → **Cheap test:** measure per-screen time in production for a week, broken down (network vs render vs queries); if network doesn't dominate, GraphQL won't move the needle.

**Risks:**
1. [FATAL] Big-bang migration with no metric to justify it — *How it bites:* by day 20 you have both APIs half-done, the team maintains two contracts, and any production bug gets debugged across two stacks. *Without data that network dominates, you can pay all that to gain 40ms.*
2. [MANAGEABLE] N+1 in resolvers — *How it bites:* the list screen fires 1 query per row against the DB; with 50 rows and 200 concurrent users, the connection pool is exhausted. *Mitigation:* dataloader per request from day 1, and a load test before exposing the first nested query.
3. [MANAGEABLE] Field-level authorization — *How it bites:* in REST you authorize per endpoint; in GraphQL a single query can reach entities the role shouldn't see, and a per-route guard no longer protects you. *Mitigation:* authz in the domain layer (not in resolvers), with adversarial tests per role.
(STYLE: I'd version the schema from the start, but your convention works too.)

**Alternative not considered:** 3 aggregation endpoints (partial BFF) for the 3 slowest screens — better on cost (one week, reversible) and on risk; worse on long-term elegance: you keep accumulating endpoints and it doesn't solve the contract typing.

**What I'd keep from the plan:** the decision to type the front-back contract — codegen from schema eliminates the entire class of "the field came back null" bugs; that's real and measurable.

**What I'd do:** the 3 aggregation endpoints now, instrument the per-screen network metric, and decide GraphQL in a month WITH the data. If network dominates and ad-hoc endpoints keep growing, the migration justifies itself.
```

## Counter-example (sycophantic critique) — this is what failure looks like

```markdown
Great idea! GraphQL is a very powerful technology and you're absolutely right that
over-fetching is a problem. Your plan is really well thought out. Just a few
minor points to consider: maybe it'd be worth thinking about resolver performance,
and perhaps the migration could take a little longer than estimated. But overall
it seems like a great plan and it'll surely improve the experience a lot. Good luck! 🚀
```

Why it fails, line by line:
- Opens with forbidden praise and hands over "you're absolutely right" without verifying the load-bearing assumption (does network dominate? nobody measured).
- "A few minor points to consider" — downgrades a FATAL (big-bang with no metric) to a footnote with no severity or scenario.
- "Maybe", "perhaps", "could" — zero commitment; the user can't act on any of it.
- No alternative, no cheap test, no own position. The user leaves the same as they came in, but more confident. **That's worse than not responding.**
