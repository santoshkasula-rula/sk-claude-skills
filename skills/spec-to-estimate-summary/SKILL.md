---
name: spec-to-estimate-summary
description: Use when the user wants to turn a spec, technical design, and estimates doc into a single polished HTML project brief (problem, solution, requirements, architecture with diagrams, and per-milestone effort estimates with rationale).
---

# Spec to Estimate Summary

## When to use this

The user has (or will attach) a spec/PRD, a technical design doc, and an estimates doc for a project — in Markdown, HTML, or similar — and wants them turned into a single, polished, self-contained HTML brief that's easy for a mixed technical/non-technical audience to read. Trigger on requests like "turn these into an estimate doc," "write up the architecture and estimates for X," or "make a project brief from these files."

## What to do

Generate **one self-contained HTML file** (inline CSS, no external dependencies, light + dark mode via `prefers-color-scheme`) following the exact structure and visual language described below. Only the content changes per project — the skeleton, CSS classes, and section order stay the same.

### Before writing anything: consistency check

Cross-check the three source documents:
- Do the milestones in the estimates doc map to real components in the technical design?
- Does every functional requirement in the spec show up somewhere in the design or estimates?
- Does every estimate number have an underlying piece of work described in the design?

If anything is missing, contradictory, or a source document wasn't provided, **stop and ask the user** rather than inventing or guessing the gap. Do not fabricate numbers, requirement IDs, or architecture details not present in the source docs.

### CSS / shell to reuse

Use CSS custom properties for a light theme with a `@media (prefers-color-scheme: dark)` override. Body font: Inter/system-ui, 15px/1.55. A `.wrap` container capped at `max-width:1040px`, centered, with generous padding. `h1` for the title, `h2` for numbered major sections (with a top border and anchor `id`s), `h3` for subsections. A `.meta` class for small muted metadata lines. Reusable component classes:

- `nav` — flex-wrapped row of pill-style anchor links (`border-radius:999px`) to every `h2` section, sticky-feeling but simple.
- `.tiles` / `.tile` — responsive grid of big-number scorecards (big bold number + small muted label underneath).
- `.card` — bordered rounded box for callouts; variants `.card.warn` (amber bg) and `.card.ok` (green bg).
- `.grid2` — two-column responsive grid for paired cards.
- Tables (`.tbl table`) — clean bordered tables, numeric columns right-aligned via `.n`, a `.total` row style for sums.
- `.chip` — small rounded pill labels for tags; semantic variants `.bot` (teal), `.human` (purple/accent), `.ops` (amber), and confidence/complexity levels `.hi` (red), `.med` (amber), `.lo` (green).
- `.flow` / `.node` / `.arrow` — a simple left-to-right flowchart made of flex boxes and arrow glyphs (no JS/canvas needed). Node variants: `.node.new` (new work, accent border), `.node.reuse` (reused pattern, green), `.node.later` (dashed, muted, out of scope/future).
- `.legend` — small muted line explaining the node color key under each flow diagram.
- `.bar` — a horizontal stacked bar (flex children with inline `width:%` and background color) for showing day-split breakdowns (e.g., core/ops/rollout).
- `.ms` / `.ms-h` / `.ms-b` — a bordered "milestone" card: header row with milestone name + big day count, body with outcome text, a `.why` box, and a task table.
- `.why` — a sub-box inside each milestone containing: a complexity chip + one-line rationale sentence, a "Why these days" bulleted breakdown (each bullet ties a day figure to a concrete cause), and a "What would change it" bullet list of risk-driven deltas.
- Tag convention: append `<span class="tag">[assumed]</span>`, `[inferred]`, or `[from spec]`/`[SAID PRD ...]` to any claim not explicitly stated verbatim in a source doc, so the reader can see what's sourced vs. synthesized.
- `@media print { nav { display:none } .ms { break-inside:avoid } }` for print-friendliness.

### Section order and content

