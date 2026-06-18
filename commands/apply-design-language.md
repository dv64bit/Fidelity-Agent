---
description: Adopt a reference's design language and apply it to the existing codebase — restyle the UI only, never the functionality.
argument-hint: [optional: scope, target stack, or notes]
---

Adopt the design language of the provided reference (screenshot / mock / live site / inspiration) and apply it to the **existing** build — restyling the UI/UX only, with zero changes to application logic, behaviour, structure, routing, data, or content.

Use the **apply-design-language** skill and follow its process:
1. **Phase 0 intake** — confirm the reference is attached, identify the existing build's styling architecture and token source of truth, confirm scope, and restate the presentation-only boundary.
2. **Extract** the reference's design language precisely via **extract-design-language** (exact hex/fonts/px + do/don't rules), or consume an existing DESIGN.md.
3. **Inventory** the existing app's design layer and build an explicit **old → new token map** (mapped by role).
4. **Apply at the token source first** (Tailwind config / `@theme` / CSS variables / theme object), plan before destructive edits, **pilot on one component or screen**, then roll out. Component-level styling only where tokens can't reach — styling props only, never logic.
5. **Verify** — the app still builds and behaves identically (most important), the UI faithfully adopts the new language, contrast/focus still pass WCAG AA, and nothing was added or removed.

Governing rule: **change how the app looks, never how it works.** Apply exact tokens, not vague descriptors — and if a visual goal would require a structural/logic change, flag the trade-off instead of refactoring behaviour.

Scope / notes: $ARGUMENTS
