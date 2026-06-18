---
name: extract-design-language
description: Extract the complete design language from a reference — a screenshot/image, a live URL/website, or an existing codebase — into a structured, reusable design-language brief (tokens + patterns + rationale). Use whenever the user wants to "extract the design language", "capture the design system", "reverse-engineer this UI's styling", "figure out the tokens behind this", "document how this looks", or analyse what makes a reference look the way it does, from any source. Triggers on the /extract-design-language command and on any request to derive design intent, tokens, or patterns from an image, site, or code. Use this when the goal is to UNDERSTAND and capture a design language for reuse; if the goal is specifically to emit a spec-compliant DESIGN.md file from a codebase, use export-designmd instead (it builds on this analysis).
---

# Extract Design Language

Capture the *design language* of a reference — not just a list of colors, but the system of decisions and the intent behind them, in a form precise enough to reapply elsewhere. The output is a portable brief: discrete tokens for the machine, plus the qualitative reasoning that tells a future agent *why* those values exist and *when* to use them. Tokens without rationale produce technically-correct-but-soulless reuse; rationale without tokens produces vibes nobody can implement. You need both.

## Pick the source mode

The user's reference is one of three things. Confirm which, then follow that mode. If the reference is genuinely missing (a prompt implies an image but none is attached), check and ask.

### Mode A — Image / screenshot

Read the image forensically. This overlaps the analysis discipline in the `ss-replicate` skill — read its `references/analysis-pass.md` if available and apply the same element-by-element rigour. Extract tokens by *inference* from pixels, and explicitly mark confidence: pixel-derived values are estimates, not ground truth. Flag anything a static image cannot reveal (states, motion, responsive behaviour).

### Mode B — Live URL / website

Fetch the page and inspect the *actual* styling rather than guessing from the rendered look — this is ground truth, not inference, so prefer it strongly over eyeballing.

- Read computed styles, CSS custom properties (`:root { --… }`), and any Tailwind/utility classes present.
- Pull the real font stack, the actual hex/oklch values, the spacing and radius scales, the shadow definitions.
- Identify the framework/design-system fingerprint (shadcn/ui, Material, Chakra, Bootstrap, a bespoke system) from class names and structure.
- Capture interaction styling where reachable (hover/focus rules in CSS).
- Respect copyright and IP: extract the *system* (tokens, patterns), do not wholesale-copy proprietary content, brand marks, or copy.

### Mode C — Existing codebase

Inspect the source of truth directly. Look in this priority order:

1. **Token/theme sources** — `tailwind.config.*`, a Tailwind v4 `@theme` block, `theme.ts/js`, CSS custom properties, Style Dictionary / DTCG JSON, Tokens Studio exports, an existing `DESIGN.md`.
2. **Global styles** — `globals.css`, `app.css`, reset/base layers, font imports.
3. **Component library** — the primitive components (Button, Input, Card…) and their variant definitions (e.g. `cva`/`tailwind-variants` configs), which encode the real radii, heights, and state styles.
4. **Usage patterns** — grep for how tokens are actually applied across the app to distinguish defined-but-unused tokens from load-bearing ones.

Report tokens that are *defined* vs tokens that are *actually used* — orphaned tokens are noise and shouldn't be presented as part of the live language.

## What to extract (every mode)

Capture each of these. For inferred (image) sources, tag confidence; for code/URL sources, cite where the value came from.

- **Color system** — every color with hex/oklch, its semantic role (surface, raised surface, primary, secondary, muted text, border, semantic states, focus ring), and relationships (is `primary-hover` a fixed darken of `primary`?).
- **Typography** — font families and their roles, the full type scale (size/weight/line-height/letter-spacing per named style), and rules (e.g. "display sizes for page titles only").
- **Spacing & layout** — base unit (4px vs 8px grid), the spacing scale, container widths, grid conventions, breakpoints.
- **Shape** — radius scale, border conventions, the flat-vs-elevated strategy.
- **Elevation & depth** — shadow steps with values, or the non-shadow method (borders, contrast) for flat systems.
- **Motion** — durations, easing curves, what animates and what deliberately doesn't (if observable).
- **Component patterns** — the recurring primitives and their canonical styling/variants.
- **Design philosophy** — the qualitative through-line: density (airy vs compact), tone (playful vs institutional), contrast strategy, what the system optimises for. This is the part that prevents generic reuse — name the personality in concrete terms, not adjectives alone ("compact 8px rhythm with 1px hairline borders and near-zero elevation" beats "clean and minimal").

## Output format

Produce a structured brief with these sections, in this order:

```
# Design Language: [source name]

## Source & confidence
[What was analysed, which mode, and how reliable the values are —
inferred from pixels vs read from code/CSS.]

## Overview / Personality
[The qualitative through-line in concrete terms.]

## Color
[Table: token | value | role | notes]

## Typography
[Families, then a table: style | family | size | weight | line-height | tracking | usage]

## Spacing & Layout
[Base unit, scale, containers, breakpoints.]

## Shape
[Radius scale, border conventions, flat-vs-elevated strategy.]

## Elevation & Depth
[Shadow steps with values, or the flat alternative.]

## Motion
[Durations, easing, what does/doesn't animate. Omit if unobservable.]

## Component Patterns
[Per primitive: canonical styling and variants observed.]

## Do's and Don'ts
[Concrete usage rules that keep reuse faithful and non-generic.]
```

Keep the brief implementation-ready: a downstream agent (or the `export-designmd` skill) should be able to consume it directly. If the user's end goal is a drop-in `DESIGN.md` for a codebase, hand off to `export-designmd`, which renders this analysis into the official spec format.

## Guardrails

- Distinguish inference from fact. Never present a pixel-estimated hex as if it were read from source.
- Extract the system, not the content. Tokens and patterns are fair to capture; proprietary copy, imagery, and brand marks are not — flag and placeholder them.
- Avoid the generic-AI failure mode: do not flatten a distinctive language into the usual "Inter / slate-900 / rounded-lg / shadow-md" defaults unless that is genuinely what the source uses. Name what makes *this* one specific.
