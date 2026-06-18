---
description: Apply Emil Kowalski's design-engineering best practices (UI polish, animation, interaction craft) to the current work.
argument-hint: [optional: file, component, or area to review/build]
---

Apply elite design-engineering best practices to the current work — UI polish, component design, animation decisions, and the invisible details that make software feel right.

Use the **emil-design-eng** skill (Emil Kowalski's design-engineering philosophy, bundled with this agent). When reviewing UI code, follow its required Before/After/Why markdown-table format and its review checklist. When building, apply its animation decision framework (should it animate at all → purpose → easing → duration), spring guidance, perceived-performance rules, accessibility rules (`prefers-reduced-motion`, touch hover gating), and the Sonner principles for loved components.

This skill is bundled locally, so it's available offline. To (re)install or update it from source into an agent's skills directory, run:

```bash
npx skills add https://github.com/emilkowalski/skill --skill emil-design-eng
```

Target / area: $ARGUMENTS
