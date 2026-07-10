# Tools — strict contracts for tool-calling

**Mandatory mental model: the LLM calling your tools is a clumsy malicious user.** It will send missing arguments, wrong types, made-up IDs, and it will call the same tool twice. The tool never "trusts" the LLM: it validates, bounds, and reports.

## Contract for every tool (checklist — GATE)

- [ ] **Strict schema:** every field typed, `required` explicit, enums where the domain is closed, no open `additionalProperties`. The schema is the first line of defense, not documentation.
- [ ] **Description for the LLM, not for humans:** what it does, when to use it, when NOT to, what it returns, and examples of valid arguments if the format isn't obvious. An ambiguous description = garbage calls.
- [ ] **Full internal validation** even though the schema already validates: ID existence against the DB, ranges, end-user permissions (the tool runs with the permissions of the conversation's USER, not system permissions).
- [ ] **Errors returned TO the LLM as a result**, not as an exception that kills the loop: `{ error: "order_not_found", hint: "confirm the number with the customer" }`. A good error message lets the LLM correct itself or escalate; a stack trace leaks internals into the prompt.
- [ ] **Idempotency on write tools:** the LLM CAN call twice (network retry, confused loop). Use idempotency keys (`conversationId + intent`), or check-before-create, or design the operation so repeating it is harmless. "Create order" without idempotency = duplicate orders in production, guaranteed.
- [ ] **The tool result is untrusted content** when it contains external data (emails, product descriptions, prior messages): see `guardrails.md`, injection section.

## Read vs write

| Type | Examples | Rule |
|---|---|---|
| Read | look up an order, search a product, check hours | The LLM can call them freely (with per-user scoping). |
| Reversible write | add an internal note, mark a follow-up | Allowed with idempotency + an auditable log. |
| Write with external effect | create an order, charge, cancel, email a third party | **Explicit end-user confirmation before executing** ("Confirming: cancel order #123?" → yes) or a human approval queue. The bot proposes, the human disposes. |

If a write tool with external effect has no confirmation step, it's an incident with a future date.

## The agentic loop is always bounded

```python
MAX_ITERATIONS = 8          # per conversation turn
MAX_COST_USD = 0.50         # budget per turn

for i in range(MAX_ITERATIONS):
    resp = call_llm(...)
    if resp.stop_reason != "tool_use":
        return resp                      # final response
    if spent() > MAX_COST_USD:
        return escalate("budget exceeded")   # to a human, see guardrails.md
    results = [run_tool(tc) for tc in resp.tool_calls]
    # each run_tool catches ITS OWN exceptions and returns {error: ...} as a result
return escalate("loop did not converge")    # NEVER a while True
```

Loop rules:

- **Iteration and cost cap** per turn. On exceeding them: escalate to a human with context, don't cut off silently and don't keep burning money.
- **Dumb-loop detection:** same tool + same arguments twice in a row → don't run it again; return to the LLM "you already called this with these arguments, the result was X" or escalate.
- **Parallelism only on reads.** Writes: sequential and in the requested order.
- **Log per iteration:** tool, arguments, result (truncated), duration. When a bot does something weird in production, this log is the only way to know what happened.

## Common mistakes you'll avoid

| Mistake | Consequence | Fix |
|---|---|---|
| Tool throws an uncaught exception | The bot's whole turn crashes | try/catch inside the tool → `{error}` to the LLM |
| Description "gets order info" | The LLM calls it for everything or nothing | When to / when not to / what it returns |
| `orderId: string` with no format validation | The LLM invents IDs and the tool queries garbage | Validate format + existence; error with a hint |
| No per-user scoping | One customer's bot reads another's orders | Every query filtered by the conversation's identity |
| Giant tool result (50KB JSON) | Bloated context, 10x cost, worse responses | The tool summarizes/selects fields; the LLM doesn't need the full row |
