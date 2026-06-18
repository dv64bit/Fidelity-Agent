---
description: Extract the complete design language from an image, a live URL, or an existing codebase into a structured, reusable brief.
argument-hint: [source: image / URL / path, plus any focus]
---

Extract the complete design language from the reference and produce a structured, reusable design-language brief (tokens + patterns + rationale).

Use the **extract-design-language** skill:
1. Identify the source mode — **A) image/screenshot**, **B) live URL/website**, or **C) existing codebase** — and confirm the reference is actually present.
2. For URL and codebase sources, read the *actual* styling (computed styles, CSS variables, theme/token files, component variants) as ground truth — don't eyeball it. For images, infer and mark confidence.
3. Extract the full system: color (with roles), typography, spacing/layout, shape, elevation, motion, component patterns, and the qualitative design philosophy.
4. Output the brief in the skill's section order, keeping it implementation-ready.

Distinguish inference from fact, extract the system rather than proprietary content, and name what makes *this* language specific instead of flattening it to generic defaults.

If the end goal is a drop-in `DESIGN.md` for a codebase, hand off to **/export-designmd**.

Source / focus: $ARGUMENTS
