---
name: git-delivery
description: Use when about to run git add, git commit, or git push; when creating a branch or pull request; when writing a commit message or PR description; when the user says "commit this", "push the changes", "open the PR", or "merge it"; when deciding what goes in a PR or how to split large work; or before any history-rewriting operation (amend, rebase, force-push).
---

# Git delivery — auditable commits, branches, and PRs

## Principle

Work isn't delivered when the code works: it's delivered when a human reviewer can audit it. The unit of delivery is the atomic commit and the single-topic PR with a verifiable description. A 2,000-line dump with an "updates" message isn't a delivery; it's debt the reviewer pays.

**Violating the letter of these rules is violating their spirit.** No change is "too small" for the process, and no deadline justifies a blind `git add .`.

## Protocol (load the reference BEFORE acting)

| Moment | What you do | GATE to proceed | Reference |
|---|---|---|---|
| Before `git add` | Review the full diff, file by file | You can name the intent of EVERY staged file; zero secrets/builds/temp files | `references/commits.md` |
| Before `git commit` | One intent per commit + message with the why | The message answers "why?" and the staged diff contains only that intent | `references/commits.md` |
| Before creating a branch/PR | Branch per unit of work; single reviewable topic | The PR fits in one review (<~400 net lines) or is split into stacked PRs | `references/branches-prs.md` |
| Before writing the description | Complete auditable description template | All 5 sections filled, with verification evidence pasted | `references/pr-description.md` |
| Before push/amend/rebase/force | Confirm the operation is allowed | The user asked for it explicitly, or it's a normal push of your own branch | `references/branches-prs.md` |

## Forbidden without an explicit request from the user

- Committing or pushing directly to `main`/`master`/`develop`.
- `push --force` (in any variant) to a shared branch.
- `commit --amend` or `rebase` of already-pushed commits.
- Skipping hooks (`--no-verify`) or signatures.

## Red flags — STOP if you catch yourself thinking this

- "`git add .` and done, I know what I changed" → you don't. Stray files, `.env`, and builds get in exactly like that. Diff first.
- "I'll squash it all into one commit, it's the same feature anyway" → if it mixes refactor + feature + formatting, that's 3 commits. Split them.
- "The message doesn't matter, the diff explains itself" → the diff says WHAT changed; only the message can say WHY. Write it.
- "I'll generate the PR description by summarizing the diff" → restating the diff is noise. The reviewer already sees the diff; they need context, verification, and risks.
- "I'll force-push to keep the history clean" → shared history isn't rewritten without an explicit request.
- "It's a small change, straight to main" → production incidents are also small. Branch + PR.

## Known rationalizations

| Excuse | Reality |
|---|---|
| "Reviewing the whole diff takes too long" | It takes 2 minutes. A committed secret forces you to rotate credentials and purge history: hours, plus the incident. |
| "Nobody reads commit messages" | They're read exactly when something breaks (`git log`, `git blame`, bisect). There, "fix stuff" costs hours of archaeology. |
| "One big PR is more efficient than three small ones" | A 2,000-line PR gets rubber-stamped without a real read, or weeks of ping-pong. Three stacked PRs get reviewed seriously and merge sooner. |
| "The PR already describes what I did, CI runs the tests" | CI verifies it compiles and the suite passes; it doesn't verify the change does what the PR promises. Verification evidence goes in the description. |
| "I'll amend the commit to avoid dirtying the history" | If it's already pushed, amending rewrites history others may have. New commit. |
