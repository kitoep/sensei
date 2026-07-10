---
name: ai-features
description: Use when building any LLM-powered feature — chatbots, WhatsApp/customer-support bots, tool-calling/function-calling, agents, RAG, AI classification/extraction, prompt changes, integrating AI provider APIs (Anthropic, OpenAI, etc.), or writing/updating evals; when the user says "add a bot", "hook up the AI", "improve the prompt", or asks to add AI to a product.
---

# AI Features — build with LLMs without hallucinating

## Principle

An LLM feature is not "call the API and cross your fingers." It's: a provider-agnostic layer that isolates the vendor, tools with strict contracts, guardrails that assume hostile input, and evals that decide whether a prompt is allowed to change. The LLM is a **non-deterministic, untrusted-by-design** component: everything around it must compensate for that.

**Rule zero — no hallucinating:** no model name, API parameter, or SDK method goes from your memory straight into the code. Either you verified it against the provider's current docs/SDK in this session, or you flag it as unverified. Models and APIs change faster than your training.

**Violating the letter of these rules is violating their spirit.** There are no "temporary" prototypes exempt from the layer, and no prompt change "too small" for the eval.

## Protocol (load the reference BEFORE working in its area)

| Work area | What it covers | GATE to proceed | Reference |
|---|---|---|---|
| 1. LLM layer | Own provider-agnostic interface: retries, timeouts, cost, logging | No SDK call outside the layer; models/params verified | `references/llm-layer.md` |
| 2. Tools | Tool-calling: strict schemas, errors, idempotency, bounded loop | Every tool passes the contract checklist | `references/tools.md` |
| 3. Guardrails | Prompt injection, scope, escalation, PII, output validation | Guardrails checklist covered or gaps declared | `references/guardrails.md` |
| 4. Evals | Golden set, regression as a prompt-change gate, LLM-judge | No prompt ships without the regression eval run | `references/evals.md` |

A new feature touches all 4 areas. A prompt change touches at least area 4. A new tool touches areas 2 and 3.

## Red flags — STOP if you catch yourself thinking this

- "The model is called `gpt-5-turbo` / `claude-4-sonnet`, I remember" → you don't remember, you're generating it. Verify against current docs/SDK.
- "I'll just import the provider SDK right here" → every call goes through the layer. No "temporary" exceptions.
- "The LLM will call the tool with the right arguments" → the LLM is a clumsy malicious user. The tool validates everything.
- "It's an internal bot, it doesn't need guardrails" → every piece of text entering the prompt is hostile until proven otherwise.
- "The new prompt looks way better" → "looks better" is not a metric. Run the golden set.
- "I'll just parse the response with a regex" → the LLM output is validated against a schema, with a defined fallback for when it returns garbage.
- "I'll `JSON.parse` the response directly" → and when it returns markdown-wrapped JSON, or an apology? Fallback required.

## Known rationalizations

| Excuse | Reality |
|---|---|
| "Verifying the model name is paranoia, I always use this one" | Providers deprecate models every few months. A hallucinated/stale model fails at runtime or (worse) responds worse and nobody knows why. |
| "The abstraction layer is over-engineering for a single provider" | The layer isn't (only) about swapping providers: it's where retries, timeouts, cost, and logging live. Without it, every production bug is archaeology. |
| "Evals are for later, first make it work" | Without a golden set you don't know if it "works." The first prompt defines the baseline; without a baseline, every change is a blind bet. |
| "Prompt injection is theoretical, my users aren't hackers" | The most common injection isn't a hacker: it's a customer pasting an email/document that contains instructions. And one viral case is enough to burn the product. |
| "20 eval cases prove nothing" | 20 real cases catch the gross regressions nothing catches today. It's the minimum viable, not the end state. |
| "The agent loop stops on its own when it's done" | A loop with no iteration or budget cap is an infinite invoice waiting for an edge case. |