1. **Title + meta line** — project name, phase, draft date/status, "replaces vN" if relevant.
2. **Nav** — pill links to every section below by anchor.
3. **Scorecard** (`.tiles` + roll-up table) — total delivery days, core engineering days, buffer days, stakeholder-engagement days, review/compliance days as tiles; then a per-milestone roll-up table (engineering / buffer / engagement / review / delivery days / running total). Follow with two `.card`s: one explaining what each column means, one explaining "how to read the engineering days" (e.g., bot drafts/writes code, human directs/reviews/validates against real data) with the `.chip` legend shown inline.
4. **§1 Problem statement** — short paragraph, then `.grid2` of two cards: "What is blocked" and "What success looks like," each a short bullet list.
5. **§2 Solution approach** — one bold summary sentence, a `.flow` diagram of the build sequence (one node per step/milestone), a `.card.warn` naming the single biggest accepted risk, and a `.meta` line on alternatives considered and explicit out-of-scope items.
6. **§3 Requirements** — `<h3>User stories</h3>` as a bulleted list by persona/role (bold the role, one sentence need each), then `<h3>Functional requirements</h3>` as a table: Area / What the product does / Spec IDs / Milestone. Add a closing `.meta` note on anything left undefined in the spec that the design had to assume.
7. **§4 Architecture overview** — bold one- or two-sentence plain-language summary of what talks to what and why (no jargon dump). Then one or two `.flow` diagrams showing the actual data/control flow, using `.node.new` for new build, `.node.reuse` for components/patterns that already exist, `.node.later` dashed for explicitly out-of-scope/future work — with a `.legend` line underneath. Optionally add a precedent table (Piece / Existing pattern found / What it means) if the design doc references existing code/infra being copied. Add `.card.warn` boxes for any architecture-level risks or unconfirmed assumptions, and a short list of "choices this phase still needs to validate."
8. **§5 Estimates** — intro line stating the units (engineering days, one engineer, no buffer/waits unless stated). A `.bar` stacked bar showing the split (e.g., core/ops/rollout) across the total. A summary table: # / Milestone / what the user gets / core / ops / rollout / total days / running total / confidence chip. Then **one `.ms` block per milestone**, each containing:
   - Header: milestone name + total days (broken down in a small `.meta` aside, e.g. "(3.75 core · 0.5 ops · 0.5 rollout)").
   - Outcome paragraph: what's usable, by whom, and what gate/sign-off it needs.
   - `.why` box: complexity chip (Low/Medium/High) + one-line rationale tied to a named architecture component; a "Why these days" bullet list where every sub-estimate traces to a concrete cause (reused pattern = cheap, new/unverified/business-logic-heavy = expensive); a "What would change it" bullet listing concrete risk deltas (e.g., "+1 day if X turns out false").
   - Task table: Technical change in plain language / Who (bot/human/ops chip) / Days.
   Close with a `.card.ok` comparing this estimate to any prior version, if one exists, explaining what changed and why.
9. **§6 What moves the numbers** — a table: If this turns out true… / Effect / Milestone / confidence chip. Followed by a short "Decisions that unlock the most work" bullet list (decision + who owns it). Note any waiting/dependency time (reviews, other teams) explicitly as *not* counted in engineering days but still on the critical path.
10. **§7 Sources and open questions** — `.grid2` of source cards (what was read/cited), a "Not verified" bullet list of things assumed but unconfirmed, and a short calibration note (e.g., "re-run estimate after milestone 1 using actual review time").
11. Optional **§8 Future phase preview** if the source docs mention a later phase — same `.flow`-diagram-plus-open-questions treatment, clearly marked as unestimated.

### Style notes

- Prose is plain-language and tight; avoid jargon in the architecture overview especially — explain it like you would to a smart non-engineer.
- Keep bullets short (1–2 sentences); use bold lead-ins for scanability.
- Every number in §5 must be traceable to a cause named in §4; don't let estimates float free of the architecture.
- Use the `[assumed]`/`[inferred]`/`[from spec]` tags liberally — this is what lets the reader distinguish sourced fact from synthesis at a glance.

### Output

Write the HTML to a file and present it to the user (or create it as an artifact if the conversation supports that) so they can view and download it.
