# Interrogating: from fuzzy to answerable

## Context before questions, always

Never ask the first question blind. Before anything, look at the project: relevant files, docs, recent commits, the tracker if there is one. Two reasons:

1. Half your questions may already be answered by the codebase ("do we have auth?" is a grep, not a question).
2. The idea may already exist, wholly or in part. If it does, say so before refining anything.

A question whose answer was sitting in the repo costs you the user's trust.

## Size check before refining

Before polishing any detail, measure the idea. If it contains several independent pieces ("a dashboard with alerts, exports, and per-user permissions" is three ideas), STOP refining and decompose:

- Name the independent pieces.
- Ask which one matters first.
- Shape THAT one. The rest get one line each in the anti-scope.

Polishing details of an idea that needed splitting is the most common way to waste a shaping session.

## One question at a time

Never send a wall of questions. One per message. If a topic needs three questions, that is three messages.

Prefer multiple choice with your recommendation marked:

> Where should the alert arrive?
> a) Email (recommended: you already have the SMTP integration and it needs no new UI)
> b) WhatsApp (needs template approval, days of lead time)
> c) In-app only (cheapest, but nobody sees it if they don't log in)

Multiple choice is cheaper to answer, and a marked recommendation lets the user reply "go with your defaults" and keep moving.

## The layers (cover all five before landing)

| Layer | The question behind it | Why it kills rework |
|---|---|---|
| Problem | What real problem does this solve, and whose problem is it? | Features without an owner get built and go unused |
| Minimum | What is the smallest version that is already useful? | Defines v1; everything else is a later |
| Anti-scope | What is this NOT? Which tempting neighbor features are out? | The written NO is what keeps scope from creeping back in |
| Success | How will we know it worked? What changes, for whom, measured how? | Without this, "done" is an opinion |
| Constraints | Data available, permissions, integrations, money, deadlines | The idea that ignores a constraint gets redesigned the day it hits it |

For the express scale, one question per layer is usually enough (5 questions). Skip a layer only if the context check already answered it, and say which answer you took from where.

## Signals you are done asking

- You can state the problem in one sentence the user would sign.
- You know what v1 excludes, not just what it includes.
- No requirement can still be read two ways: each one has exactly one interpretation.

If a new answer contradicts an earlier one, surface it immediately ("earlier you said X, this implies Y, which one wins?"). Do not average them silently.
