# Feedback and motion — the user is never left without a response

A UI feels broken when the user acts and nothing visible happens. Mother rule: **every user action has a visible response in <100ms** — even if the real operation takes seconds, the acknowledgment is immediate (pressed state, spinner in the button, optimistic update). Freezing the UI or leaving the button unchanged while it "thinks" is forbidden.

## Which pattern per case (decision table)

| Situation | Mandatory pattern | Never |
|---|---|---|
| Confirmation of a successful non-blocking action (saved, sent, copied) | Brief toast (3–5s), auto-dismisses, with undo if the action is reversible | A "Success!" modal you have to close |
| Field validation error | Inline message next to the field, on submit or blur; the field marks an error state; focus to the first field with an error | A generic toast or alert "check the form" |
| Operation error (network, server) | Message in the action's context with a retry button; preserves what the user typed | Losing the user's input; error only in the console |
| Destructive or irreversible decision (delete, cancel subscription) | Confirmation modal that names the object ("Delete 'Consultation from Mar 12'?") and a differentiated destructive button | Confirming with a toast, or deleting without confirmation |
| Initial page/view load | Skeleton that respects the final layout (no jump when it loads) | Full-page spinner; blank screen |
| Short action (<1s expected) triggered by a button | Spinner/loading state INSIDE the button, button disabled meanwhile | Disabling with no indicator; a global spinner |
| Long operation (>3s) | Progress or status message + the ability to keep using the app if possible | Blocking everything with no information |
| Mutation with high success probability (toggle, like, reorder, mark done) | Optimistic UI: reflect the change now, revert with a notice (toast with the reason) if the server fails | Optimistic on payments, deletions or non-reversible actions |

## The 4 mandatory states of every data view

No view is done without its 4 states implemented. When building, write all 4 from the start:

1. **Loading**: skeleton in the shape of the final content.
2. **Empty**: NEVER a dry "no data". Format: what this emptiness means + a suggested action to get out of it ("No appointments yet. [Schedule the first]"). The first use of the product IS the empty state: it's designed, not left to fall.
3. **Error**: what failed in the user's language (no codes or jargon) + a retry button + the user's data intact.
4. **With data / success**: the happy state, including the "lots of data" case (pagination? scroll?) and the "extreme datum" case (very long text, huge figure).

## Animation rules

1. **Durations**: micro-interactions (hover, pressed, toggle, tooltip appearance) 150–200ms; element transitions (modal, drawer, toast, expand) 200–300ms; never >400ms in functional UI. Slower = the UI feels heavy.
2. **Easing**: `ease-out` for elements coming in (fast at the start, lands softly), `ease-in` or `ease-in-out` for those going out. Never `linear` for spatial movement.
3. **Animate only `transform` and `opacity`** (composited, 60fps). Forbidden to animate layout properties (`width`, `height`, `top`, `margin`) except with a specific technique that doesn't cause a reflow per frame.
4. **`prefers-reduced-motion` is respected ALWAYS**: with the preference on, movement animations reduce to fades or are removed. It's an accessibility requirement, not optional.
5. **Animation communicates cause-and-effect, never decorates.** Every animation must answer "what does this explain to the user?": where the element came from (the modal emerges from the center, the drawer from the edge), where it went (the deleted item collapses, the toast exits toward its edge), what changed (the toggle slides). If the answer is "it looks cool" → remove it. Cascading entrance animations across the whole page, decorative parallax and elements floating on loop: out, unless the design system's visual direction explicitly calls for it on a landing page.
6. **Consistency**: same entrance/exit patterns for the same type of element across the app. The modal always enters the same way.

## Done checklist (feedback) per feature

- [ ] Every action has an ack <100ms (go through each button/interaction of the feature)
- [ ] The 4 states of each view implemented
- [ ] Errors preserve the user's input and offer a way out (retry/fix)
- [ ] Destructive actions confirm by naming the object; reversible ones offer undo
- [ ] Animations within duration ranges, transform/opacity only, with reduced-motion respected
