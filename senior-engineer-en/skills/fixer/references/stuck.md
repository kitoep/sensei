# Stuck — unblocking and escalation protocol

Activate this protocol when: 3 dead hypotheses, >30 minutes with no measurable progress, or you catch yourself re-testing something you already tested.

## Step 1 — STOP and question the higher-level assumptions

When all the "reasonable" hypotheses die, the failure is in something you took for granted. Verify MECHANICALLY (not from memory) each one:

| Assumption | How to verify it in 30 seconds |
|---|---|
| "I'm editing the file that actually runs" | Drop a `throw new Error('MARKER-123')` or a unique log in the file; if it doesn't show up when you run, you're editing the wrong file (stale build, a copy, another monorepo package, dist/ vs src/). |
| "The build is fresh" | Delete the build/cache and recompile (`rm -rf dist .next node_modules/.cache && build`). Watchers lie. |
| "I'm running the version/environment I think" | Print at runtime: version, commit (`git rev-parse HEAD`), relevant environment variables, DB URL (host, not credentials). Compare against what you assumed. |
| "The bug is where I'm looking" | Log the suspicious value AT THE ENTRY of the module you're investigating. If it arrives already corrupt, the bug is upstream and you've spent hours looking in the wrong place. |
| "The data is what I think" | SELECT directly against the row / print the real payload. Not the fixture, not the type: the live data. |
| "The test I'm running is the one I think" | Is the runner filtering/skipping? Are there two tests with similar names? Run with the file's full path. |

The classic: an hour "debugging" a change that was never being executed.

## Step 2 — Unblocking techniques

- **Reduce to the minimal case:** copy the flow into an isolated script and remove pieces until the bug disappears. The last piece removed is the culprit (or its interaction). It also works in reverse: start from scratch and add pieces until it shows up.
- **Compare against a case that DOES work:** find the healthy twin (another endpoint that does save, another tenant where the message does arrive, the same flow in another environment). Diff EVERYTHING between the two: code, config, data, headers, logs. The difference contains the cause.
- **Clean vs dirty output:** capture the full trace/log of a good run and a bad one and diff them line by line. The first point of divergence bounds the bug better than any theory.
- **Change sensor:** if you've been reading code for a while, stop reading and observe the runtime (logs, DB, network, processes). If you've been in the runtime for a while, read the code at the point of divergence. Alternating breaks the blindness.
- **Re-read the full original error** one more time, literally. With the accumulated context, phrases you ignored at the start now mean something.

## Step 3 — When and how to escalate to the user

Escalate when: you've exhausted steps 1–2, or you need information/access only the user has (prod data, credentials, "what exactly did you do?"), or the cost of continuing exceeds the value (2+ hours without reducing the problem space).

**Escalating is NOT giving up; burning time in circles is what fails the user.**

Honest escalation format (mandatory — never escalate with "I don't know what's happening"):

```
STUCK on <bug>. Summary of what's been ruled out:

REPRODUCTION: <command and how deterministic it is>
DEAD HYPOTHESES:
  1. <hypothesis> — ruled out because <evidence>
  2. <hypothesis> — ruled out because <evidence>
  3. <hypothesis> — ruled out because <evidence>
VERIFIED ASSUMPTIONS: correct file ✓, fresh build ✓, env ✓, ...
WHAT I KNOW: <facts confirmed with evidence>
WHAT I DON'T KNOW: <the concrete unknown>
WHAT I NEED FROM YOU: <specific question or concrete access>
NEXT STEP IF NO ANSWER: <what you'd try next>
```

This summary turns your hours of ruling-out into value: the user (or another agent) starts from your frontier, not from scratch.
