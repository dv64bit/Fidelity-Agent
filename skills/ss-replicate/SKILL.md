---
name: ss-replicate
description: Replicate a UI from a screenshot, reference image, or design mock into working code with maximum visual fidelity. Use this whenever the user provides a UI image and wants it rebuilt, cloned, copied, or reproduced "exactly" / "pixel-perfect" / "1:1" / "as-is" — including dashboards, landing pages, mobile screens, components, side sheets, modals, tables, and forms. Triggers on "clone this screen", "replicate this UI", "build this from the screenshot", "match this design exactly", "recreate this", or the /ss-replicate command. This is a REPLICATION skill (reproduce what is shown), not a design skill (do not improve, redesign, or add). Prefer this over freehand coding any time visual fidelity to a reference image is the goal.
---

# UI Replication (ss-replicate)

Reproduce a reference UI as faithfully as the medium allows. The single most common failure mode is not "the model can't code the UI" — it is **drift**: the corner radius lands at 10px instead of 12px, the weight reads semibold instead of medium, the shadow is one elevation step too heavy, the spacing breathes 4px too much. This skill exists to eliminate drift through a disciplined process, not optimism.

## The one rule that governs everything: replication is not design

You are reproducing, not designing. Do not "improve" the layout, "modernise" the spacing, "clean up" the hierarchy, swap the font for something nicer, or fill empty states with invented content. If the reference has an awkward 13px gap, you reproduce a 13px gap. The user asked for a copy; deviation — however well-intentioned — is the defect, not the feature. If you catch yourself mentally reframing a choice as "this would look better," stop: that instinct is the signal you are about to introduce drift.

The only exception is information genuinely absent from the reference (a hover state, an error state, content below the fold). Those are *unknowns*, not *improvements* — handle them in Phase 0, not by guessing.

## Why one-shot reproduction never lands (and what to do about it)

When the model reads an image it loses metadata your eye reconstructs for free. A 12px-radius card is just a smooth boundary at the pixel level; nothing in the image says the intended value was 12 rather than 10 or 14. Drift concentrates in five places:

- **Font weight** — regular vs medium vs semibold
- **Spacing grid** — a 4px scale vs an 8px scale
- **Shadow elevation** — one diffuse blur looks like several
- **Color tokens** — `#EEEEEE` vs `#F2F2F2`
- **Icon stroke width** — 1.5px vs 2px

The model is not failing; it is correctly inferring *continuous* values from pixels, and those values won't land on *discrete* design-system tokens unless you force the choice. So the whole method is: **constrain the model to a discrete scale up front, then verify visually and correct.** Guessing is replaced by choosing; choosing is then checked against the image.

## Process

Run these phases in order. Do not skip Phase 0 or Phase 3 — they are where fidelity is won.

### Phase 0 — Intake and follow-up questions (ask before building)

Confirm an image is actually present (a prompt implying one does not mean one was attached — check). Then resolve the variables that determine fidelity. Ask the user only what you cannot infer; infer the rest and state your assumption inline.

Ask about:

1. **Target stack.** Framework + styling system (e.g. React + TypeScript + Tailwind + shadcn/ui, plain HTML/CSS, SwiftUI, Flutter). If the user has an established stack, default to it and say so rather than asking.
2. **Token scale to snap to.** The allowed spacing / radius / type-size / shadow values. If the user has a design system (Untitled UI, shadcn/ui, Fluent 2, an existing `DESIGN.md`, a Tailwind theme), snap to *its* scale — do not invent one. If none exists, propose a sensible discrete scale (spacing 4/8/12/16/24/32, radius 4/8/12/16, type 12/14/16/18/24) and proceed.
3. **Scope.** The whole screen, one component, or a region. A tight scope produces far higher fidelity than "rebuild everything" — narrow it if the user is vague.
4. **Unknowns in the reference.** Anything the static image cannot show: interactive states (hover/focus/active/disabled), responsive behaviour, real data vs placeholder, content below the fold, motion. List what you see missing and ask how to handle it — reproduce as static, stub it, or leave it out.

Keep this to one focused round. If the user said "just build it," make defensible assumptions, state them, and move on — do not stall.

If the image is low resolution or a 1x capture, note that glyph edges will blur and weight/stroke estimates get shaky; ask for a 2x export or a tighter crop if available. Do not let a poor source silently degrade the result without flagging it.

### Phase 1 — Forensic analysis pass (mandatory, before any code)

