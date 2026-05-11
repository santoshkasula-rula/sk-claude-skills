---
name: estimate-effort
description: Generates a PM-readable, engineer-actionable implementation plan from a PRD and Zoom transcript. Runs an interactive RAD interview, builds value-based milestones, and maps work across multiple repos when run at the root level.
disable-model-invocation: true
argument-hint: "<prd-file> <transcript-file> [--root]"
allowed-tools: Read Bash(find *) Bash(ls *) Write
---

# estimate-effort

Generate an implementation plan from: $ARGUMENTS

Parse arguments:
- Arg 1: path to the PRD file
- Arg 2: path to the Zoom transcript file
- `--root` flag (optional): enable multi-repo discovery and cross-repo dependency mapping

If either file path is missing or unreadable, ask the user to provide it before continuing.

---

## Step 1 — Read source documents

Read both the PRD and the transcript in full.

---

## Step 2 — Repo discovery (only if `--root`)

Run:

```bash
find . -maxdepth 3 \( -name "package.json" -o -name "go.mod" -o -name "requirements.txt" -o -name "Cargo.toml" -o -name "pom.xml" \) ! -path "*/node_modules/*" ! -path "*/.git/*"
```

From the results, list each service/repo found. Then scan for shared library imports or cross-service references to identify which repos affect each other.

---

## Step 3 — Extract signals from documents

From the **PRD**, extract:
- Project name and objective
- Key features and user stories
- Business value statements and success metrics

From the **transcript**, extract:
- Technical components and systems mentioned
- **Anxiety markers** — phrases like: *"I'm worried about"*, *"legacy"*, *"not sure how long"*, *"that's tricky"*, *"depends on"*, *"blocked by"*, *"we'd need to check with"*
- Named blockers, external dependencies, or third-party requirements
- Any named risks or open questions the team raised

For every anxiety marker found, record the component it refers to. That component will receive a risk flag and a time buffer in the milestones.

---

## Step 4 — RAD interview (mandatory — do not skip, do not assume answers)

Ask the user all three questions together in a single message. Wait for their full response before proceeding to Step 5.

> **Before I write the plan, I need ground-truth input from the team. Please answer all three:**
>
> 1. **Risks** — What is the riskiest or most uncertain part of this work? (e.g. legacy systems, unclear requirements, unfamiliar tech)
> 2. **Assumptions** — What are we assuming is already true or in place? (e.g. infra, staging access, API contracts, team availability)
> 3. **Dependencies** — What are we blocked on or waiting for? (e.g. another team's PR, a third-party approval, a shared service not yet built)

---

## Step 5 — Design milestones

Create 3–5 incremental milestones. Each milestone must:
- Deliver a concrete, demonstrable artifact (not just "work in progress")
- Be sequenced so earlier milestones unblock later ones
- Map to a specific business value from the PRD
- Carry a realistic effort estimate in days or weeks
- Be flagged with a risk indicator if it touches a component from an anxiety marker or RAD answer

Apply a **20% buffer** to the total estimated effort to account for RAD risks.

If `--root`: assign each milestone's tasks to the responsible repo(s) and explicitly note cross-repo dependencies. Sequence milestones to respect those dependencies.

---

## Step 6 — Write IMPLEMENTATION_PLAN.md

Write the plan to `IMPLEMENTATION_PLAN.md` in the current directory using the output structure defined in [`OUTPUT_TEMPLATE.md`](OUTPUT_TEMPLATE.md).

Content rules:
- **Executive Summary**: written for a non-technical audience — no jargon, focus on business value and timeline
- **Milestones table**: every row must have a deliverable, a "why this matters" business value statement, and an effort estimate
- **RAD section**: combine signals extracted from the transcript (Step 3) with the user's answers (Step 4) — cite the source for each item
- **Technical Breakdown**: one section per repo; list the specific files or areas to change with a one-line rationale per entry. Enough detail for an engineer to pick up the work, not a full spec.
- **Cross-repo dependency list** (if `--root`): show which milestones block which, and across which repos
- **Engineer Updates table**: leave empty — this is a living document for engineers to fill in as estimates change

After writing the file, confirm the path and tell the user which section to share with PMs vs. engineers.
