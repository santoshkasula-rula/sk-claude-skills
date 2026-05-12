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

## Step 4 — Codebase pattern scan

Before forming any opinions, scan the codebase to understand how things are currently built. Look for:

- How similar features are structured (routing, layering, data access patterns)
- How the UI components interact with backend services (REST vs event-driven, polling vs push, shared state management)
- How config, secrets, and environment differences are handled
- How existing cross-service calls are made (sync vs async, retry patterns, error handling)
- Any evidence of prior migrations, refactors, or tech debt comments near the areas this feature will touch

Use `find`, `ls`, and `Read` on key files. Do not scan exhaustively — focus on the areas most relevant to what the PRD is asking for.

From this scan, form 2–4 specific design tradeoffs the team will need to decide. Each tradeoff must:
- Be grounded in something you actually observed in the codebase — not a generic best practice
- Present two concrete options (Option A / Option B) with a one-line consequence for each
- Flag which RAD risks or dependencies make one option safer than the other

---

## Step 4b — RAD + tradeoffs confirmation (one message, not two)

Send one message combining the pre-filled RAD draft and the design tradeoffs. Do not send two separate messages. Only surface gaps — do not re-ask for things already confirmed in the documents.

> **Here's what I found — correct or add anything, and weigh in on the tradeoffs:**
>
> ---
> **RAD**
>
> Risks: [list with source] — *anything missing?*
> Assumptions: [list with source] — *anything missing?*
> Dependencies: [list with source] — *anything missing?*
> Out of scope: [extracted, or "not found — what should be excluded?"]
> Success criteria: [extracted, or "how will you know this is done?"]
>
> ---
> **Design tradeoffs — your call**
>
> *[Tradeoff 1 — grounded in codebase observation]*
> - **Option A:** [what it is] → [consequence]
> - **Option B:** [what it is] → [consequence]
> - *Leans toward A/B because [RAD risk or dependency that tips the balance]*
>
> *[Tradeoff 2]*
> - **Option A:** [what it is] → [consequence]
> - **Option B:** [what it is] → [consequence]
> - *Leans toward A/B because [reason from codebase or RAD]*
>
> *(repeat for each tradeoff)*
>
> Which options are you leaning toward?

Wait for the user's response. Merge all corrections, additions, and tradeoff decisions before proceeding to Step 5. Record each chosen option — it will inform the Technical Breakdown.

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

## Step 6 — Write TECHNICAL_DESIGN.md

Write to `TECHNICAL_DESIGN.md` in the current directory. Follow `OUTPUT_TEMPLATE.md`. Key rules:

- **Executive Summary** — plain language, no repo names, include: goal, effort totals (raw + buffered), success criteria, and a short RAD summary (2–4 bullets max) — one sentence each on the most significant risk, the key assumption the estimate depends on, and any hard dependency that could shift the timeline.
- **Scope** — in scope and out of scope lists.
- **Milestones** — deliverable, business value, Engineering / Operational / Rollout effort, risk flag.
- **RAD** — every item cites its source (PRD, Transcript, or User).
- **Technical Breakdown** — one section per repo, specific files/areas, one-line rationale each. Open with a "Design decisions" line per repo noting which tradeoff option was chosen and why.
- **Cross-repo map** — only if `--root`.
- **Engineer Updates** — empty table, leave for the team.

End with:
> **Share with PMs / leadership:** Executive Summary, Scope, Milestones
> **Share with engineers:** full document
