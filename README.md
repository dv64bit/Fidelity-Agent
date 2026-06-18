# Fidelity

A design-engineering agent for high-fidelity UI work. The through-line across all four skills is *fidelity*: faithful replication, faithful capture, faithful craft, faithful export. Built to avoid the generic-AI aesthetic and produce results a senior designer would ship.

## Install

### Option 1 — Install the full agent (recommended)

Clone the repo into your Claude Code plugins directory:

```bash
git clone https://github.com/dv64bit/Fidelity-Agent.git ~/.claude/plugins/fidelity
```

Then add it to your Claude Code config (`~/.claude/settings.json`):

```json
{
  "plugins": ["~/.claude/plugins/fidelity"]
}
```

Restart Claude Code. All four commands (`/ss-replicate`, `/extract-design-language`, `/implement-best-practice`, `/export-designmd`) are available immediately.

### Option 2 — Install individual skills via CLI

Pick only the skills you need:

```bash
npx skills add https://github.com/dv64bit/Fidelity-Agent --skill ss-replicate
npx skills add https://github.com/dv64bit/Fidelity-Agent --skill extract-design-language
npx skills add https://github.com/dv64bit/Fidelity-Agent --skill export-designmd
npx skills add https://github.com/dv64bit/Fidelity-Agent --skill emil-design-eng
```

Or install all four at once:

```bash
npx skills add https://github.com/dv64bit/Fidelity-Agent --skill ss-replicate && \
npx skills add https://github.com/dv64bit/Fidelity-Agent --skill extract-design-language && \
npx skills add https://github.com/dv64bit/Fidelity-Agent --skill export-designmd && \
npx skills add https://github.com/dv64bit/Fidelity-Agent --skill emil-design-eng
```

---

## Usage

### `/ss-replicate` — Replicate a UI from a screenshot

Give Claude a reference image and ask it to replicate the UI into code.

```text
/ss-replicate
```

Then attach a screenshot or paste an image URL. Claude will:

1. Run a forensic analysis pass (spacing, color, type, radius, shadow)
2. Snap values to a discrete token scale
3. Build the UI
4. Render → diff → fix until it matches

**When to use:** You have a design mock, screenshot, or reference and want the code to match it exactly — not a redesign, a replication.

---

### `/extract-design-language` — Extract a design system from any source

```text
/extract-design-language
```

Works from three source types:

- **Image/screenshot** — paste or attach
- **Live URL** — paste the link
- **Codebase** — point Claude at your project

Outputs a structured brief: discrete tokens (color, type, spacing, radius, shadow, motion) plus the rationale behind each decision.

**When to use:** Before building new UI in an existing product, or when you need to document what a design system actually is rather than what it should be.

---

### `/implement-best-practice` — Apply Emil Kowalski's design-engineering craft

```text
/implement-best-practice
```

Applies Emil Kowalski's UI polish philosophy to your component or page: micro-interactions, animation timing, hover states, the invisible details that separate good from great.

**When to use:** You've built the structure and want to elevate the craft — not change what it does, but make it feel right.

---

### `/export-designmd` — Export a `DESIGN.md` from a codebase

```text
/export-designmd
```

Generates a spec-compliant `DESIGN.md` (Google Stitch open format) from your existing codebase:

- YAML frontmatter: typed design tokens (color, type, spacing, radius, shadow, motion)
- Markdown body: rationale, usage rules, do/don't guidance
- Eight canonical sections, lint-clean, `{path.to.token}` references
- Exports to CSS custom properties, Tailwind `@theme`, or W3C DTCG JSON

**When to use:** You want AI coding agents (Claude Code, Cursor, Copilot) to read your actual design language before generating UI — not guess it from training data.

---

## How the pieces fit

```text
extract-design-language  →  export-designmd   (analyse → emit spec)
ss-replicate             →  implement-best-practice  (build → polish)
```

- **Replicate** a reference you can see → `/ss-replicate`
- **Understand and capture** a design language from any source → `/extract-design-language`
- **Polish** the craft of what you build → `/implement-best-practice`
- **Export** an existing codebase's system as a portable spec → `/export-designmd`

---

## Skills & commands

| Command | Skill | What it does |
| --- | --- | --- |
| `/ss-replicate` | `ss-replicate` | Replicate a UI from a screenshot or reference image into working code with maximum visual fidelity. |
| `/extract-design-language` | `extract-design-language` | Extract the complete design language from an image, a live URL, or an existing codebase into a structured, reusable brief. |
| `/implement-best-practice` | `emil-design-eng` | Apply Emil Kowalski's design-engineering philosophy — UI polish, animation decisions, interaction craft. |
| `/export-designmd` | `export-designmd` | Generate a spec-compliant `DESIGN.md` from a codebase (Google Stitch open format). |

---

## Provenance

The skills are grounded in current sources, not training-data assumptions:

- **ss-replicate** — discrete-token-constraint method, image-prep practices, three-part output (including "what I couldn't read"), smallest-diff iteration, and visual self-correction loop. Reflects current screenshot-to-UI accuracy practice (2026).
- **export-designmd** — built on the open **DESIGN.md** spec introduced by Google Stitch (announced March 2026, open-sourced April 2026): YAML token frontmatter + Markdown rationale, eight canonical sections, eight linter rules, `{path.to.token}` references, CSS / Tailwind `@theme` / W3C DTCG-JSON exports.
- **implement-best-practice** — Emil Kowalski's `emil-design-eng` skill, bundled verbatim, installable via `vercel-labs/skills` (`npx skills add owner/repo --skill name`).

## Design intent

Every skill is written to fight drift and the generic-AI default. They favour discrete tokens over continuous guesses, name what makes a specific system specific, and treat verification (visual diff, linting, checklists) as part of the job rather than an afterthought.
