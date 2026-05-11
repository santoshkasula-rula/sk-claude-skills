---
name: estimate-effort
description: Generates a PM-readable, engineer-actionable implementation plan from a PRD and/or Zoom transcript. Runs an interactive RAD interview, builds value-based milestones with calendar dates, and maps work across multiple repos when run at the root level.
disable-model-invocation: true
argument-hint: "<prd-file> [transcript-file] [--root]"
allowed-tools: Read Bash(find *) Bash(ls *) Write
---

# estimate-effort

Generate an implementation plan from: $ARGUMENTS

Parse arguments:
- Arg 1: path to the PRD file (required)
- Arg 2: path to the Zoom transcript file (optional but strongly recommended)
- `--root` flag (optional): enable multi-repo discovery and cross-repo dependency mapping

If the PRD path is missing or unreadable, stop and ask for it. If no transcript is provided, note that risk and assumption signals will be limited — ask the user if they want to proceed without it.

Read `OUTPUT_TEMPLATE.md` from the same directory as this skill before writing the output file.

---

## Step 1 — Normalize inputs

**Read the PRD** in full.

**If a transcript is provided:** Read it, then normalize it before extracting signals:
- Strip timestamps, speaker labels, and `[inaudible]` / `[crosstalk]` markers
- Merge fragmented sentences split across speaker turns into coherent statements
- The result is a clean prose summary of what was discussed — use this for Step 3, not the raw text

---

## Step 2 — Repo discovery (only if `--root`)

Run:

```bash
find . -maxdepth 3 \( -name "package.json" -o -name "go.mod" -o -name "requirements.txt" -o -name "Cargo.toml" -o -name "pom.xml" \) ! -path "*/node_modules/*" ! -path "*/.git/*"
```

List each service/repo found. Then scan for shared library imports or cross-service references to identify which repos affect each other. Note any repo that appears to be a shared dependency.

---

## Step 3 — Extract signals

**From the PRD**, extract:
- Project name and objective
- Key features and user stories
- Business value statements and success metrics (exact numbers if present, e.g. "reduce drop-off by 15%")
- Anything explicitly listed as out of scope

**From the normalized transcript** (if available), extract:
- Technical components and systems mentioned
- **Anxiety markers** — phrases like: *"I'm worried about"*, *"legacy"*, *"not sure how long"*, *"that's tricky"*, *"this is risky"*, *"depends on"*, *"blocked by"*, *"we'd need to check with"*, *"I've never touched that"*
- Named blockers, external dependencies, or third-party requirements
- Any open questions the team raised without resolution

**Conflict rule:** If the PRD describes something as straightforward but the transcript contains an anxiety marker about the same component, **the anxiety marker wins**. Flag that component as high risk regardless of what the PRD says. The transcript is ground truth from the people doing the work.

For every anxiety marker found, record the component it refers to — it will receive a 🔴 risk flag and a time buffer in the milestones.

---

## Step 4 — RAD interview (mandatory — do not skip, do not assume answers)

Ask all questions in a single message. Wait for the user's complete response before continuing to Step 5.

> **Before I write the plan, I need a few ground-truth answers. Please fill these in:**
>
> 1. **Risks** — What is the riskiest or most uncertain part of this work? (e.g. legacy systems, unclear requirements, unfamiliar tech)
> 2. **Assumptions** — What are we assuming is already true or in place? (e.g. infra, staging access, API contracts, team availability)
> 3. **Dependencies** — What are we blocked on or waiting for? (e.g. another team's PR, a third-party approval, a shared service not yet built)
> 4. **Scope boundary** — What are we explicitly NOT building in this phase? (List anything adjacent that might be assumed.)
> 5. **Success criteria** — How will we know this is done and working? (e.g. metric targets, stakeholder sign-off, a specific user journey working end-to-end)

---

## Step 5 — Design milestones

Create 3–5 incremental milestones. Each milestone must:
- Deliver a concrete, demonstrable artifact — not "work in progress"
- Be sequenced so each one unblocks the next
- Map explicitly to a business value or success criterion from the PRD
- Include a **target completion date** calculated from the user's stated start date
- Break effort into three distinct buckets — never roll them together:
  - **Engineering:** heads-down coding, design, and code review
  - **Operational:** deployment, config changes, infrastructure setup, secrets/env provisioning
  - **Rollout:** QA, testing cycles, feature flag ramp, stakeholder demo, incremental rollout steps
- Carry a risk indicator: 🔴 if it touches an anxiety-marker component or RAD-flagged risk, 🟡 for medium uncertainty, 🟢 for well-understood work

**Buffer rule:** Sum each effort column independently across all milestones. Add 20% to the Engineering total only — Operational and Rollout estimates are typically known quantities and should not be inflated. Show raw and buffered Engineering totals separately in the Executive Summary. Never embed the buffer inside individual milestones.

If `--root`: assign each milestone's tasks to the responsible repo(s). Sequence milestones to respect cross-repo dependencies — a repo that provides a contract must reach its milestone before dependent repos can proceed.

---

## Step 6 — Write IMPLEMENTATION_PLAN.md

Write the plan to `IMPLEMENTATION_PLAN.md` in the current directory, following the structure in `OUTPUT_TEMPLATE.md`.

Content rules:

- **Executive Summary**: non-technical language throughout. Repo names must not appear here — use business system names (e.g. "checkout flow", "payment processing"). Include: business goal, raw estimate, buffered estimate, a one-sentence risk callout if any milestone is 🔴, and success criteria.
- **Scope section**: include both "In scope" and "Out of scope" lists, sourced from the PRD and the user's Step 4 answer.
- **Milestones table**: every row has a deliverable, a "why this matters" business value statement, and a risk indicator. Effort is split across three columns: Engineering, Operational, and Rollout. Never combine them.
- **RAD section**: every entry must cite its source (Transcript, PRD, or RAD interview). Combine all three sources.
- **Technical Breakdown**: one section per repo; list specific files or areas to change with a one-line rationale. Enough for an engineer to start, not a full spec.
- **Cross-repo dependency map** (if `--root`): show which milestone in which repo must complete before another can start.
- **Engineer Updates table**: leave empty rows — this is a living document.

After writing the file, output a two-line routing note:
- "Share with PMs / leadership: Executive Summary, Scope, Milestones"
- "Share with engineers: full document"
