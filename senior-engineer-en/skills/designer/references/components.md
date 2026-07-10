# Components — library first, never reinvent controls

Building interactive controls by hand produces accessibility and keyboard bugs that don't show in the demo but break the product. The rule is library-first; custom is the justified exception.

## Hard rule

**FORBIDDEN to build by hand** when a proven accessible option exists in the project's ecosystem:

- Date pickers / time pickers / calendars
- Dropdowns, selects, comboboxes and autocompletes
- Modals, dialogs, drawers, popovers, tooltips
- Tables with sort/filter/pagination/selection
- Tabs, accordions, menus (including context menu)
- Toasts/notifications, sliders, switches

These components have years of solved edge cases (focus trapped in a modal, keyboard navigation in a combobox, correct ARIA, scroll-lock, popover collisions). Rebuilding them "because it's quick" throws away that work and the result is always worse.

## Library decision flow

1. **Does the project already use a component library?** → Use that one. Don't introduce a second UI library without discussing it with the user (two libraries = two aesthetics + double weight).
2. **No library?** → Before the first screen, evaluate options in the framework's ecosystem and **propose the ideal one to the user with justification** (2–3 candidates, your recommendation first and why). The choice is the user's; the criteria are yours.

## Selection criteria (in this order)

1. **Accessibility included**: keyboard, focus, ARIA and screen readers solved by the library (not "accessible if you add it"). It's criterion #1 because it's the most expensive to retrofit.
2. **Compatibility with the design system's tokens**: can you map your palette/type/radii without fighting the library? Prefer libraries with variable/token theming over closed-aesthetic ones.
3. **Composability**: pieces that combine and get styled, not config-driven monoliths impossible to tune.
4. **Active maintenance**: recent releases, issues attended to, community. A dead library is immediate debt.
5. **Weight and delivery model**: tree-shakeable / per-component; on mobile or public pages, the bundle matters.
6. **Real compatibility with the framework and its render mode** (SSR/streaming/native as applicable).

## Landscape by type (recommend per case, without marrying brands)

- **Headless/unstyled + your own styles**: primitives with behavior and accessibility solved, 100% your appearance. Ideal when the design system is strong and you want your own identity. Cost: you style everything.
- **Full kit with theming**: components already styled and themable by tokens. Ideal for internal tools, admin/dashboards, or when speed matters more than unique identity. Cost: risk of looking "out of the box" (mitigable, below).
- **Copy-in / code generated into your repo**: components that live in your code and you edit freely. Ideal for products with a strong custom design and a team that wants full control without maintaining primitives. Cost: updates are yours.
- **What the project already has** wins by default over the three above.

On native or cross-platform mobile, the equivalent is: system/framework components first, ecosystem library second, custom last.

## When you DO build custom

Only if at least one holds:

1. **The component IS the product** (the editor of an editor, the calendar of a scheduling app, the canvas of a design tool): there you invest, with keyboard + ARIA + complete states as part of the scope, not as a future improvement.
2. **No proven accessible option exists** for that pattern in the ecosystem (rare; verify before claiming it).
3. It's purely presentational with no interaction (a card, a badge, a layout): that's not a "control", build it with the tokens.

## Map the design system to the library (so it doesn't look "out of the box")

1. **Configure the library's theme with the tokens** (palette, type, radii, spacing) via its official theming mechanism — never by overriding CSS component by component.
2. **Change the telltale defaults**: factory primary color, default radius, default shadows and the default font are ALWAYS replaced by the design system's. A library with its defaults intact is tell #0 of a generic product.
3. **Create thin wrappers** for the 5–8 most-used components (Button, Input, Select, Dialog, Toast...) that fix the variants allowed by the design system. Screens consume the wrappers, not the library directly — so a token or library change touches a single place.
4. **Document in `docs/design-system.md`** which library was chosen, why, and the token→theme-variable table.
