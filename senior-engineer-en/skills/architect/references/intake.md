# Intake — the interview before recommending

Goal: turn "I have an idea" into measurable conditions. Ask the questions one at a time or in blocks of 2-3 (not all 9 at once). Record every answer; the final recommendation will cite them.

**Golden rule:** if the user doesn't know an answer, assume the SMALL scenario and make it explicit in the recommendation ("assumed <1,000 users/month because there's no data"). Never assume the large scenario: over-provisioning costs real money today; under-provisioning costs a migration tomorrow — and migrating from something simple is cheaper than paying for phantom infra for 12 months.

## The 9 questions (each with its why)

1. **How many users/how much traffic do you REALISTICALLY expect in the next 6 months?** (not the 5-year dream)
   *Why:* it sizes everything. The difference between 100 and 100,000 users/month changes the architecture; the difference between 100,000 and "millions someday" changes nothing today. If the answer is a fantasy ("thousands of businesses"), ask again: *"how many customers do you have committed or in the pipeline TODAY?"*.

2. **What's your monthly infrastructure budget?** (a number, not "not much")
   *Why:* it's the most honest constraint. $10/month and $200/month produce different recommendations and both are valid. Ask for a number or a range; if there isn't one, assume the minimum.

3. **Who operates this?** (just you? is there someone doing devops/on-call?)
   *Why:* it determines managed vs self-hosted. A "cheap" self-run server costs hours of patching, backups, and waking up at 3 AM. If a single person operates it and also develops: managed is almost mandatory.

4. **What happens if the system goes down for 1 hour on a Tuesday at 11 AM?** (do you lose money? customers? nothing?)
   *Why:* it defines the availability budget. "It'd be embarrassing" → single-instance with good automatic restart is enough. "I lose customer transactions/appointments" → redundancy and health checks. "Someone could get hurt" → a different design league entirely. Most people overestimate this: ask again with the concrete scenario.

5. **Data: how much volume, how sensitive, is there compliance?** (PII, health, payments, minors)
   *Why:* sensitive data imposes non-negotiable requirements (encryption, data residency, multi-tenant RLS, auditing) that constrain providers and raise the cost floor. Catching it late = re-architecture.

6. **Do you need real time?** (chat, push notifications, live dashboards — or is "every 30 seconds" enough?)
   *Why:* genuine real time (websockets, queues) adds pieces with operational cost. 80% of "I need real time" is solved with short polling, which adds no infra. Distinguish which one it is.

7. **Which integrations are mandatory?** (WhatsApp, payment gateway, ERP, third-party APIs)
   *Why:* integrations impose technical constraints (public webhooks → you need a stable URL; retry jobs → you need a worker/queue) and sometimes dictate region or provider.

8. **What technologies does the team already know?** (languages, frameworks, DBs they've worked with seriously)
   *Why:* the known stack ships in weeks; the theoretical optimum ships in months and with rookie bugs. New tech is only justified if a condition demands it — and then it's declared as a cost (learning curve) in the recommendation.

9. **What's the MOST uncertain thing about the business today?** (the pricing model? the channel? the core feature?)
   *Why:* what's uncertain defines what must be cheap to change. If the business model may pivot, the domain must be decoupled from the infra; if the channel is uncertain (web? WhatsApp? app?), the logic can't live in the frontend.

## Closing the intake

Before moving to recommend, summarize what you understood in 3-5 lines and ask for corrections:

```
Understood: [N users at 6 months], budget [$X/month], operated by [who],
1h outage = [consequence], data [sensitivity/compliance],
team knows [stack]. Most uncertain: [X]. Anything to correct?
```

If the user corrects, update and summarize again. Only with the confirmed summary (or with explicit assumptions where there was no answer) move on to prioritizing the dominant force.
