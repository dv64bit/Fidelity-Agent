---
name: apply-design-language
description: Apply a reference's design language to an existing codebase — restyle the UI only, never the functionality. Use when the user wants to "apply this design language to my app", "restyle my build to look like this", "reskin my codebase", or adopt a reference's aesthetic without changing logic. Triggers on /apply-design-language. Changes tokens, color, type, spacing, shape, elevation, motion, and component styling only. Never touches business logic, state, APIs, routing, or IA. Distinct from ss-replicate (clones one screen as new code) — this restyles the whole existing app. Prefer when the codebase already exists and only its visual identity should change.
---

# Apply Design Language

Take the design language of a reference and make the user's existing application *wear* it — same app, same screens, same behaviour, new visual identity. The accuracy bar is high: faithfully adopt the reference's color, type, spacing, shape, elevation, and motion. The safety bar is equally high: nothing about how the application *works* may change.

## The governing boundary: presentation only

This is the rule the whole skill is built around. You are changing how the app *looks*, never how it *works*.

**You MAY change** (the presentation layer): design tokens (color, typography, spacing, radius, shadow, motion), theme/CSS variables, utility classes and stylesheets, component *styling* (className/style/variant props), font choices, icon style, visual density, and animation/transition styling.

**You MUST NOT change** (the application layer): business/component logic, state management, hooks, data fetching, API calls, routing, navigation flow, information architecture, component contracts/props that aren't styling, event handlers, conditional rendering logic, copy/content, accessibility *semantics* (roles, labels, focus order), tests for behaviour, or the DOM structure where structure carries behaviour. Do not add features, states, sections, or content the user did not ask for — a common failure mode is an agent "helpfully" adding hover states or copy that weren't requested. Restyling is the job; additions are scope creep.

If a visual change would require a structural or logic change to land correctly, stop and surface the trade-off to the user rather than silently refactoring behaviour.

## Why apply at the token layer (the accuracy + safety unlock)

The highest-leverage and lowest-risk way to re-theme is to change the **token source once** and let it propagate, rather than editing each component. In a Tailwind/CSS-variable/themed codebase you define theme variables in one place (e.g. `tailwind.config.*`, a `@theme` block, `:root` custom properties, or `theme.ts`) and every component that consumes those tokens adapts automatically — without touching component logic. This is simultaneously the most *accurate* path (consistent propagation, no missed instances) and the *safest* (you edit styling sources, not behaviour). Read `references/application-playbook.md` for how to do this across stacks and where to fall back to component-level styling.

Specificity is what separates a faithful adoption from generic AI slop. Apply **exact** values — exact hex/oklch, exact font names, pixel/rem spacing and radius — not vague descriptors like "dark" or "modern sans." Vague input yields the aggregate Tailwind-template look; precise tokens yield the reference's actual identity.

## Process

### Phase 0 — Intake
- Confirm the reference is actually present (an implied image may not be attached — check).
- Identify the existing build's **styling architecture**: framework, styling system (Tailwind config? CSS variables? CSS-in-JS? a component library like shadcn/ui, MUI, Chakra?), and where the token/theme source of truth lives. This determines the application strategy. Ask only what you can't discover by inspecting the codebase.
- Confirm **scope**: the whole app, a section, or a single screen. Default to piloting on one screen/component first (see Phase 3).
- Restate the boundary: "I'll restyle the UI to match this design language and won't change functionality, content, or structure — confirm that's the intent." Resolve any ambiguity here.

### Phase 1 — Extract the reference's design language (precisely)
Derive the reference's full design language using the `extract-design-language` skill (or consume an existing `DESIGN.md`/brief if the user has one). Capture exact tokens for color (with roles), typography, spacing/grid, shape, elevation, and motion, plus the qualitative personality and a set of **do/don't rules**. The output must be specific enough to apply without further guessing.

### Phase 2 — Inventory the existing app and map old → new
- Inventory the app's *current* design layer: its existing tokens/theme, fonts, spacing scale, component styling conventions.
- Build an explicit **token map**: current token → new token (e.g. `--color-primary #3B82F6 → #2665FD`, `--radius 6px → 8px`, body font `Inter → Geist`). Map by *role*, not by raw value, so the new language lands semantically (the old primary becomes the new primary, the old surface becomes the new surface).
- Identify where each maps in the codebase (the theme file, the CSS variables, component variant configs).

### Phase 3 — Apply (token layer first, pilot then roll out)
1. **Plan first.** Present the application plan: which files change, the token map, and the rollout order. Get a quick confirm for anything destructive.
2. **Apply at the token source** wherever the architecture allows — this re-themes broadly with minimal edits and zero logic changes.
3. **Pilot on one component/screen** before the full app. Render it, compare to the reference's language, fix gaps. Catching mismatches on one button across three states is far cheaper than discovering them after ten screens.
4. **Roll out** to the rest once the pilot matches. Use component-level styling only where tokens can't reach (one-off hardcoded values, third-party components), and change styling props only — never logic.
5. Keep edits as **clean diffs** the user can review; don't rewrite files wholesale when a token change suffices.

### Phase 4 — Verify
- [ ] **Functional regression check** — the app still builds and runs; logic, routing, data, and behaviour are untouched. This is the most important check.
- [ ] **Visual fidelity** — the UI now reads in the reference's design language: color roles, type, spacing rhythm, radii, elevation, motion all adopted faithfully.
- [ ] **Contrast & legibility** — new color pairings still meet WCAG AA; the palette swap didn't break readability or focus visibility.
- [ ] **No scope creep** — no features, states, sections, or copy were added or removed; accessibility semantics intact.
- [ ] **Consistency** — the language is applied uniformly, not half-migrated (no islands of the old theme).
- Render and visually review where possible; if you can't render, say so and self-review against this checklist and the token map.

## Output format
1. **The diffs** — the changed files (theme/token source first, then any component-level styling), as reviewable changes.
2. **What changed and why** — the token map (old → new) and the application strategy, so the user can audit fidelity.
3. **Boundary confirmation** — an explicit statement of what was *not* touched (logic, routing, data, content), and any place where a visual goal would have required a structural change that you flagged instead of making.

## Anti-patterns
- Touching logic, state, data, routing, or content to achieve a visual change.
- Editing components one-by-one when a token-source change would propagate cleanly.
- Applying vague styling ("make it modern") instead of the reference's exact tokens — yields generic slop.
- Adding hover/empty/error states, sections, or copy nobody asked for.
- Swapping the palette without re-checking contrast and focus visibility.
- Declaring done without confirming the app still runs and behaves identically.
