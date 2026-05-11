---
name: estimate-effort
description: Generates a PM-readable, engineer-actionable implementation plan from a PRD and/or Zoom transcript. Extracts RAD from documents, then confirms gaps with the user before writing the plan.
disable-model-invocation: true
argument-hint: "[--root]"
allowed-tools: Read Bash(find *) Bash(ls *) Write
---

# estimate-effort

Read `OUTPUT_TEMPLATE.md` from `${CLAUDE_SKILL_DIR}`.
If `--root` is in $ARGUMENTS, enable multi-repo discovery in Step 2.

---

## Step 1 — Collect files

Ask for each file in sequence. Read it before asking for the next.

1. **PRD** — "Path to your PRD?" — required, re-ask if unreadable.
2. **Transcript** — "Zoom transcript or meeting notes? (path or skip)" — if provided, normalize it: strip timestamps, speaker labels, `[inaudible]`, and merge fragmented turns into prose.
3. **Additional context** — "Any other files? (architecture docs, API specs, prior plans — path or done)" — repeat until 'done'.

---

## Step 2 — Repo discovery (only if `--root`)

```bash
find . -maxdepth 3 \( -name "package.json" -o -name "go.mod" -o -name "requirements.txt" -o -name "Cargo.toml" -o -name "pom.xml" \) ! -path "*/node_modules/*" ! -path "*/.git/*"
```

Identify each service and any cross-repo shared dependencies.

---

## Step 3 — Extract signals

**From all documents**, extract and record. Use the exact terminology, feature names, and system names as they appear in the documents — do not rename or generalize them.

| Signal | Source |
| :--- | :--- |
| Project name, objective, success metrics | PRD |
| Features in scope, features explicitly out of scope | PRD |
| Technical components mentioned | Transcript / docs |
| Risks detected — anxiety markers: *"worried about"*, *"legacy"*, *"not sure how long"*, *"tricky"*, *"depends on"*, *"blocked by"*, *"never touched"* | Transcript / docs |
| Named blockers, external dependencies, open questions | Transcript / docs |
| Assumptions stated or implied | All sources |

**Conflict rule:** Transcript anxiety markers override PRD optimism on the same component — always.

---

## Step 4 — RAD confirmation (targeted, not a full interview)

Present what you extracted as a pre-filled draft. Only ask about genuine gaps — do not re-ask for things already found in the documents.

> **Here's what I found in your documents — please correct or fill in any blanks:**
>
> **Risks**
> - [Extracted risk 1] *(source)*
> - [Extracted risk 2] *(source)*
> - Anything I missed?
>
> **Assumptions**
> - [Extracted assumption 1] *(source)*
> - Anything I missed?
>
> **Dependencies**
> - [Extracted dependency 1] *(source)*
> - Anything I missed?
>
> **Out of scope** — [Extracted or "not found — what should be excluded?"]
>
> **Success criteria** — [Extracted or "not found — how will you know this is done?"]

Wait for the user's response. Merge corrections and additions before proceeding.

---

## Step 5 — Design milestones

Create 3–5 milestones. Use the language from the documents — milestone names, deliverable descriptions, and business value statements must reflect the actual words, feature names, and goals used in the PRD, transcript, and any additional context provided. Do not substitute generic placeholders.

Each milestone must:
- Deliver a concrete, demonstrable artifact
- Be sequenced so each unblocks the next
- Map to a specific business value from the PRD
- Break effort into three buckets (never combine):
  - **Engineering** — coding, design, code review
  - **Operational** — deployment, infra, config, secrets
  - **Rollout** — QA, testing, feature flag ramp, stakeholder demo
- Carry a risk flag: 🔴 anxiety-marker or RAD risk · 🟡 uncertain · 🟢 well-understood

**Buffer:** 20% added to Engineering total only. Show raw + buffered separately. Never embed inside milestones.

If `--root`: assign tasks to repos, sequence to respect cross-repo dependencies.

---

## Step 6 — Write IMPLEMENTATION_PLAN.md

Follow `OUTPUT_TEMPLATE.md`. Key rules:

- **Executive Summary** — plain language, no repo names, include: goal, effort totals (raw + buffered), risk callout if any 🔴, success criteria.
- **Scope** — in scope and out of scope lists.
- **Milestones** — deliverable, business value, Engineering / Operational / Rollout effort, risk flag.
- **RAD** — every item cites its source (PRD, Transcript, or User).
- **Technical Breakdown** — one section per repo, specific files/areas, one-line rationale each.
- **Cross-repo map** — only if `--root`.
- **Engineer Updates** — empty table, leave for the team.

End with:
> **Share with PMs / leadership:** Executive Summary, Scope, Milestones
> **Share with engineers:** full document
