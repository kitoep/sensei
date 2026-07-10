# Front-end and UI/UX testing

Default tool: **Playwright** (or whatever the project already uses — detect with `ls e2e/ tests/ playwright.config.* cypress.config.*` before introducing a new one).

## Core rule

A UI test that only verifies something "renders" proves nothing. Every UI case **exercises an interaction and verifies the effect** — and the effect that counts is the persisted one: after a page reload, the data is still there / gone.

## Implementation rules (Playwright)

1. **Stable selectors:** `getByRole` (first), `getByLabel`, `getByTestId`. Forbidden: CSS classes, positional XPath, `nth-child`. If there's no stable selector, report it as debt — don't build a brittle one.
2. **Wait on state, never on time:** `await expect(locator).toBeVisible()` / `toHaveText()`. `waitForTimeout` is forbidden; if the test only passes with a sleep, it's flaky and reported as FAIL.
3. **Every test starts from known state:** its own login/seed, not dependent on the order of other tests.
4. **Verify the effect, not optimistic UI:** many frontends show success before the server confirms. After the action: reload and re-verify, or query the API/DB.

## UX checklist — the cases that always get skipped

For every flow with forms/mutations, the plan includes these cases (mark explicit N/A if it doesn't apply):

| # | Case | What to verify |
|---|------|----------------|
| 1 | Loading state | When the action fires there's visible feedback (spinner/disabled); the UI isn't "dead". |
| 2 | Empty state | A list/table with no data shows an empty state with a next action, not a blank table or an error. |
| 3 | Visible server error | With the backend returning 500 (or down), the user sees a clear message — not a frozen screen or a console-only error. |
| 4 | Double-click on submit | Rapid double-click → a SINGLE entity created (verify by counting after reload). |
| 5 | Invalid form | Submit invalid data → a specific message NEXT TO the field, the submit doesn't proceed, and the server also rejects it (test via direct API: UI validation is not enforcement). |
| 6 | Back button | After completing the flow, the browser back button doesn't duplicate the action or break state. |
| 7 | Session expired mid-action | With an invalidated token/cookie, the next action redirects to login with a message — it doesn't fail silently or lose the work without warning. |
| 8 | Permissions in UI + API | What the role can't do isn't shown (UX) AND the API rejects it even if called directly (enforcement). |

## Responsive

1. Detect the project's real breakpoints (Tailwind config/design tokens/CSS) — don't invent them.
2. Test the critical flows at concrete viewports: at least **375×667** (mobile), **768×1024** (tablet), and **1280×720** (desktop), adjusted to the detected breakpoints.
3. What to verify per viewport: nothing essential is hidden or cut off, there's no horizontal scroll, touch targets are usable, navigation (collapsed menu) works.

## Minimum accessibility

- The whole critical flow is completable **with the keyboard alone** (Tab in a logical order, Enter/Escape work, focus is visible).
- Inputs have an associated label (if `getByLabel` can't find it, it's an accessibility bug — report it).
- Icons/images with an action have an accessible name (`getByRole('button', { name: ... })` must be able to find them).
- Contrast: essential text is legible; when in doubt, screenshot and flag for review.

## Visual verification

When the change is UI (layout, styles, new component):

1. Screenshot the final state at the 3 viewports (`page.screenshot`), saved with a descriptive name.
2. Compare them against the expected result (design/spec) and cite them as evidence in the verdict.
3. If something looks broken but "the tests pass", the result is FAIL — visual evidence overrides the incomplete assertion.
