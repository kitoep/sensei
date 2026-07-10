# Anti-generic — catalog of AI-design "tells" and corrective rules

A generic design isn't ugly: it's interchangeable. It looks like a thousand other products and conveys none of the design system's adjectives. This file loads WHENEVER visible UI is produced. Use it at two moments: while designing (to avoid) and before delivering (self-audit).

## Catalog of tells (if one appears, fix it)

| # | "An AI made this" tell | Corrective rule |
|---|---|---|
| 1 | Purple/blue gradient (or any gradient) as decoration by default | Flat color from the design system. Gradient only if the chosen visual direction explicitly defines it as distinctive |
| 2 | Centered hero: big title + subtitle + 2 buttons + 3 feature cards below | Break the symmetry: asymmetric layout, real product content (screenshot, live data, demo) instead of filler cards |
| 3 | Emojis as icons (🚀 ✨ 💡 in features/titles) | A consistent icon set from ONE library (the one the project uses). Zero emojis in UI |
| 4 | Large, diffuse shadows on every card | Elevation per the design system: 2–3 levels max; consider thin borders instead of a shadow |
| 5 | Giant, uniform border-radius on everything (buttons, cards, inputs, modals) | Design-system radii per element type; large containers take a smaller or proportional radius |
| 6 | Placeholder copy: "Unlock the power of...", "Take your X to the next level", lorem ipsum | Real microcopy from the product's domain, written for the intake's target. If you don't know the real content, ask — copy IS design |
| 7 | 5 shades of gray mixed with no intent | Only the neutrals from the design system's scale |
| 8 | Everything centered (titles, paragraphs, lists, forms) | Left-aligned by default for content and forms. Centered only for short ceremonial moments (empty state, confirmation) |
| 9 | Uniform density: all airy or all cramped, no hierarchy | Density per target (see below) and VARIED: what matters breathes, what's secondary compacts |
| 10 | Primary color scattered all over the screen (buttons, links, icons, borders, badges) | Primary ≤10% of the surface; a typical screen has ONE dominant primary element |
| 11 | Cards inside cards inside cards | At most one level of container with a background/border; group with space and typography, not more boxes |
| 12 | Data table/list with the same visual weight on every column | Prioritize: the column the user is looking for goes first and heavier; metadata in secondary text |

## Construction rules (positive)

1. **One distinctive, consistent decision > ten effects.** Choose ONE memorable element from the design system (a typeface with character, an unusual use of color, a border style) and apply it with discipline across the product. That's identity; accumulated effects are noise.
2. **Hierarchy is built with size, weight and space — not more colors.** Before adding a color to make something stand out, try: more size, more weight, more space around it. Color is the last resort and comes from the design system.
3. **Every screen answers "what do I want the user to do here?" with ONE dominant element.** Write it in a sentence before mocking up. Two calls to action with the same weight = no decision made. Everything else on the screen is visually subordinate to that element.
4. **Density per target** (from the intake): heavy-use/expert tool → dense, base type 14, more rows visible; consumer/occasional product → airy, base 16, one idea per view. Never "default density".
5. **Real content rules.** Design with real or realistic domain data (names, figures, long and short texts). Design that only works with "John Doe — Lorem ipsum" is broken.

## Self-audit before delivering (mandatory checklist)

Run the screen through these questions; if any fails, fix it before showing the user:

- [ ] Zero tells from the table (go through all 12)?
- [ ] Can I name the screen's dominant element, and is it visually obvious?
- [ ] Do all values (color, size, space, radius) come from design-system tokens?
- [ ] Is the copy real and domain-specific (zero placeholder)?
- [ ] Does it convey the 3 intake adjectives? (name them and justify in one line)
- [ ] Could a user tell this product apart from a generic template in a logo-less screenshot?