Do not write a line of markup until you have read the reference like an inspector, not a viewer. Read `references/analysis-pass.md` for the full element-by-element checklist. Produce a structured inventory covering, at minimum:

- **Layout skeleton** — the box model: outer container, columns/grid, major regions, alignment, what is fixed vs fluid.
- **Spacing** — gaps and padding, each snapped to the agreed scale (record the snapped value *and* note if it sat awkwardly between two steps).
- **Typography** — family, size, weight, line-height, letter-spacing, case, color — per distinct text style, not per line.
- **Color** — every fill, border, and text color as a hex/token, with its role (surface, primary action, secondary text, border).
- **Shape & elevation** — radii, borders (width + color), shadows (offset/blur/spread/color), described as discrete steps.
- **Iconography** — icon set if identifiable, stroke width, size.
- **Components** — name the *primitive* each block maps to in the target system (this is a Button, this is a Card, this is a Tooltip), so you instantiate components rather than hand-drawing pixels. Reproducing via the design system's own primitives is what kills drift on radius, height, and state.

State explicitly anything you could not read with confidence — those are the items you flagged in Phase 0.

### Phase 2 — Build, constrained to the discrete scale

Implement against the inventory, not against the image directly. Hard constraints:

- **Snap every numeric value to an allowed token.** Never emit continuous junk like `padding: 13.5px` or `rounded-[11px]`. If a measurement sits between two steps, pick the nearer one and note it.
- **Specify exact properties, never `transition: all`.** Reproduce only the transitions visible or specified.
- **Instantiate components, do not draw them.** Use the target system's Button/Card/Input/etc. so heights, radii, and states inherit correctly.
- **Reproduce empty/placeholder content verbatim.** Do not invent copy, names, numbers, or avatars beyond what is shown.
- **Keep structure honest.** Semantic HTML, real layout primitives (fl/grid), no absolute-positioning hacks to fake alignment that the original achieves with flow.

### Phase 3 — Visual self-correction loop (this is where fidelity is actually earned)

A one-shot build is a draft, never the deliverable. Close the gap by *seeing your own output*, not by re-reading the prompt:

1. Render the build (run it, or use the available browser/preview tooling to load it).
2. Capture a screenshot of the rendered result at the same viewport as the reference.
3. **Diff against the reference, region by region.** Look specifically at the five drift zones above. Name discrepancies as concrete measurements: "card padding renders ~20px, reference is ~16px"; "heading reads semibold, reference is medium"; "shadow too diffuse — drop one elevation step."
4. Apply the **smallest possible diff** to fix each named discrepancy. Do not rewrite working structure — changing classes you were not asked to change reintroduces drift. Fix the specific values, nothing else.
5. Repeat until the diff is down to imperceptible. Treat yourself as a junior engineer under QA: each pass, the bar is "what is still different," not "what could be nicer."

If you cannot render in the current environment, say so, and instead deliver the build plus an explicit self-review against the reference and the Phase 4 checklist — do not pretend a loop happened.

### Phase 4 — Verification checklist

Before presenting, confirm every item. Report any you could not verify.

- [ ] Layout skeleton matches: same regions, same order, same alignment.
- [ ] Every spacing value snaps to the agreed scale; none invented.
- [ ] Type styles match family / size / weight / line-height / case per style.
- [ ] Colors match by token/hex and by *role*.
- [ ] Radii, borders, and shadow elevation match as discrete steps.
- [ ] Components are real primitives, not hand-drawn lookalikes.
- [ ] No invented content, no "improvements," no redesigns.
- [ ] Unknowns (states, responsive, below-fold) are handled as agreed in Phase 0, not silently guessed.
- [ ] A visual diff pass was done (or its absence was flagged).

## Output format

Deliver three parts, always:

1. **The code** — complete and runnable in the target stack.
2. **Fidelity rationale** — a short bulleted log of the non-obvious values you chose and why (which spacing step, which weight, which elevation), so a reviewer can audit drift without re-measuring.
3. **What I could not read from the reference** — the explicit list of unknowns and how each was handled. This is not an apology; it is the surface that lets the user close the last gap fast.

## Anti-patterns (do not do these)

- Building straight from the image with no analysis pass.
- Emitting continuous values (`12.7px`, arbitrary hex) instead of snapping to tokens.
- "Improving" spacing, hierarchy, copy, or fonts.
- Rewriting the whole component to fix a 4px gap.
- Declaring "pixel-perfect" without a visual diff pass.
- Inventing hover/error/empty states the user did not ask you to design.
