---
description: Implement an existing DESIGN.md into the codebase with verifiable, zero-drift fidelity — wire tokens, purge overrides, and prove conformance.
argument-hint: [optional: path to DESIGN.md, scope, or notes]
---

Implement the provided `DESIGN.md` into the codebase so the result matches the spec **exactly**, with a verified conformance audit and no discrepancy.

Use the **implement-designmd** skill and follow its process:
1. **Parse the spec fully** — frontmatter tokens (resolve every `{path.to.token}` reference, flag broken or circular refs) and prose usage rules. Build a flat **conformance table** as the source of truth. Identify the codebase's token architecture and restate the presentation-only boundary.
2. **Make the spec the single source of truth** — translate the table 1:1 into the codebase's exact token format (CSS variables / Tailwind `@theme` or config / theme object / shadcn variables), with no rounding or interpretation. Create a token architecture if none exists — this is the real drift fix.
3. **Purge overrides** — find hardcoded/arbitrary values that bypass tokens; replace exact matches with the token, and **flag near-misses instead of silently snapping them.**
4. **Run the token-by-token conformance audit** — re-extract implemented values, diff against the spec table (on-spec / missing / mismatch / overridden / orphaned), score it, fix every discrepancy, and **re-audit until 100% on-spec with zero overrides.**
5. **Lock it in** — offer a no-hardcoded-values lint rule, a DESIGN.md pointer in CLAUDE.md/AGENTS.md, and an optional CI conformance gate.

Governing rules: the spec is **normative, not inspirational**; **unverified is unfinished**; and change how the app looks, never how it works.

Path / scope: $ARGUMENTS
