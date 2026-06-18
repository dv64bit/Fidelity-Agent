---
name: export-designmd
description: Generate a spec-compliant DESIGN.md file from an existing codebase, so the project's real design language becomes a portable, drop-in context file that AI coding agents read before generating UI. Use whenever the user wants to "export a DESIGN.md", "create a design.md", "make our design system AI-readable", "produce a design token file for Cursor/Claude Code", "document the design system as markdown", or turn an existing front-end into a reusable design spec. Triggers on the /export-designmd command. DESIGN.md is the open format introduced by Google Stitch (YAML token frontmatter + Markdown rationale); this skill emits it correctly — canonical section order, typed tokens, lint-clean — rather than improvising a format. Use extract-design-language first if the design language still needs to be analysed; use this skill to render that analysis into the official DESIGN.md spec.
---

# Export DESIGN.md

Produce a `DESIGN.md` that is faithful to the codebase and compliant with the open spec — not a plausible-looking markdown file that drifts from the format or from the actual tokens. `DESIGN.md` is the format Google Stitch introduced and open-sourced: it pairs **YAML frontmatter** (machine-readable, typed design tokens — the normative values) with a **Markdown body** (human/agent-readable rationale — why those values exist and when to apply them). The combination is the point: JSON tokens alone can't express "use primary only for the single highest-emphasis action on a screen," and prose alone can't be enforced. Read `references/design-md-spec.md` for the full format, section order, token schema, and linter rules before writing the file.

## Why this matters

Without a `DESIGN.md` in the repo, a coding agent regenerates UI from training-data priors every prompt — the generic beige "AI aesthetic," inconsistent radii, drifting button heights. With one at the repo root, the agent reads it first and produces UI that matches the product. So the file is only as good as its fidelity to the real system: a `DESIGN.md` that lists tokens the codebase doesn't actually use is worse than none, because it teaches the agent wrong values confidently.

## Process

### Step 1 — Extract from source of truth (do not guess)
Derive tokens from the codebase itself, in this priority order. If the design language hasn't been analysed yet, run the `extract-design-language` skill (Mode C, codebase) first and consume its brief.

1. **Token/theme files** — `tailwind.config.*`, Tailwind v4 `@theme` blocks, `theme.ts/js`, CSS custom properties in `:root`, Style Dictionary / DTCG JSON, Tokens Studio exports.
2. **Global CSS** — base layers, font imports, resets.
3. **Component variants** — `cva` / `tailwind-variants` / styled definitions for primitives (Button, Input, Card…), which hold the real radii, heights, and state styling.
4. **Actual usage** — grep how tokens are applied so you separate load-bearing tokens from orphans.

Record where each value came from. Prefer values read from source over values inferred from rendered output.

### Step 2 — Map to the spec's typed token groups
Organise extracted values into the spec's token groups in the YAML frontmatter: `name`, `colors`, `typography`, `spacing`, `rounded` (radius), and where applicable elevation/shadow and shape groups. Use the spec's typed shapes (e.g. typography entries carry `fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `letterSpacing`). Use `{path.to.token}` reference syntax for cross-references (e.g. a hover color referencing its base). See the reference file for exact schemas.

### Step 3 — Write the Markdown body in canonical section order
The spec defines eight sections that, when present, **must** appear in this order (sections may be omitted, but order is enforced by the linter's `section-order` rule):

1. **Overview** (a.k.a. "Brand & Style")
2. **Colors**
3. **Typography**
4. **Layout** (a.k.a. "Layout & Spacing")
5. **Elevation & Depth**
6. **Shapes**
7. **Components**
8. **Do's and Don'ts**

For each section, write the *rationale*, not a re-dump of the tokens: roles and usage rules for colors ("primary — highest-emphasis action only; never two per view"), purpose-named type styles ("display/xl — page titles only"), the spacing rhythm and grid strategy, the elevation-vs-flat approach, canonical component styling, and a specific, opinionated Do's/Don'ts list. The Do's and Don'ts section is where you kill the generic-AI aesthetic — make the rules concrete to *this* system.

### Step 4 — Lint before delivering
Validate against the spec's linter rules so the file is clean on arrival:

- `broken-ref` — every `{path.to.token}` reference resolves.
- `missing-primary` — a primary color is defined.
- `contrast-ratio` — component background/text pairs meet WCAG AA.
- `orphaned-tokens` — no colors defined but never referenced (or justify them).
- `missing-typography` — typography tokens exist, not just colors.
- `missing-sections` — spacing and radius scales are present.
- `section-order` — sections follow the canonical order above.
- `token-summary` — sanity-check the count per group.

Report any rule you could not satisfy and why (e.g. a contrast pair that genuinely fails in the source — flag it rather than silently "fixing" the brand).

### Step 5 — Place and wire it up
- Write the file as exactly `DESIGN.md` at the **repository root** (this is where agents look; do not rename or bury it).
- Tell the user to reference it from their agent config so it's read on every UI task: add a pointer in `CLAUDE.md` / `AGENTS.md`, a `.cursor/rules` entry, or `global_rules.md`, e.g. "For any UI work, follow the tokens and rules in DESIGN.md."
- Offer the optional exports the spec supports for projects with a build pipeline: CSS custom properties (`:root`), a Tailwind v4 `@theme` block, and W3C DTCG-format JSON for Style Dictionary / Tokens Studio. `DESIGN.md` coexists with an existing token pipeline — it doesn't replace it.

## Output

Deliver the `DESIGN.md` file itself (written to the repo root, or to the working directory with a clear note on where it belongs), plus a short summary of: what was extracted and from where, any linter findings, and the one-line config snippet to wire it into the user's agent. Keep the file faithful, typed, ordered, and lint-clean.

## Guardrails
- Fidelity over completeness: a small file of real tokens beats a large file of invented ones.
- Don't flatten a distinctive system into default tokens. If the codebase uses Geist at a compact 8px rhythm with hairline borders, the file should say exactly that.
- The spec is young (alpha) and evolving; the section order and linter rules above reflect the current open spec. If the user has pinned a specific spec/CLI version, follow theirs.
