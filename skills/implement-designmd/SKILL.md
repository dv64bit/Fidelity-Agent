---
name: implement-designmd
description: Implement an existing DESIGN.md design-system file into a codebase with verifiable, zero-drift fidelity — wiring its tokens into the project's single source of truth, purging hardcoded overrides, and proving conformance with a token-by-token audit. Use whenever the user has a DESIGN.md (or DESIGN.md-style token spec) and wants it applied to a web app or codebase "exactly", "with 100% accuracy", "faithfully", or "without discrepancy", or says the design language "isn't being implemented the same" / "drifts" when applied. Triggers on the /implement-designmd command. This is the return leg of export-designmd (export produces the spec; this implements it). It is a PRESENTATION-LAYER transformation — it changes tokens, theme sources, and component styling, never application logic, state, routing, data, or IA. Distinct from apply-design-language, whose source is a reference image/site; here the source is specifically a DESIGN.md spec and the defining feature is the conformance audit that guarantees the implemented code matches the spec.
---

# Implement DESIGN.md

Take a `DESIGN.md` and make the codebase conform to it **exactly** — then prove it. The failure this skill exists to kill is *drift*: an agent reads a `DESIGN.md`, produces something "close but not quite," and nobody verifies. The result reads almost on-brand but never matches. The cure is structural, not stylistic: stop letting the code *infer* the design language and force it to *reference* the spec, eliminate every value that silently overrides the spec, and finish with a token-by-token conformance audit that has to come back clean.

## Why DESIGN.md drifts when implemented (and the actual fix)

When visual decisions are hardcoded and scattered across components, any tool — human or AI — infers them inconsistently, so each pass lands close to the spec but not identical. A `DESIGN.md` used as a *document to eyeball* doesn't fix this; the agent still re-infers values every time. It only stops drifting when the spec's tokens become the **single source of truth that the code literally references** (CSS variables, a Tailwind theme, a theme object), so a value can't be "almost right" — it's the token or it's a bug. Three things must all be true for zero drift:

1. **The spec is normative, not inspirational.** Every token in the frontmatter is a hard value the code must use verbatim. No rounding, no "this is basically the same," no improving.
2. **There are no overrides.** Hardcoded raw values (`#3b82f6`, `bg-[#3b82f6]`, `rounded-[6px]`, inline px) bypass the token system and silently win. They must be found and removed.
3. **Conformance is verified.** The implemented code is diffed back against the spec, token by token, until there is zero discrepancy. Unverified is unfinished.

## The boundary: presentation only

This changes how the app *looks*, never how it *works*. You may change token/theme sources, CSS, and component *styling*; you may not change business logic, state, hooks, data fetching, API calls, routing, IA, component contracts (non-styling), event handlers, conditional logic, or copy. Do not add features, states, or content. If a faithful token can't land without a structural change, flag the trade-off rather than refactoring behaviour.

## Process

### Phase 0 — Parse the spec and build the conformance table
- Locate and fully parse the `DESIGN.md`. Read both halves: the **YAML frontmatter tokens** (the normative values) and the **Markdown prose** (usage rules, Do's and Don'ts — also enforceable, not decoration). See `references/conformance-playbook.md` for the spec structure.
- **Resolve every `{path.to.token}` reference** to its concrete value and expand all token groups into a single flat **conformance table**: `token name → resolved value → role`. This table is the source of truth for the audit. If a reference doesn't resolve or two references form a loop, stop and report it — the spec is broken and faithful implementation is impossible until it's fixed.
- Confirm the codebase's styling architecture and where its token source of truth lives (or whether one exists). Confirm scope and restate the presentation-only boundary.

### Phase 1 — Make the spec the single source of truth
Translate the conformance table into the codebase's exact token format, **1:1 with no interpretation**:

- **CSS variables** → write the resolved values into `:root` (and theme scopes).
- **Tailwind v4** → write them into the `@theme` block. **Tailwind v3 / config** → map into `theme.extend` (colors, fontFamily, borderRadius, boxShadow, spacing, etc.).
- **CSS-in-JS / component lib** → write the theme object (`ThemeProvider`, `createTheme`, `extendTheme`), or for shadcn/ui the CSS-variable theme it already reads.

If the codebase has **no token architecture**, create one — this is the real fix, not a nicety. Without a referenced source of truth the code will keep re-inferring and drift will return. Wire fonts as real `@font-face`/imports (exact families from the spec), plus the full type scale, spacing, radius, elevation, and motion tokens. Read the playbook for per-stack specifics.

### Phase 2 — Purge overrides and wire consumption
- Scan the whole codebase for values that **bypass** the token system: raw hex/rgb/hsl, Tailwind arbitrary values (`[...]`), inline `style` px, magic numbers. The playbook has grep patterns per stack.
- For each, replace **exact matches** with the corresponding token. **Never silently snap a near-miss** to a token — a value that's close but not equal to a spec token is either a deliberate exception or a bug; flag it for a decision instead of hiding it. Auto-fixing near-misses is itself a source of "mystery drift."
- Ensure components consume **semantic** tokens (`--color-primary`, `bg-primary`), not raw values or primitives.
- Apply the prose usage rules and Do's/Don'ts as constraints (e.g. "primary only for the single highest-emphasis action"; "no radii outside the scale").

### Phase 3 — Conformance audit (the differentiator — this is what guarantees no discrepancy)
This phase is non-negotiable; it's the step the drifting workflow skips.

1. **Re-extract the implemented values** from the codebase: read back the token source, and where you can render, read computed styles too.
2. **Diff against the conformance table, token by token.** For each spec token assign a status: `on-spec` (referenced and resolves to the exact value), `missing` (spec token not implemented), `mismatch` (implemented value ≠ spec value), `overridden` (a hardcoded value bypasses the token in use), or `orphaned` (token defined but unreferenced).
3. **Report a conformance result** — the per-token table plus a score. The target is **100% on-spec with zero overrides**.
4. **Fix every discrepancy and re-audit.** Loop until the table is clean. Do not declare done at "mostly matching."
5. Where rendering is available, add a visual spot-check against the spec's stated personality (density, contrast, tone) — but the token-level audit is the spine; the visual check is corroboration.

### Phase 4 — Lock it in so it stays at 100%
Drift returns the next time someone hardcodes a value. Offer (and, if the user agrees, add) guardrails:
- A lint rule (Stylelint / ESLint design-token rule, e.g. no-hardcoded-colors) that blocks raw values so future edits must use tokens.
- A pointer from `CLAUDE.md` / `AGENTS.md` / Cursor rules declaring `DESIGN.md` the normative source for all UI work.
- Optionally a CI check that re-runs the conformance audit and fails on regression.

## Output format
1. **The diffs** — token source first (the single source of truth), then override replacements, as reviewable changes.
2. **The conformance report** — the token-by-token table with statuses and a score, ending at 100% on-spec / zero overrides, or with any remaining discrepancies listed explicitly with reasons.
3. **Boundary confirmation + guardrails** — what was not touched (logic, routing, data, content), and the anti-drift guardrails offered or added.

## Anti-patterns
- Treating `DESIGN.md` as inspiration to approximate rather than a normative spec to reference.
- Implementing values by eyeballing instead of wiring them into a referenced source of truth.
- Leaving hardcoded values that silently override the tokens.
- Auto-snapping near-misses to tokens (hides intent and causes mystery drift) — flag them.
- Rounding or "improving" spec values.
- Declaring done without a token-by-token conformance audit at 100%.
- Touching application logic, routing, data, or content to land a visual change.
