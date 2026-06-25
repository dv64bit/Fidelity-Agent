# Conformance Playbook — implementing DESIGN.md with zero drift

The spine of this skill is one idea: the spec's tokens must become the values the code *references*, and the implemented code must be *diffed back* against the spec until there is no discrepancy. This file covers the mechanics.

## Table of contents
1. Parse the spec completely
2. Resolve references and build the conformance table
3. Translate to the codebase's token format (per stack)
4. Find and purge overrides (per stack)
5. The token-by-token conformance audit
6. Guardrails against future drift

## 1. Parse the spec completely

A `DESIGN.md` (Google Stitch open format) has two normative halves — read both:

- **YAML frontmatter** — typed token groups: `name`, `colors`, `typography` (per level: `fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `letterSpacing`, …), `spacing`, `rounded` (radius), and optionally elevation/shadow, shape, and motion. These are the hard values.
- **Markdown prose** — Overview, Colors, Typography, Layout, Elevation & Depth, Shapes, Components, Do's and Don'ts. The usage rules here are enforceable constraints (e.g. "primary only for the single highest-emphasis action"), not flavour text.

Capture both. The tokens drive the audit; the prose drives the usage rules you apply in components.

## 2. Resolve references and build the conformance table

DESIGN.md uses `{path.to.token}` reference syntax (inspired by the W3C DTCG spec). Before implementing:

- Resolve every reference to a concrete value (e.g. `primary-hover: "{colors.primary}"` → the hex of `colors.primary`).
- Detect unresolved references and reference cycles. If found, **stop and report** — the spec is internally broken and cannot be implemented faithfully until fixed.
- Flatten everything into a single table — the source of truth for Phase 3:

```
token name            | resolved value      | role
colors.primary        | #2665fd             | primary action
colors.surface        | #0b1326             | background
rounded.md            | 8px                 | default radius
typography.body.size  | 1rem                | body text
...
```

## 3. Translate to the codebase's token format (per stack)

Translate the table 1:1 — no rounding, no unit "normalisation" that changes the value, no interpretation.

- **CSS custom properties:** write resolved values into `:root` (and `[data-theme]`/dark scopes). `--color-primary: #2665fd;` etc.
- **Tailwind v4 (`@theme`):** `--color-primary`, `--font-*`, `--radius-*`, `--shadow-*` inside `@theme`.
- **Tailwind v3 (config):** `theme.extend.colors`, `fontFamily`, `borderRadius`, `boxShadow`, `spacing`, `fontSize` (with line-height/tracking tuples where the scale defines them).
- **CSS-in-JS:** the `ThemeProvider` theme object (styled-components / Emotion).
- **MUI:** `createTheme({ palette, typography, shape, shadows })`. **Chakra:** `extendTheme({ colors, fonts, radii, shadows })`.
- **shadcn/ui:** its CSS-variable theme (`--background`, `--primary`, `--radius`, …) — remap those and the whole system follows.
- **Fonts:** wire the exact families via `@font-face` or the framework's font loader; don't substitute a "close" font.

If no token architecture exists, create one here (CSS variables are the lowest-friction). This is the structural fix — components must reference tokens, or they will re-infer and drift.

## 4. Find and purge overrides (per stack)

Overrides are values that bypass the token system and silently win. Hunt them:

- **Raw colors:** grep for hex (`#[0-9a-fA-F]{3,8}`), `rgb(`, `rgba(`, `hsl(`.
- **Tailwind arbitrary values:** grep for `\[` inside class strings — `bg-[#...]`, `text-[#...]`, `rounded-[...]`, `p-[...]`, `gap-[...]`.
- **Inline styles:** `style={{ … }}` / `style="…"` with literal px/colors.
- **Magic numbers:** literal `px`/`rem` in CSS that should map to spacing/radius tokens.

For each finding:
- If it **exactly equals** a spec token's value → replace with the token (semantic where possible: `bg-primary`, `var(--color-primary)`).
- If it's a **near-miss** (close but not equal) → **flag, don't auto-change.** A near-miss is either a deliberate exception the user must confirm or a bug; silently snapping it both hides intent and manufactures the very "mystery drift" this skill exists to remove.
- Prefer **semantic** tokens in components (`--color-error`) over primitives (`--red-500`) or raw values.

## 5. The token-by-token conformance audit

This is the step the drifting workflow omits. After implementing:

1. **Re-extract implemented values** — read the token source back; where you can run the app, also read computed styles on representative components.
2. **Diff against the conformance table.** Per token, assign:
   - `on-spec` — referenced and resolves to the exact spec value.
   - `missing` — spec token not implemented.
   - `mismatch` — implemented value ≠ spec value.
   - `overridden` — a hardcoded value bypasses the token where it's used.
   - `orphaned` — token defined but never referenced.
3. **Score it.** Report on-spec count / total and override count. Target: **100% on-spec, 0 overrides**.
4. **Fix and re-audit** every non-`on-spec` row. Loop until clean. "Mostly matching" is not done.

Sample report shape:

```
Conformance: 41/43 on-spec (95%) · 2 overrides · 0 unresolved refs
  mismatch   rounded.md        impl 6px vs spec 8px        src/components/Card.tsx
  overridden colors.primary    bg-[#3b82f6] bypass         src/components/CTA.tsx
→ fixing 2 findings, re-auditing…
Conformance: 43/43 on-spec (100%) · 0 overrides ✓
```

## 6. Guardrails against future drift

100% today regresses the moment someone hardcodes a value. Offer:

- **Lint rule** — Stylelint or ESLint design-token rule (e.g. no-hardcoded-colors / no-arbitrary-values) so raw values fail lint and edits must use tokens.
- **Agent pointer** — a line in `CLAUDE.md` / `AGENTS.md` / `.cursor/rules`: "All UI must use the tokens defined in DESIGN.md; never hardcode colors, spacing, or radii."
- **CI gate (optional)** — re-run the conformance audit in CI and fail on regression, mirroring how design-token linting is enforced at pre-commit and CI stages.
