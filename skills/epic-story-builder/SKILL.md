---
name: epic-story-builder
description: Takes an Effort Breakdown Analysis and generates a weekly-demo-sized Epic and Story list. Each story delivers a meaningful, demonstrable increment. Output is a markdown file ready for Jira import.
disable-model-invocation: true
argument-hint: "[effort-breakdown-file]"
allowed-tools: Read Write
---

# epic-story-builder

Read `OUTPUT_TEMPLATE.md` from `${CLAUDE_SKILL_DIR}`.

---

## Step 1 — Locate the Effort Breakdown Analysis

If a file path is in $ARGUMENTS, read it directly.

If no argument provided, ask:
> "Path to your Effort Breakdown Analysis file?"

Read the file. If unreadable, ask again.

---

## Step 2 — Ask one clarifying question

> "Any context on the team before I size the stories — team size, any engineers part-time on this, or any known sprint constraints? (or skip)"

Wait for the response. If skipped, proceed with no assumptions about team size.

---

## Step 3 — Extract from the Effort Breakdown Analysis

From the file, extract:

- Project name
- Milestones — name, deliverable, business value, effort buckets (Engineering / Operational / Rollout), risk flag
- Technical Breakdown — per repo: files/areas, change needed, rationale, design decisions
- RAD — risks, assumptions, dependencies (with sources)
- Success criteria

---

## Step 4 — Build Epics and Stories

**One Epic per milestone.** Epic name = milestone name from the document — preserve exact language.

**Sizing rule:** Each story must be completable and demonstrable within one week by one engineer. Apply this strictly:
- Group related file changes into one story if together they produce a visible, working increment
- Split a change into multiple stories if it would take more than a week alone
- Operational and Rollout work get their own stories — never bundle with Engineering stories

**Each story requires:**
- A title in plain action language (verb + what it does for the user or system)
- Type: `Engineering` · `Operational` · `Rollout`
- Repo (or `—` for cross-cutting)
- Effort in days
- **Demo scenario** — one sentence: what does the engineer show on Friday? Make it concrete. If nothing is visibly demonstrable, the story is too small or too internal — merge it or reframe it.
- **Done when** — one line acceptance criterion
- RAD flag if the story touches a risk, assumption, or dependency from the analysis (cite which one)

**Story ordering within an Epic:** sequence by dependency — stories that unblock others come first.

**If `--root` work:** note the repo for every story. Flag cross-repo stories explicitly.

---

## Step 5 — Write output file

Write to `Epics and Stories - [Project Name].md` following `OUTPUT_TEMPLATE.md`.

The markdown tables are the single source of truth — humans edit them freely. YAML is generated on the fly by `/jira-import` when needed.

End with:
> **Next step:** Review story list with the team, adjust sizing, then run `/jira-import` to push to Jira. *(coming soon)*
