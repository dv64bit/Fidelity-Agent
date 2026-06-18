---
description: Replicate a UI from a screenshot or reference image into working code with maximum visual fidelity.
argument-hint: [optional: target stack, scope, or notes]
---

Replicate the provided UI reference (screenshot / image / mock) into working code with maximum visual fidelity.

Use the **ss-replicate** skill and follow its process exactly:
1. Confirm the image is actually attached. Run **Phase 0 intake** — ask only the follow-up questions you can't infer (target stack, the discrete token scale to snap to, scope, and how to handle unknowns like states/responsive/below-the-fold).
2. Do the mandatory **forensic analysis pass** before writing any code.
3. Build constrained to the discrete token scale — never emit continuous values, never "improve" the design.
4. Run the **visual self-correction loop** (render → screenshot → diff against reference → smallest-possible-diff fix → repeat). If you can't render here, say so and self-review against the checklist.
5. Deliver the three-part output: code, fidelity rationale, and "what I could not read from the reference."

Remember the governing rule: **this is replication, not design.** Reproduce what is shown; do not redesign, modernise, or invent.

User notes: $ARGUMENTS
