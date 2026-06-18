---
description: Generate a spec-compliant DESIGN.md from the current codebase so AI agents read the real design language before generating UI.
argument-hint: [optional: repo path or scope]
---

Generate a spec-compliant `DESIGN.md` from the existing codebase and make it a drop-in context file for AI coding agents.

Use the **export-designmd** skill and follow its process:
1. **Extract from the source of truth** — token/theme files, global CSS, component variants, and actual usage. If the design language hasn't been analysed yet, run **extract-design-language** (codebase mode) first. Prefer values read from source over inferred ones.
2. Map values into the spec's **typed token groups** in YAML frontmatter (`name`, `colors`, `typography`, `spacing`, `rounded`, plus elevation/shape where used), using `{path.to.token}` references where appropriate.
3. Write the Markdown body in the **canonical section order** (Overview → Colors → Typography → Layout → Elevation & Depth → Shapes → Components → Do's and Don'ts), writing rationale and usage rules, not a token re-dump.
4. **Lint** against the spec rules (broken-ref, missing-primary, contrast-ratio, orphaned-tokens, missing-typography, missing-sections, section-order, token-summary) and report any you couldn't satisfy.
5. Write the file as exactly `DESIGN.md` at the repo root, give the one-line snippet to wire it into CLAUDE.md / AGENTS.md / Cursor rules, and offer optional CSS / Tailwind `@theme` / DTCG-JSON exports.

Fidelity over completeness — real tokens only, and name what makes this system specific.

Scope: $ARGUMENTS
