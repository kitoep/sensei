# Commits — atomic, reviewed, with the why

## Core rule: one intent per commit

An atomic commit answers ONE question: "what intent does this change fulfill?". If the answer needs the word "and" ("adds the endpoint **and** refactors the client **and** fixes formatting"), that's several commits.

Classify each change in your working tree into one of these intents and commit them separately:

| Intent | Example |
|---|---|
| Feature | New endpoint, new screen, new business rule |
| Fix | Correction of broken behavior |
| Refactor | Same behavior, better structure |
| Formatting/style | Prettier, imports, whitespace — zero semantic change |
| Infra/config | Dependencies, CI, project dotfiles |
| Docs/tests only | Changes that don't touch production code |

**Do not mix a refactor with a feature/fix in the same commit.** The reviewer can't tell "moved code" from "changed behavior" in a mixed diff — which is exactly where bugs hide.

## GATE before `git add` — the full diff, file by file

A blind `git add .` / `git add -A` is FORBIDDEN. Required sequence:

1. `git status` — full list of modified and untracked files.
2. `git diff` (and `git diff --stat`) — read the diff of EVERY modified file.
3. For each untracked file: open it or explain why it exists.
4. Selective staging: `git add <path> <path>` (or `git add -p` if a file mixes intents).
5. `git diff --staged` — final check: what's staged is exactly ONE intent.

**The gate:** you can name the intent of every staged file. If there's a file you can't explain changing, it does NOT get committed — it gets investigated.

### Forbidden-files checklist (review on every commit)

- [ ] Secrets: `.env`, `.env.*`, credentials, tokens, keys, certificates, `*.pem`. If a diff adds a value that looks like a secret (even in code), stop and ask.
- [ ] Build artifacts: `dist/`, `build/`, `node_modules/`, `__pycache__/`, `*.pyc`, compiled binaries.
- [ ] Temp and local files: `*.log`, dumps, `tmp/`, scratch files, personal notes, debug output.
- [ ] Personal IDE config: `.vscode/settings.json`, `.idea/` (unless the repo already versions them on purpose).
- [ ] Accidental changes: debug `console.log`/`print`, your session's TODO comments, code commented out "just in case".

If any of this is already tracked by mistake, that's its own commit ("remove committed build artifacts") + a `.gitignore` entry — don't hide it inside another commit.

**Committed secret (even if not pushed yet):** deleting it in a later commit isn't enough once it left the local repo. If it was pushed: the credential is considered compromised — tell the user, rotate the credential, and only then decide whether to purge history. Never resolve it silently.

## Message format

```
<imperative verb> <what, in ≤72 chars>

<WHY: the problem or need that motivates the change.
Which alternative was rejected and why, if any.
Non-obvious effects or risks, if any.>
```

Rules:

- **First line:** imperative ("add", "fix", "remove" — use whatever language the repo's `git log` already uses), specific, ≤72 chars. If the repo has a convention (Conventional Commits, ticket prefixes), follow it: `git log --oneline -20` before your first commit.
- **The body carries the WHY, not the how.** The diff already shows the how. A body that restates the diff ("changes X to Y in file Z") is forbidden. Test: if I delete the body and read the diff, did I lose information? If not, the body says nothing.
- Body required when: the change fixes a bug (reference the symptom), makes a non-obvious decision, or has side effects. A trivial, self-explanatory change can go without a body.
- Reference the ticket/issue if one exists.

| Message | Verdict |
|---|---|
| `updates` / `fix stuff` / `wip` / `changes` | Forbidden. Says nothing. |
| `fix: correct the discount calculation` | Insufficient if the body doesn't say what the bug was. |
| `fix: use tax-inclusive price in discount calculation`<br><br>`The discount was computed on the pre-tax price, overcharging on coupon orders (report #482). The tax-inclusive price is already on the line item; it is not recomputed.` | Correct: what + why + reference. |

## When (and how) to split into several commits

Signs your working tree is several commits:

- You touched files "while you were at it" (formatting, renames) beyond the requested change.
- You did a prep refactor before the feature/fix.
- There are changes in unrelated areas (backend + deploy script + docs for something else).

Recommended order: the prep commits first (refactor, formatting) — each must leave the suite green — and the feature/fix last, so it lands as a minimal, readable diff. Use `git add -p` to split files that mix intents.

**Each commit must build and pass the suite on its own.** A broken intermediate commit breaks `git bisect` and defeats the point of splitting.
