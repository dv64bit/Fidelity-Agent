# Application Playbook — re-theming an existing build at the token layer

The goal: adopt a reference's design language by changing the **smallest, highest-leverage surface** (usually the token/theme source), so the new look propagates everywhere without touching application logic. Edit styling sources, not behaviour.

## Table of contents
1. Find the source of truth
2. Strategy by stack
3. The presentation vs logic boundary
4. Rollout discipline
5. Contrast & accessibility safeguards

## 1. Find the source of truth

Before editing anything, locate where styling is *defined* vs where it's *consumed*. Change definitions; leave consumption alone. Look for, in priority order:

1. **Token/theme files** — `tailwind.config.{js,ts}`, a Tailwind v4 `@theme` block, `theme.ts/js`, a `tokens.*` file, Style Dictionary / DTCG JSON, an existing `DESIGN.md`.
2. **Global CSS** — `:root { --… }` custom properties, base layers, font `@import`/`@font-face`.
3. **Component variant configs** — `cva` / `tailwind-variants` / styled-component theme objects that define primitive styling.
4. **Per-component styling** — className strings, inline styles, CSS modules (the fallback surface for values the token layer can't reach).

The closer to #1 you can apply the change, the more accurate and the safer it is.

## 2. Strategy by stack

**Tailwind (config-based, v3):** Remap the theme in `tailwind.config` — `theme.extend.colors`, `fontFamily`, `borderRadius`, `boxShadow`, `spacing`. Components using semantic classes (`bg-primary`, `rounded-lg`) inherit the new language with zero component edits. Replace any hardcoded arbitrary values (`bg-[#3b82f6]`, `rounded-[6px]`) with token classes as a cleanup pass.

**Tailwind v4 (`@theme`):** Edit the `@theme` block's CSS variables (`--color-*`, `--font-*`, `--radius-*`, `--shadow-*`). Same propagation benefit.

**CSS custom properties:** Rewrite the values in `:root` (and any dark-mode/`[data-theme]` scopes). Everything referencing `var(--…)` updates automatically. This is the cleanest possible re-theme.

**CSS-in-JS (styled-components / Emotion):** Edit the `ThemeProvider` theme object. Components reading `props.theme.*` adopt the new values. Don't rewrite the styled component definitions unless a value is hardcoded.

**Component libraries:**
- *shadcn/ui:* the theme lives in CSS variables (`--background`, `--primary`, `--radius`, etc.) — remap those. Because shadcn components live in the repo and read these variables, the whole system re-themes from one file.
- *MUI:* edit `createTheme({ palette, typography, shape, shadows })`.
- *Chakra:* edit the theme `extendTheme({ colors, fonts, radii, shadows })`.

**Plain HTML/CSS:** centralise into CSS variables first if values are scattered, then theme via those variables.

## 3. The presentation vs logic boundary

| Change freely (presentation) | Never change (application) |
| --- | --- |
| Color tokens / palette | Business & component logic |
| Typography (family, scale, weight) | State management, hooks, effects |
| Spacing / radius / shadow tokens | Data fetching, API calls |
| Utility classes, stylesheets, CSS vars | Routing & navigation flow |
| Component *styling* props / variants | Information architecture |
| Icon style, visual density | Component contracts (non-styling props) |
| Animation / transition styling | Event handlers, conditional logic |
| | Copy / content / data |
| | A11y semantics (roles, labels, focus order) |
| | DOM structure that carries behaviour |
| | Behaviour tests |

If a styling goal genuinely needs a structural/logic change to land (e.g. the reference's layout implies a different component composition), do not silently refactor — describe the trade-off and let the user decide.

## 4. Rollout discipline

- **Plan, then apply.** Present the file list, token map, and order before destructive edits.
- **Pilot first.** Apply to one representative component (e.g. a primary button in default/hover/disabled, or one screen), render, compare to the reference language, fix gaps — then roll out. Catching a mismatch once beats fixing it across ten screens.
- **Small diffs.** Prefer token-source edits over wholesale file rewrites. Reviewable changes keep the user in control and make regressions obvious.
- **Migrate completely.** Don't leave islands of the old theme (stray hardcoded colors, an un-updated component). Grep for arbitrary values and old hex codes to find stragglers.

## 5. Contrast & accessibility safeguards

A palette swap can quietly break legibility. After applying:

- Re-check WCAG AA contrast for text-on-surface and key UI pairings; if a faithful brand color fails contrast in a given role, flag it rather than silently altering the brand.
- Confirm focus rings remain visible against the new surfaces.
- Confirm interactive states (hover/active/disabled) remain distinguishable.
- Leave roles, labels, and focus order exactly as they were — restyling must not degrade accessibility semantics.
