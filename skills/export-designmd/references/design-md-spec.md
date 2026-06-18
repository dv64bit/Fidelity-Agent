# DESIGN.md — format reference

`DESIGN.md` is an open format introduced by Google Stitch (announced March 2026, open-sourced April 2026) and adopted across AI coding agents (Claude Code, Cursor, v0, Lovable, Bolt, Kiro). It makes a project's design system legible to LLMs as persistent repo context — "DESIGN.md is to design what AGENTS.md is to code conventions."

## Structure: two parts in one file

1. **YAML frontmatter** — typed, machine-readable design tokens. These are the *normative* values agents must use.
2. **Markdown body** — human/agent-readable rationale in canonical `##` sections. This is what JSON token files cannot provide: the *why* and the *when*.

The token system is inspired by the W3C Design Tokens (DTCG) spec — typed token groups and `{path.to.token}` reference syntax for cross-referencing values.

## Canonical section order (enforced)

Sections may be omitted, but any that are present must appear in this sequence (the `section-order` linter rule flags deviation):

1. Overview (or "Brand & Style")
2. Colors
3. Typography
4. Layout (or "Layout & Spacing")
5. Elevation & Depth
6. Shapes
7. Components
8. Do's and Don'ts

## Token group schemas (frontmatter)

```yaml
---
name: <design system / brand name>

colors:
  primary: "#2665fd"        # required (missing-primary rule)
  primary-hover: "{colors.primary}"   # {path.to.token} references allowed
  surface: "#0b1326"
  on-surface: "#dae2fd"
  # ...add semantic roles: secondary, muted, border, success, warning, danger, focus-ring

typography:                 # required (missing-typography rule)
  # 9–15 named levels is common. Each level may carry:
  # fontFamily, fontSize, fontWeight, lineHeight, letterSpacing, fontFeature, fontVariation
  headline:
    fontFamily: Geist
    fontSize: 2rem
    fontWeight: 600
    lineHeight: 1.2
  body:
    fontFamily: Geist
    fontSize: 1rem
    fontWeight: 400
    lineHeight: 1.5

spacing:                    # part of missing-sections rule
  sm: 8px
  md: 16px
  lg: 32px

rounded:                    # radius scale; part of missing-sections rule
  sm: 4px
  md: 8px
  lg: 12px

# Optional, where the system uses them:
# elevation / shadow steps, shape tokens, motion durations/easing
---
```

## Markdown body — what each section holds

- **Overview** — the personality/intent in concrete terms ("Architectural minimalism meets journalistic gravitas; premium matte finish"). Not just adjectives.
- **Colors** — each color with its hex *and its role and usage rule*: "Primary (#2665fd): CTAs and active states only — never two primaries per view." Usage context is what stops an agent applying a CTA color to decoration.
- **Typography** — families and purpose-named styles: "Headlines: Geist Semi-Bold — institutional, trustworthy. Body: Geist Regular 16px — long-form readability." Specify the family once at the top so the agent can't drift. Numbers, not "large font."
- **Layout** — grid model, container max-widths, margins/safe areas, the spacing rhythm (e.g. "strict 8px scale with a 4px half-step for micro-adjustments"), breakpoints.
- **Elevation & Depth** — how hierarchy is conveyed. If shadows: spread/blur/color per step. If flat: state the alternative (borders, color contrast).
- **Shapes** — radius usage, border conventions, pill vs fixed-radius rules.
- **Components** — canonical styling per primitive: "Buttons: rounded-lg, primary filled. Inputs: 1px border, surface-variant background." Pin the rules ("button radius 8px, always") to stop cross-session drift.
- **Do's and Don'ts** — specific, opinionated rules. This section most directly defeats the generic-AI aesthetic. Add it last and make every line concrete to this system.

## Linter rules (validate before delivering)

| Rule | Checks |
| --- | --- |
| `broken-ref` | `{path.to.token}` references all resolve |
| `missing-primary` | a primary color is defined |
| `contrast-ratio` | component background/text pairs meet WCAG AA |
| `orphaned-tokens` | no colors defined but never referenced |
| `missing-typography` | typography tokens present, not only colors |
| `missing-sections` | spacing and radius scales present |
| `section-order` | sections in the canonical order above |
| `token-summary` | informational count per token group |

## Placement & wiring

- File name: exactly `DESIGN.md`, at the **repository root**.
- Reference it from agent config so it loads on every UI task: a line in `CLAUDE.md` / `AGENTS.md`, a `.cursor/rules` entry, `global_rules.md`, or `.kiro/steering`.

## Exports (optional, for build pipelines)

The same tokens can export to: CSS custom properties (`:root { --colors-primary: … }`), a Tailwind v4 `@theme` block, and W3C DTCG-format JSON (for Style Dictionary, Tokens Studio, or any consumer of the community standard). `DESIGN.md` coexists with an existing token pipeline rather than replacing it.

## Minimal valid example

```markdown
---
name: Productivity App
colors:
  primary: "#2665fd"
  surface: "#0b1326"
  on-surface: "#dae2fd"
typography:
  headline: { fontFamily: Geist, fontSize: 2rem, fontWeight: 600 }
  body: { fontFamily: Geist, fontSize: 1rem, fontWeight: 400 }
rounded: { sm: 4px, md: 8px, lg: 12px }
spacing: { sm: 8px, md: 16px, lg: 32px }
---

## Overview
Minimal interface for a productivity app. Clean lines, high information density.

## Colors
- **Primary** (#2665fd): CTAs, active states.
- **Surface** (#0b1326): Backgrounds.

## Typography
- Headlines: Geist, semi-bold.
- Body: Geist, regular, 14–16px.

## Components
- Buttons: rounded-lg, primary filled.
- Inputs: 1px border, surface-variant background.

## Do's and Don'ts
- Do use primary only for the single primary action per view.
- Don't introduce new radii outside the rounded scale.
```
