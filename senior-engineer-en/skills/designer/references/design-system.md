# Design system first — how to build it WITH the user

The design system is built in dialogue with the user BEFORE the first screen. It's not a bureaucratic document: it's the visual decisions made once so you don't re-decide them (badly) on every screen. Final output: a file in the project (by default `docs/design-system.md`) that every future screen respects as a contract.

## Phase 1 — Identity intake (questions for the user)

Ask these one at a time or grouped in a single message with options; don't advance to tokens without answers. The ones marked ⭐ are the 4 essentials of the quick version.

1. ⭐ **Who is the end user and in what context do they use this?** — In a hurry or at ease? Mobile or desktop? An expert who uses it 8 hours a day or an occasional novice? This decides density, type size and tolerance for a learning curve. A dashboard for expert operators is dense and sober; an onboarding for novices is airy and guided.
2. ⭐ **Which 3 adjectives should the product convey?** — Ask for exactly 3 (e.g. "reliable, modern, warm"). Every later visual decision is validated against them: does this color/type/spacing convey those adjectives?
3. ⭐ **References you like, and WHY?** — Ask for 1–3 products/sites the user likes and what they like about each (the density? the color? the type?). The "why" is what's valuable; a reference without a why produces shallow imitation.
4. ⭐ **Existing brand or from scratch?** — If there's a logo/brand colors, they're inherited as a constraint. If not, you'll propose a palette from the adjectives.
5. **Dark mode?** — Decided NOW, not later: adding dark mode after the fact doubles the cost. If the answer is "someday", define the tokens as semantic variables from the start (`--surface`, `--text-primary`) even if only light mode is implemented.
6. **Content language(s) and tone?** — Affects text lengths (German breaks layouts English doesn't) and microcopy.

## Phase 2 — Visual directions (BEFORE tokenizing)

With the intake done, present the user **2–3 distinct visual directions** to choose from. Each direction is described in 4–6 lines: personality, tentative palette (name the colors), tentative typography, density, one distinctive element. Example format:

> **Direction A — "Trusted clinic":** warm neutrals + a deep green as primary; serif type for headings (authority) and sans for UI; generous spacing; distinctive: cards with a thin border instead of a shadow.
> **Direction B — "Pro tool":** cool near-white gray background, saturated blue primary used with GREAT restraint; a single sans in 2 weights; dense; distinctive: tabular type for numbers.

If the medium allows, materialize the directions in a static HTML sample screen so the user SEES instead of imagining. The user picks (or mixes); only then do you tokenize.

## Phase 3 — Concrete tokens

Document ALL these groups with exact values. An undefined token = a decision that will be made badly in a hurry later.

### Color
- **Primary**: 1 color, with its usage scale (hover, pressed, subtle/bg). Rule: the primary appears on ≤10% of the surface of a typical screen; its job is to signal the main action, not to decorate.
- **Neutrals**: a 5–8 step scale (background, surface, border, secondary text, primary text) chosen with intent — decide whether they're cool, warm or pure grays based on the adjectives.
- **Semantic**: success, error, warning, info. Never use the semantic color for decoration nor the primary for states.
- **Mandatory AA check**: every defined text/background pair must meet WCAG AA (4.5:1 normal text, 3:1 large text and UI components). Verify the ratios by computing them, not "by eye"; document the ratio next to the pair.

### Typography
- At most 2 families (1 is valid and usually better). Note the fallback stack.
- **Fixed size scale** (no ad-hoc sizes): define 6–8 steps with a name and use, e.g. `xs 12 / sm 14 / base 16 / lg 18 / xl 22 / 2xl 28 / 3xl 36` with a line-height per step. Every screen uses ONLY these steps.
- Allowed weights (typically 2–3: regular, medium, bold) and where each is used.
- Numbers in data/tables: use the tabular variant (`font-variant-numeric: tabular-nums`) if the product shows figures.

### Spacing, radii, shadows, borders
- **Fixed spacing scale on base 4 or 8**: e.g. `4, 8, 12, 16, 24, 32, 48, 64`. No margin/padding outside the scale is allowed.
- **Radii**: define 2–3 (e.g. `sm 4, md 8, full`) and what each applies to. A giant uniform radius everywhere = a tell of generic design.
- **Shadows**: define 2–3 elevation levels with exact values and when each is used (or decide "this product uses borders, not shadows" — a valid, distinctive direction).
- **Borders**: standard width and color.

### Dark mode (if applicable)
- Semantic tokens with a value per mode. Rule: in dark mode you don't invert colors, you re-pick them (dark neutrals carry less contrast between steps; shadows are replaced by surface difference).

## Phase 4 — Document and close

1. Write `docs/design-system.md` with: intake summary (target, adjectives, chosen direction), all tokens with values, and the usage rules (primary ≤10%, mandatory spacing scale, verified AA pairs).
2. If the project uses a styling system (CSS variables, Tailwind config, library theme), **materialize the tokens there in the same PR** — the doc and the code never diverge.
3. Ask for explicit user approval of the document. Only with approval is the gate to writing screens opened.

## Ongoing governance

- A new screen that "needs" a color/size/space outside tokens → first discuss adding the token (with the user), then use it. Never a magic inline value.
- Changes to the design system are made in its file + its materialization in code, never by overriding styles locally in a screen.
