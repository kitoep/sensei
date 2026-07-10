# Evidence taxonomy — what counts as "verified" by task type

## The evidence ladder

From lowest to highest. Always report the HIGHEST level you reached, by its real name — never let a low level disguise itself as a high one.

| Level | What you proved | What you did NOT prove |
|---|---|---|
| 0. "Looks good" (reading the diff) | Nothing | Everything |
| 1. Typecheck / lint / build | That the code compiles | That it does anything correct |
| 2. Unit tests pass | What the tests cover | The real flow, integration, state |
| 3. Real execution of the flow | That the exercised path responds | The paths not exercised, the persisted state |
| 4. Real state queried | The observable effect in the system (DB, file, live environment) | Nothing — this is the standard for "done" |

**Rule:** a behavior claim ("works", "is set", "responds") requires level 3 minimum; an effect claim ("saved", "migrated", "deployed", "sent") requires level 4. Level 1-2 only authorizes you to say "compiles / the tests pass".

## Minimum evidence by task type

### New feature
- Run the full end-to-end flow the way the user would (real request, real command, or real UI) — not just the function in isolation. Paste the output.
- Minimum 3 cases: the happy one + 2 adversarial (invalid input, empty/boundary case). "I tested the happy one" doesn't authorize "it works"; it authorizes "the happy case works".
- If the feature persists or sends something: query the real state (DB row, created message, generated file) and paste the result.

### Refactor
- Full suite of the area BEFORE and AFTER, with summaries pasted (`X passed, 0 failed` in both). Same green before and after = behavior preserved; green only after proves nothing.
- Search for all uses of the changed symbol/contract (`grep` pasted) — each call-site updated or justified.
- Run the main flow that goes through the refactored code once (level 3), not just the tests.

### Config change
- Don't read the file you edited: query the EFFECTIVE config in the running system (config endpoint, variable printed by the process, framework command like `config:show`). The correct file with the process not restarted is the classic fake green.
- Exercise the behavior the config controls and paste the observable difference.

### Deploy
- Query the LIVE environment: deployed version/commit (`/version`, header, startup log), not the green pipeline. Green pipeline = the deploy ran, not that yours is up.
- Run against the deployed environment the flow you changed and paste the real response.
- Check post-deploy logs for startup errors (paste the relevant lines or "0 errors in the first N minutes").

### Data/schema migration
- Declare in WHICH environment it ran. "The migration exists" ≠ "the migration ran".
- Query the real schema afterward (`\d table`, `SHOW CREATE TABLE`) and paste the relevant part.
- If it migrates data: count before/after (`SELECT count(*) ...`) and sample real migrated rows.
- The app is still alive against the new schema: a real post-migration request pasted.

### UI
- Screenshot or Playwright run of the REAL rendered state — correct JSX is not evidence that it looks right.
- The non-happy states you touched: loading, empty, error, long content. The one you didn't capture, declare unverified.

### Script / job / cron
- Actually run it, with real or representative data, and paste the output. A script never executed is broken by default (imports, permissions, paths).
- Verify the effect on state (files created, rows affected, messages queued) — not just exit code 0.

### External integration (third-party API, webhook, LLM)
- At least one REAL call (sandbox if it exists) with request and response pasted. Mocks test your code against your assumption of the contract, not against the contract.
- If you could only test with mocks, report it that way: "verified against mocks; real call pending".

### Documentation that claims behavior
- Execute every command/snippet the doc claims, in a clean environment if it's about installation. Unverified docs document your memory, not the system.

## If you can't execute the verification

Don't invent a level: lower the claim. Report "applied, verification pending", say WHY you couldn't (no access to the environment, missing credential) and provide the exact command the user must run and what output to expect. See `references/language.md`.
