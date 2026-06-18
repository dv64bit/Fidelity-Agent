# Forensic Analysis Pass — element-by-element reading

Read the reference the way an inspector reads a building, not the way a visitor looks at it. Work top-to-bottom, outside-in. The goal is a written inventory complete enough that you could rebuild the UI from the inventory alone, without the image.

## Table of contents
1. Layout skeleton
2. Spacing
3. Typography
4. Color
5. Shape and elevation
6. Iconography and imagery
7. Component mapping
8. States and the unreadable

## 1. Layout skeleton

- Identify the outermost container and its max-width / alignment (centered, full-bleed, fixed sidebar + fluid main).
- Identify the grid or flex structure of each major region. Count columns. Note gutters.
- Note what is fixed (headers, sidebars, sticky footers) vs what scrolls.
- Note alignment within each region: left/center/right, baseline vs center, distribution (space-between, packed).
- Record reading order and z-layering (overlays, sheets, popovers sitting above content).

## 2. Spacing

- Measure padding inside each container and the gaps between siblings.
- Snap each to the agreed discrete scale. Record the snapped value. If a measurement sat between two steps (e.g. ~14px between an 8 and 16 scale), record that tension — it often means a 4px half-step is in play, or the source uses a different base unit.
- Determine the base unit (4px grid vs 8px grid). Most systems are one or the other; identifying it early makes every later snap trivial.
- Distinguish *outer* spacing (margins between components) from *inner* spacing (padding within a component) — they often use different steps.

## 3. Typography

Catalogue by *style*, not by line. A screen usually has 4–8 distinct text styles; find them.

For each style record: font family (name it if recognisable; otherwise describe — geometric sans, humanist sans, slab, mono), size (snapped to the type scale), weight (regular/medium/semibold/bold — be precise, this is a top drift zone), line-height, letter-spacing (especially tightened headings and tracked-out labels/uppercase), case, and color.

If the family is not identifiable, say so and pick the closest defensible default for the target stack rather than guessing a specific licensed font.

## 4. Color

- Extract every distinct color as a hex (or as the design-system token if you can map it).
- Tag each with its *role*: surface/background, raised surface, primary action, secondary/ghost action, primary text, secondary/muted text, border/divider, semantic (success/warning/danger/info), focus ring.
- Note transparency and overlays (scrims behind modals, hover tints).
- Roles matter more than raw values for fidelity: applying the right hex to the wrong role still reads wrong.

## 5. Shape and elevation

- **Radius**: per component, snapped to the radius scale. Watch for fully-rounded (pill) vs fixed-radius.
- **Borders**: width and color, and whether a border is doing the job a shadow appears to do (flat designs separate with 1px borders, not elevation).
- **Shadows**: describe as discrete elevation steps (none / soft / medium / strong), each with rough offset, blur, spread, and color/opacity. One soft diffuse shadow is frequently mis-read as a heavier one — err toward the lighter step and verify in Phase 3.

## 6. Iconography and imagery

- Identify the icon set if possible (Lucide, Heroicons, Phosphor, SF Symbols, Fluent icons). Record stroke width (1.5 vs 2 is a drift zone) and nominal size.
- For images/avatars/logos: record dimensions, aspect ratio, radius, and object-fit. Do not recreate copyrighted logos or brand marks — placeholder them and flag it.

## 7. Component mapping

For each visual block, name the *primitive* it maps to in the target design system: Button (and variant — primary/secondary/ghost/destructive), Input, Select, Checkbox, Card, Badge/Tag, Avatar, Tooltip, Popover, Dropdown menu, Dialog/Modal, Sheet/Drawer, Table, Tabs, Breadcrumb, Pagination, Toast.

Instantiating the real primitive is the highest-leverage fidelity move: the component already carries the correct height, radius, padding, focus ring, and state styling, so you inherit them instead of approximating them pixel by pixel.

## 8. States and the unreadable

A static image cannot show: hover/focus/active/disabled/loading states, validation/error states, empty states, responsive breakpoints, scroll behaviour, motion/transitions, and anything below the fold.

List every such unknown explicitly. These are the items to raise in Phase 0 and to report in the "what I could not read" section of the output. Never silently invent them — an invented state is a redesign, which violates the one rule.
