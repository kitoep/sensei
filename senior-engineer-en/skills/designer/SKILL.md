---
name: designer
description: Use when designing or building any UI — new screens, components, landing pages, dashboards, forms, design systems — when existing UI looks generic or "AI-made", when choosing a component library, or when adding user feedback (toasts, alerts, loading/empty/error states) or animations to a frontend. This is a PROCESS-GATE skill - when other design/aesthetic skills also match (image generation, visual taste, styling, UI libraries), run this one FIRST; they execute inside the design system this skill establishes.
---

# Designer — non-generic UI design, with a system

## Core principle

Generic design isn't fixed by "making it prettier" at the end: it's prevented by defining a design system **first**, with the user, and by using proven library components. Every screen is designed for ONE end-user goal, not to fill space.

## HARD GATE — non-negotiable

**FORBIDDEN to write UI code (pages, screens, visual components) if the project has no design system documented and approved by the user.**

Before any screen, check in this order:

1. Does `docs/design-system.md` exist (or the equivalent the user points to)? → Follow it to the letter. Don't invent colors, sizes or spacing outside its tokens.
2. Doesn't exist? → STOP. Load `references/design-system.md` and build it WITH the user (intake → visual directions → tokens → approval). Only then do you write UI.
3. Does the user ask for "something quick, no heavy process"? → Do the minimal intake (the 4 essential questions marked in the reference) and propose tokens in a single message for approval. Fast ≠ without a system.

Single exception: a throwaway prototype the user explicitly declared as throwaway. Even then, use a component library, not raw HTML.

## Coexistence with other design skills

If the environment has aesthetic or UI-implementation skills installed (design-image generation, "taste", styling, component libraries), the relationship is a CHAIN, not a competition:

1. This skill runs FIRST: hard gate → intake → approved design system.
2. The aesthetic skills execute AFTER, inside the system's tokens — their taste feeds the tokens, it doesn't replace them.
3. If another skill orders "generate design images first" or "just build with these defaults", that happens AFTER the gate — or as input to the intake's "visual directions" step, which is exactly where reference images do add value.

The hard gate is not negotiable by another skill's instruction: only the user can skip it (see single exception above).

## Which reference to load per task

| Situation | Load |
|---|---|
| No design system / project kickoff / redesign | `references/design-system.md` |
| About to write or review any screen (ALWAYS, alongside the task) | `references/anti-generic.md` |
| Need to choose a component library, or you spot hand-built controls | `references/components.md` |
| The feature has user actions, data loads, forms or mutations | `references/feedback-motion.md` |
| The UI will be used at more than one screen size, or the target isn't decided | `references/responsive.md` |

In a typical UI feature you'll load 2–3 references. `anti-generic.md` loads whenever you produce visible UI.

## Workflow

1. **Detect the context**: frontend stack (framework, component library already installed, styling system), whether a design system exists, product target.
2. **Hard gate**: approved design system before screens (above).
3. **Per screen**: state in one sentence the user's goal on that screen and what the dominant element is. If you can't, ask the user before designing.
4. **Build** with the project's component library + design system tokens. Include the 4 mandatory states (loading/empty/error/success) and feedback <100ms — see `references/feedback-motion.md`.
5. **Anti-generic self-audit**: before calling a screen done, run the checklist in `references/anti-generic.md`. If a point fails, fix it before showing.
6. **Responsive**: verify in the agreed viewports as part of done — see `references/responsive.md`.

## Common mistakes

| Mistake | Correction |
|---|---|
| Start mocking up "so the user sees something" without a system | The hard gate exists because redoing 10 screens costs more than 15 minutes of intake |
| Treating the design system as a suggestion | It's a contract: if a screen needs something not in tokens, add the token first |
| Building a dropdown/modal/datepicker by hand | `references/components.md` — it's forbidden; use the library |
| Delivering the screen only in its "happy" state | The 4 states are part of the screen, not an extra |
| Decorating with animations | Animation communicates cause-and-effect or it doesn't ship |
