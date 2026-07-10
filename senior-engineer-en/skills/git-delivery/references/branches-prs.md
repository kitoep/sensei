# Branches and PRs — scope, size, and forbidden operations

## Branches

**Rule:** branch per unit of work. Never work directly on `main`/`master`/`develop` — if you're already sitting on the default branch, create the branch BEFORE the first commit.

Naming: follow whatever convention the repo already uses (`git branch -r` to see it). If there's no convention: `<type>/<short-kebab-description>` — `feat/checkout-oxxo`, `fix/tax-discount`, `chore/bump-eslint`. With a tracker: include the ID (`feat/YU-482-checkout-oxxo`).

One branch = one future PR. If an unrelated bug shows up while you work: do NOT fix it on this branch. Note it, or branch off `main` separately for that fix.

Before opening the branch, start from an up-to-date base: `git fetch origin && git switch -c feat/x origin/main`. A branch born from a stale base creates conflicts that contaminate the PR.

## PR scope

**A PR = one topic a reviewer can audit in one sitting.**

| Metric | Threshold | When exceeded |
|---|---|---|
| Net lines changed (excluding lockfiles/generated) | ~400 | Consider splitting into stacked PRs |
| Net lines | ~1000 | FORBIDDEN as a single PR except the mechanical case (below) |
| Distinct topics (feature + unrelated refactor + unrelated fix) | >1 | Split, always |
| Files touched | ~20 | Warning sign: is it one topic or several? |

Mechanical exception: a large but uniform change (global rename, auto-formatting, lockfile regeneration, codemod) can be a big PR — ALWAYS on its own, never mixed with logic changes, and stating in the description the command that generated it so the reviewer can reproduce it instead of reading it.

### How to split large work: stacked PRs

Split by deliverable layers, each green and reviewable on its own:

1. **PR 1 — prep:** refactors/renames that enable the feature, with no behavior change. Base: `main`.
2. **PR 2 — core:** the minimal behavior change (model + logic + tests). Base: PR 1's branch.
3. **PR 3 — surface:** UI, extra endpoints, consumer migration. Base: PR 2's branch.

Stack rules:

- Each PR states in its description which one it depends on ("Stacked on #12; review only the last commit range").
- They merge in order; after merging PR 1, rebase PR 2 onto `main` (this IS an allowed rebase: it's your own PR branch, and you flag it in the PR).
- Alternative when the stack isn't worth it: merge PR 1 first and open PR 2 afterward, in series.
- An incomplete-but-safe feature (dark launch / feature flag off) is the standard way to merge the core without exposing a half-built surface.

## Operations forbidden without an explicit request from the user

"Explicit" = the user asked for it in those words in THIS conversation. "I did it before on another PR" is not permission.

| Operation | Why | Allowed alternative |
|---|---|---|
| Commit/push directly to `main`/`master`/`develop` | Skips PR review and CI | Branch + PR |
| `git push --force` / `--force-with-lease` to a shared branch or `main` | Erases others' work | New commit; if it's YOUR PR branch and you need to rebase the stack, `--force-with-lease` on your branch only, flagged in the PR |
| `git commit --amend` of an already-pushed commit | Rewrites history others may have | New commit ("fixup: ...") |
| `git rebase` of already-pushed commits on a shared branch | Same | Merge `main` into your branch to catch up |
| `git push --delete` / deleting others' remote branches | Destructive and unrecoverable | Ask for confirmation with the exact name |
| `--no-verify`, disabling hooks or signatures | The hooks exist for a reason | Fix what the hook flags |
| `git reset --hard` / `git checkout --` over changes you didn't create | Can destroy the user's work | `git stash` (recoverable) + tell them |

Before any operation in this table that the user DID ask for: state what you're about to run and its effect, in one line, and run it exactly as described.

## Before opening the PR — checklist

- [ ] Branch up to date with `main` (merge or local pre-push rebase) and conflict-free.
- [ ] The suite passes on the last commit (`X passed, 0 failed` — paste the summary in the description).
- [ ] The PR diff (`git diff main...HEAD`) contains ONLY the PR's topic. Stray change → move it to another branch.
- [ ] No forbidden files (`commits.md` checklist) across the WHOLE range, not just the last commit: `git diff main...HEAD --stat`.
- [ ] Complete description per `pr-description.md`.
- [ ] PR title = a well-written first commit line: imperative, specific, with the prefix/ticket if the repo uses one.
