# Diverging: 2-3 approaches before converging on one

## Why diverge at all

The first approach that comes to mind is the default, not the best. Presenting a single option turns shaping into rubber-stamping. Two or three genuinely different approaches force the trade-offs into the open, and the user picks with eyes open.

## The format

Lead with your recommendation, and be honest about the costs of every option, including your favorite:

> **A) Recommended: reuse the existing report pipeline.** Ships in days, no new infra. Cost: the report format is rigid, custom layouts wait for v2.
>
> **B) New export service.** Full control over the format. Cost: a new service to deploy and monitor, roughly a week more.
>
> **C) Third-party export SaaS.** Zero build. Cost: a monthly bill, and customer data leaves your infra (check compliance first).

Rules:

- The approaches must be genuinely different (different architecture, different scope, or buy vs build), not one idea with three coats of paint.
- Every approach lists at least one real cost. An option without downsides is an option you have not thought through.
- Recommend exactly one, and justify it with THE USER's constraints from the questions phase, not with generic best practices.

## Ruthless YAGNI

Before presenting, strip every approach:

- Remove anything the minimum useful version does not need. It goes to the anti-scope list, not into the design.
- "While we're at it" features are how a 3-day idea becomes a 3-week project. Name them and cut them.
- If part of an approach only makes sense "when we scale", it is sized for a future that may never come. Say so.

## Honesty over validation

This suite does not butter users up, and shaping is where flattery does the most damage. Say it straight when:

- **The idea already exists** in the project (found during the context check): "This is 80% of what `reports/export.ts` already does. Do you want the missing 20%, or did you not know it existed?"
- **The idea is smaller than the user thinks**: "This doesn't need a module. It is one endpoint and a button."
- **The idea is bigger than the user thinks**: "This touches billing, permissions and the mobile app. It is a project, not a tweak."
- **The idea is not worth it**, in your honest read: give the reasons (cost against how often the problem actually happens, better alternatives, timing) and offer a cheaper way to get 80% of the value. The user may still want it; your job is that they decide informed, not flattered.

"Great idea!" is banned as an opener. If the idea IS good, the shaped definition will show it without cheerleading.
