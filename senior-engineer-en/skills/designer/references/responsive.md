# Responsive — when it applies, with real targets

"Responsive" isn't a default applied blindly: it's a product decision. The generic mistake is twofold: making responsive what will never be used on mobile (cost without value) or "making it shrink" what will (a second-class mobile). Targets first, patterns second.

## Phase 0 — Decide targets with the user

Ask BEFORE mocking up the first screen (ideally in the design-system intake):

1. **Where will this REALLY be used?** — Not "do you want it responsive?" (everyone says yes), but: "will your end user operate this screen from their phone? in what situation?". An admin panel used in an office is desktop-first; a portal where the end customer schedules/checks is mobile-first almost always.
2. **Are there screens with different targets?** — It's normal: the admin on desktop, the customer view on mobile. Document the target PER AREA, not one global.
3. **Record the decision in `docs/design-system.md`**: targets per area + project breakpoints.

Strategy rule: **mobile-first only if mobile is a real target** of that area. If the target is desktop, design desktop-first and define what happens in small windows (minimum: nothing broken or illegible, horizontal scroll only in tables).

## Breakpoints

- Defined ONCE in the design system (typically 3–4: e.g. `sm 640 / md 768 / lg 1024 / xl 1280` or the project's styling framework's) and used ALWAYS by name. Forbidden to invent ad-hoc breakpoints per screen.
- Breakpoints cut where the CONTENT demands, not by a device list. If a layout breaks at 900px, the fix goes in the nearest standard breakpoint, not a new one at 900.

## Adaptation patterns (instead of "let it shrink")

Each layout type has its transformation. Shrinking proportionally is NOT adapting:

| Desktop | Mobile | Note |
|---|---|---|
| Multi-column data table | Stacked cards with the 2–3 key fields + detail on tap | Choosing WHICH fields survive is a design decision; ask yourself what the user is looking for in the row |
| Navigation sidebar | Drawer (hamburger) or bottom tab bar if ≤5 frequent destinations | Tab bar > drawer for constantly-used primary navigation |
| Grid of N columns | Stack of 1 column (or 2 if items are compact) | The stack's ORDER is decided: the dominant first, not the DOM order by accident |
| Large centered modal | Full-screen sheet or bottom-sheet | A desktop modal shrunk on mobile is unusable |
| 2-column form | 1 column always | Multi-column forms on mobile generate input errors |
| Hover actions (icons that appear) | Actions always visible or behind an explicit menu | On touch there's no hover: any hover-only functionality is BROKEN on mobile |
| Tooltip with necessary information | The information goes visible or behind an explicit tap | Same reason |

## Touch and legibility rules

1. **Touch targets ≥44×44px** (or the platform equivalent) on every interactive mobile element, with enough separation not to touch the neighbor.
2. **Base type on mobile ≥16px** in inputs and body (avoids the browser's automatic zoom on iOS and is the floor for legibility).
3. **Reading widths**: on large screens, running text is limited (~65–75 characters per line); a landing that stretches paragraphs to 1400px wide is broken in the opposite direction.
4. **Thumb zones**: on mobile, the screen's primary action lives in the reachable lower half; the destructive far from easy touch zones.

## Verification — part of each screen's done

No screen with a multi-device target is declared done without being verified in concrete viewports:

- [ ] Render in the agreed viewports (minimum: one mobile ~375px, one desktop ~1440px; tablet if it's a target). Use the project's tools (browser with device toolbar, Playwright with viewports, or a capture) — observed verification, not "it should look fine".
- [ ] Zero accidental horizontal scroll on mobile (horizontal scroll only exists where it was designed, e.g. inside a table).
- [ ] Everything interactive reachable and ≥44px on mobile; nothing depends on hover.
- [ ] Real long text doesn't break the layout (long name, long email, big figure).
- [ ] The 4 states (loading/empty/error/data) also look correct on mobile, not just the desktop happy state.
