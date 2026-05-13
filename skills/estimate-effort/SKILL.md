---
name: estimate-effort
description: Guides engineers through effort estimation as a thought partner — collects documents, probes for unstated risks and assumptions via targeted RAD questions, then writes a PM-readable implementation plan.
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
| Risks — anxiety markers: *"worried about"*, *"legacy"*, *"not sure how long"*, *"tricky"*, *"depends on"*, *"blocked by"*, *"never touched"* | Transcript / docs |
| Named blockers, external dependencies, open questions | Transcript / docs |
| Assumptions stated or implied | All sources |

**Conflict rule:** Transcript anxiety markers override PRD optimism on the same component — always.

---

## Step 4 — RAD thought-partner

Documents capture what was written down. This step surfaces what wasn't. Send one message that pre-fills what you found and asks 2–3 targeted questions to probe for gaps.

> **Here's what I found — correct anything, then I have a few questions:**
>
> **Risks:** [list with source, or "none found"]
> **Assumptions:** [list with source, or "none found"]
> **Dependencies:** [list with source, or "none found"]
> **Out of scope:** [list, or "not explicit — what should be excluded?"]
> **Success criteria:** [list, or "not stated — how will you know this is done?"]

Then add 2–3 targeted questions based on what's genuinely missing or uncertain. Pick from the set below — skip any where the documents already have a clear answer:

- If risks are thin or anxiety markers appeared on a specific component: *"What's the trickiest part of this — what would make it take twice as long?"* or *"You mentioned [X] is tricky — what makes it hard and have you touched it before?"*
- If assumptions are thin: *"What has to be true for this estimate to hold?"*
- If dependencies are sparse or service boundaries are unclear: *"Who or what outside your team needs to cooperate for this to ship?"*
- **Always ask about dependent repos/services** unless the documents already enumerate which services need changes: *"Which other repos or services will need changes — even small ones like config updates, contract changes, or new API calls?"* — use the answer to populate the Technical Breakdown and flag cross-service sequencing risks.
- If scope feels ambiguous: *"What's adjacent to this that someone might assume is included?"*
- If there's a new system, integration, or unfamiliar area: *"Has your team built something like [X] before, or is this new ground?"*
- **Always include one design/tradeoff probe** unless the documents already describe the implementation approach in detail: *"Have you thought through how you'd approach [key technical decision]? Any alternatives you're weighing or already ruled out?"* — tailor [key technical decision] to the most significant architectural or design choice implied by the scope (e.g. sync vs async, new service vs extending existing, client-side vs server-side logic, migration strategy).

**Rules:**
- Never ask more than 3 questions total. The dependent-repos question and the design/tradeoff probe are both "always ask" — if both apply and nothing else is missing, ask only those two.
- Never re-ask for something the documents already answered.
- Frame questions to help the engineer think, not just fill a form.
- If the engineer describes a design approach, follow up with one challenge: *"What's the main risk with that approach?"* or *"Did you consider [obvious alternative] — what ruled it out?"* — then move on.

Wait for the user's response. Update RAD with anything new before continuing.

---

## Step 4b — Tradeoff analysis (opt-in)

After RAD is confirmed, ask once:

> **Run tradeoff analysis?** (default: no)
> Scans the codebase for design patterns and surfaces 2–4 decisions you'll need to make before implementation.
> - **Yes** — full scan, grounded options
> - **No / skip** — you can add design notes manually

- If **yes**: scan the codebase (Step 4c), then proceed to Step 5.
- If **no** or no response: proceed directly to Step 5.
- If the user pastes design constraints: record them, skip scan, proceed to Step 5.

---

## Step 4c — Codebase tradeoff scan (only if yes)

Scan areas the PRD touches. Look for:
- How similar features are structured (routing, layering, data access)
- How UI components interact with backend services
- How existing cross-service calls are made (sync vs async, retry, error handling)
- Tech debt signals near areas this feature will touch

Use `find`, `ls`, and `Read` on key files. Do not scan exhaustively.

Form 2–4 tradeoffs. Each must:
- Be grounded in something observed — not generic best practices
- Present Option A / Option B with a one-line consequence each
- Flag which option is safer given the RAD risks

> **Design tradeoffs — your call:**
>
> *[Tradeoff 1 — what you observed, what the decision is]*
> - **A:** [what it is] → [consequence]
> - **B:** [what it is] → [consequence]
> - *Leans toward A/B because [RAD risk or codebase reason]*

Wait for the user's response. Record each decision — it will inform the Technical Breakdown.

---

## Step 5 — Design milestones

Create 3–5 milestones. Use the language from the documents — names, deliverables, and business value must reflect the actual words and goals from the PRD, transcript, and context provided. Do not substitute generic placeholders.

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

## Step 6 — Write output file

Write to `Effort Breakdown Analysis - [Project Name].md` in the current directory, using the actual project name extracted from the documents. Follow `OUTPUT_TEMPLATE.md`. Key rules:

- **Executive Summary** — plain language, no repo names, include: goal, effort totals (raw + buffered), success criteria, risk summary (one sentence if any 🔴 milestone), assumptions (key assumptions the estimate depends on), and dependencies (hard blockers that could shift the plan).
- **Scope** — in scope and out of scope lists.
- **Milestones** — deliverable, business value, Engineering / Operational / Rollout effort, risk flag.
- **RAD** — every item cites its source (PRD, Transcript, or Engineer). Items added during the thought-partner conversation are sourced as "Engineer".
- **Technical Breakdown** — one section per repo, specific files/areas, one-line rationale each. Open with a "Design decisions" line per repo noting which tradeoff option was chosen and why.
- **Cross-repo map** — only if `--root`.
- **Engineer Updates** — empty table, leave for the team.

End with:
> **Share with PMs / leadership:** Executive Summary, Scope, Milestones
> **Share with engineers:** full document
