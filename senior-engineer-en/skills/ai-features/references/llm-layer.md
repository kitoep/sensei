# LLM Layer — a single door to the provider

**Hard rule: no file in the project imports the provider SDK except the LLM layer.** If you find `import anthropic` / `import openai` outside `llm/` (or the project's equivalent module), that's an architecture bug: move it before continuing.

## Why the layer exists

It's not (primarily) about "swapping providers someday." It's because these 6 things must happen on **every** call, and they're only guaranteed if there's a single chokepoint:

1. Retries with backoff on 429/5xx/timeouts.
2. Explicit timeout (the SDK default can be minutes).
3. Tracking of tokens, cost, and latency per request.
4. Logging of prompt and response (with PII redaction, see guardrails.md).
5. Model selection by config, not hardcoded at the call-site.
6. Uniform error handling (rate limit ≠ content refusal ≠ network down).

## No hallucinating — verification protocol

Before writing a model name, parameter, or SDK method:

| What you're about to write | How you verify it FIRST |
|---|---|
| Model name (`claude-...`, `gpt-...`) | Provider's current docs, or the models-list API, or the model already used in the repo (grep). NEVER from memory. |
| Request parameter (`temperature`, `max_tokens`, `tool_choice`...) | Current API reference or the installed SDK's types (`node_modules/.../types`, `pip show` + source). |
| SDK method/signature | The SDK installed in THIS repo (read its types/source), not the version you remember. |

If you can't verify (no network, SDK not installed): write the code with the value in config + a `VERIFY:` comment and flag it explicitly in your report: "the model name is unverified against current docs." A flagged-uncertain value is honest; a confidently-hallucinated one is a landmine.

**Correct default:** if the repo already calls an LLM, use the same model/version already in production unless told otherwise.

## Minimal layer interface

```typescript
// llm/client.ts — the ONLY module that imports the provider SDK
export interface LlmRequest {
  purpose: string;            // "bot-reply", "classify-ticket" — for metrics and logs
  system: string;
  messages: Msg[];
  tools?: ToolDef[];
  maxTokens: number;          // ALWAYS explicit
  temperature?: number;
  model?: ModelAlias;         // your own alias ("fast" | "smart"), mapped to real IDs in config
}

export interface LlmResponse {
  text: string | null;
  toolCalls: ToolCall[];
  stopReason: "end" | "tool_use" | "max_tokens" | "refusal";
  usage: { inputTokens: number; outputTokens: number; costUsd: number; latencyMs: number };
}

export async function callLlm(req: LlmRequest): Promise<LlmResponse>
```

Non-negotiable implementation points:

- **Model aliases in config**, not IDs scattered around: `{ fast: "<verified-id>", smart: "<verified-id>" }`. Swapping models = 1 line of config.
- **Retries:** only for transient errors (429, 5xx, network timeout), exponential backoff with jitter, max 3 attempts. NEVER retry a content refusal or a 400: the result will be the same and you pay twice.
- **Per-request timeout** matched to the use case (a chat bot can't wait 120s).
- **`stopReason: "max_tokens"` is a product bug**, not a success: the response is truncated. Handle it (retry with more budget, or fail clean), don't hand it to the user.
- **Cost computed from prices in config** (these change too). Log `purpose`, tokens, cost, and latency on every call — that's what lets you answer "why did the bill 3x?" without archaeology.

## Checklist — GATE to sign off on the layer (or a feature that uses it)

- [ ] `grep` the SDK name in the repo: 0 hits outside the layer module.
- [ ] All model IDs live in config and were verified (or marked `VERIFY:`).
- [ ] `maxTokens` and timeout explicit for every `purpose`.
- [ ] A request that fails with 429 is retried; one that fails with 400 is not.
- [ ] Every call leaves a log/metric line with purpose, tokens, cost, latency.
