# estimate-effort

Analyzes PRDs and Zoom transcripts to generate a multi-repo implementation plan with RAD (Risks, Assumptions, Dependencies) analysis.

## Trigger

User invokes `/estimate-effort` or asks to "estimate effort", "create an implementation plan", or "analyze this PRD/transcript".

## Inputs

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `prd_path` | string | Path to the PRD file |
| `transcript_path` | string | Path to the Zoom transcript file |
| `target_output` | string | Output file name (default: `IMPLEMENTATION_PLAN.md`) |
| `mode` | `root` \| `local` | `root` = scan for all repos; `local` = current repo only |

## Steps

### 1. Read Inputs

Read the PRD and transcript files provided by the user.

### 2. Contextual Scan (if `mode == "root"`)

Run a discovery phase to map all services in scope:

```bash
find . -maxdepth 2 \( -name "package.json" -o -name "go.mod" -o -name "requirements.txt" \)
```

Then scan for cross-repo imports or shared library references to identify ripple-effect risks (e.g., a change in `repo-a` that breaks `repo-b`).

### 3. Transcript Sentiment Analysis

Parse the transcript for "Anxiety Markers" — phrases that signal engineer concern:

- Pattern examples: *"I'm worried about..."*, *"that part scares me"*, *"legacy X is fragile"*, *"not sure how long that'll take"*
- When an anxiety marker is detected for a component, automatically:
  - Upgrade the **Risk** level for that milestone
  - Add a time buffer to the estimate for that area

### 4. RAD Interview (Mandatory — Do Not Skip)

Before generating the final plan, stop and ask the engineer:

> **"Wait! Before I finalize the math, I need the 'Ground Truth' from the engineers. Please answer these three things:"**
>
> - **Risks:** What is the 'scariest' part of this codebase relative to this feature?
> - **Assumptions:** Are we assuming any infrastructure (e.g., CI/CD, Secrets, staging env) is already in place?
> - **Dependencies:** Are we blocked by another team's PR or a third-party API approval?

Incorporate the answers into the RAD section and adjust effort estimates accordingly.

### 5. Generate `IMPLEMENTATION_PLAN.md`

Write the output file using the template below. Apply a **20% RAD buffer** to the total estimated effort.

---

## Output Template

```markdown
# 🗺️ Implementation Plan: [Project Name]
**Date:** [Today's Date] | **Current Status:** Draft (Awaiting Engineer Sign-off)

---

## 🎯 Executive Summary (PM Focus)

* **Primary Value:** [Business value derived from PRD]
* **Total Estimated Effort:** [X] Weeks (including 20% RAD buffer)
* **Impacted Systems:** `[repo-1]`, `[repo-2]`, `[repo-3]`

---

## 🗓️ Value-Based Milestones

| Milestone | Deliverable | Business Value | Est. Effort |
| :--- | :--- | :--- | :--- |
| **M1: Core Contract** | API definitions & DB Schema | Unblocks parallel frontend/backend work. | 4 Days |
| **M2: Happy Path** | End-to-end basic flow | Allows PMs to demo the feature to stakeholders. | 1 Week |
| **M3: Hardening** | Edge cases & Load testing | Ensures production stability. | 3 Days |

---

## ⚠️ RAD Analysis (The "No-Surprises" Section)

### 🔴 Risks
* **[Risk 1]:** [Description]
    * **Mitigation:** [e.g., "Implement feature flag first"]

### 🟡 Assumptions
* **[Assumption 1]:** [e.g., "We assume the `v2/payments` endpoint is idempotent."]
* **[Assumption 2]:** [e.g., "Engineers have access to the staging environment by [Date]."]

### 🔵 Dependencies (Multi-Project)
* **Internal:** [e.g., "`billing-engine` requires the new schema from `db-migrations`."]
* **External:** [e.g., "Awaiting Stripe API key rotation."]

---

## 💻 Technical Breakdown & Patterns

### Repository: `[Repo-Name]`
* **Architectural Suggestion:** [Pattern recommendation based on file structure scan]
* **Key Files to Modify:**
    1. `[path/to/file]` — [What to change and why]
    2. `[path/to/file]` — [What to change and why]

---

## 📝 Engineer Updates

| Date | Engineer | Update/Pivot | New Est. |
| :--- | :--- | :--- | :--- |
| | | | |
```

---

## Notes

- Always run the RAD Interview (Step 4) before writing the output — never skip it.
- Anxiety markers in transcripts take precedence over optimistic estimates from the PRD.
- When `mode == "root"`, dependency detection between repos is required before milestone sequencing.
- The 20% RAD buffer is applied to the **total** effort, not per milestone.
