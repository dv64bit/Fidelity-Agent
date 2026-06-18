# Fidelity

A design-engineering agent for high-fidelity UI work. The through-line across all four skills is *fidelity*: faithful replication, faithful capture, faithful craft, faithful export. Built to avoid the generic-AI aesthetic and produce results a senior designer would ship.

## Skills & commands

| Command | Skill | What it does |
| --- | --- | --- |
| `/ss-replicate` | `ss-replicate` | Replicate a UI from a screenshot or reference image into working code with maximum visual fidelity. Asks follow-ups, runs a forensic analysis pass, snaps to a discrete token scale, and runs a render→diff→fix self-correction loop. Replication, not redesign. |
| `/extract-design-language` | `extract-design-language` | Extract the complete design language from an image, a live URL, or an existing codebase into a structured, reusable brief (tokens + patterns + rationale). |
| `/implement-best-practice` | `emil-design-eng` | Apply Emil Kowalski's design-engineering philosophy — UI polish, animation decisions, interaction craft, the invisible details. Bundled verbatim. |
| `/export-designmd` | `export-designmd` | Generate a spec-compliant `DESIGN.md` from a codebase (Google Stitch open format: YAML tokens + Markdown rationale, canonical section order, lint-clean) so AI agents read the real design language before generating UI. |

## How the pieces fit

- **Replicate** a reference you can see → `/ss-replicate`
- **Understand and capture** a design language from any source → `/extract-design-language`
- **Polish** the craft of what you build → `/implement-best-practice`
- **Export** an existing codebase's system as a portable spec → `/export-designmd`

`extract-design-language` feeds `export-designmd` (analysis → spec file). `ss-replicate` and `export-designmd` both share the forensic-reading discipline.

## Install

This is a Claude Code plugin. Commands live in `commands/`, skills in `skills/`, manifest in `.claude-plugin/plugin.json`.

The bundled third-party skill (`emil-design-eng`) is included so the agent works offline. To (re)install or update it from source via the open skills CLI:

```bash
npx skills add https://github.com/emilkowalski/skill --skill emil-design-eng
```

## Provenance

The skills are grounded in current sources, not training-data assumptions:

- **ss-replicate** — the discrete-token-constraint method, image-prep practices, the three-part output (including the "what I couldn't read" surface), smallest-diff iteration, and the visual self-correction loop reflect current screenshot-to-UI accuracy practice (2026).
- **export-designmd** — built on the open **DESIGN.md** spec introduced by Google Stitch (announced March 2026, open-sourced April 2026): YAML token frontmatter + Markdown rationale, eight canonical sections, the eight linter rules, `{path.to.token}` references, and CSS / Tailwind `@theme` / W3C DTCG-JSON exports.
- **implement-best-practice** — Emil Kowalski's `emil-design-eng` skill, bundled verbatim, installable via `vercel-labs/skills` (`npx skills add owner/repo --skill name`).

## Design intent

Every skill is written to fight drift and the generic-AI default. They favour discrete tokens over continuous guesses, name what makes a specific system specific, and treat verification (visual diff, linting, checklists) as part of the job rather than an afterthought.
