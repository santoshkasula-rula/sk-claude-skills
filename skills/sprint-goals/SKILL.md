---
name: sprint-goals
description: Generates a unified sprint goals summary across 4 team boards (PAR, MARTECH, DEA, NEV) for Eng and PM leadership — captures in-progress work, focus areas, and blockers from active Jira sprint tickets.
allowed-tools:
  - mcp__atlassian__getAccessibleAtlassianResources
  - mcp__atlassian__searchJiraIssuesUsingJql
  - mcp__atlassian__getJiraIssue
  - Write
---

# sprint-goals

Generate a unified sprint goals summary for Eng and PM leadership across the four Partnerships team boards.

**Boards:**

| Team | Project | Board ID | Board URL |
| :--- | :--- | :--- | :--- |
| PAR | PAR | 1205 | https://rula.atlassian.net/jira/software/c/projects/PAR/boards/1205 |
| MARTECH | MARTECH | 1073 | https://rula.atlassian.net/jira/software/c/projects/MARTECH/boards/1073 |
| DEA | DEA | 1042 | https://rula.atlassian.net/jira/software/c/projects/DEA/boards/1042 |
| NEV | NEV | 2103 | https://rula.atlassian.net/jira/software/c/projects/NEV/boards/2103 |

Jira base URL: `https://rula.atlassian.net`
Epic URL pattern: `https://rula.atlassian.net/browse/[EPIC-KEY]`
Issue URL pattern: `https://rula.atlassian.net/browse/[ISSUE-KEY]`

---

## Step 0 — Resolve Cloud ID

Call `getAccessibleAtlassianResources` to get the Atlassian Cloud ID. Store it and pass it as `cloudId` in every subsequent MCP call.

---

## Step 1 — Fetch active sprint tickets for all four boards

For each project, query the active sprint in parallel:

```jql
project = PAR AND sprint in openSprints() AND statusCategory != Done ORDER BY priority DESC
project = MARTECH AND sprint in openSprints() AND statusCategory != Done ORDER BY priority DESC
project = DEA AND sprint in openSprints() AND statusCategory != Done ORDER BY priority DESC
project = NEV AND sprint in openSprints() AND statusCategory != Done ORDER BY priority DESC
```

Use `searchJiraIssuesUsingJql` with `maxResults: 50` per query. Run all four in parallel.

From each result, capture per ticket:
- Key, Summary, Status, Priority, Story Points, Assignee, Epic link / Epic name, Labels, Issue type

---

## Step 2 — Identify sprint name and focus

From the search results, extract the active sprint name for each board (it will appear in sprint field metadata). Use it as the sprint header in the output.

For each team, identify the dominant focus areas by grouping tickets under their Epic. If Epic data is sparse, group by theme inferred from ticket summaries. Aim for 2–4 focus areas per team — not a flat ticket list.

---

## Step 3 — Fetch Epic context for top items (selective)

For tickets with an Epic link, call `getJiraIssue` on the Epic to get its summary and description. Limit to the top 2–3 Epics per team (highest ticket count or highest priority). Run in parallel across all teams.

Use Epic context to write richer, more meaningful goal statements — not just ticket titles.

Store each Epic's key so it can be linked in the output as `https://rula.atlassian.net/browse/[EPIC-KEY]`.

---

## Step 4 — Write output file

Write to `/Users/skasula/source-code/markdown-files/Sprint Goals - [YYYY-MM-DD].md`, using today's date (e.g. `Sprint Goals - 2026-05-27.md`).

Structure:

```
# Sprint Goals — [Sprint Name]
Generated: [YYYY-MM-DD]

## Table of Contents
- [How to Use This Doc](#how-to-use-this-doc)
- [PAR](#par--[tagline-slug])
- [MARTECH](#martech--[tagline-slug])
- [DEA](#dea--[tagline-slug])
- [NEV](#nev--[tagline-slug])
- [Cross-Team Dependencies](#cross-team-dependencies) *(if applicable)*
- [Leadership Summary](#leadership-summary)

---

## How to Use This Doc
[2 sentences: audience, purpose — focus areas for leadership to track progress and remove blockers]

---

## PAR — [2–4 word team focus tagline]
🔗 [View Board](https://rula.atlassian.net/jira/software/c/projects/PAR/boards/1205) · Sprint: [Sprint Name]

**Sprint focus:** [1–2 sentences synthesizing what this team is shipping this sprint, in plain language.]

### Key Initiatives
- **[[Epic Name]](https://rula.atlassian.net/browse/[EPIC-KEY])** — [1 sentence: what they're working toward and why it matters]
- **[[Epic Name]](https://rula.atlassian.net/browse/[EPIC-KEY])** — [1 sentence]
- **[Theme Name]** — [1 sentence — use plain text if no Epic key available]

---

## MARTECH — [2–4 word team focus tagline]
🔗 [View Board](https://rula.atlassian.net/jira/software/c/projects/MARTECH/boards/1073) · Sprint: [Sprint Name]

**Sprint focus:** [...]

### Key Initiatives
- **[[Epic Name]](https://rula.atlassian.net/browse/[EPIC-KEY])** — [1 sentence]
- ...

---

## DEA — [2–4 word team focus tagline]
🔗 [View Board](https://rula.atlassian.net/jira/software/c/projects/DEA/boards/1042) · Sprint: [Sprint Name]

**Sprint focus:** [...]

### Key Initiatives
- **[[Epic Name]](https://rula.atlassian.net/browse/[EPIC-KEY])** — [1 sentence]
- ...

---

## NEV — [2–4 word team focus tagline]
🔗 [View Board](https://rula.atlassian.net/jira/software/c/projects/NEV/boards/2103) · Sprint: [Sprint Name]

**Sprint focus:** [...]

### Key Initiatives
- **[[Epic Name]](https://rula.atlassian.net/browse/[EPIC-KEY])** — [1 sentence]
- ...

---

## Cross-Team Dependencies
[Only include if tickets across teams reference each other or share an Epic. Otherwise omit this section entirely.]

---

## Leadership Summary
[3–5 bullet points across all teams: the most important things leadership needs to know this sprint — biggest bets, highest-risk items, anything that needs a decision or unblocking from above.]
```

**Linking rules:**
- Always link Epic names using `[Epic Name](https://rula.atlassian.net/browse/EPIC-KEY)` when an Epic key is known.
- If an Epic key is not available, use plain bold text — do not fabricate a URL.
- Board links are hardcoded per the board table above — always include them.
- TOC anchor slugs must match the actual heading text (lowercase, spaces → hyphens).

**Tone:** Plain language. Write for someone who is not reading Jira. Goal names should describe outcomes, not tasks. Avoid ticket IDs in the goals section.

---

## Step 5 — Print summary to chat

After writing the file, print the full **Leadership Summary** section directly in the chat response so leadership gets the key points without opening the file.

Then output:
> Full sprint goals written to `/Users/skasula/source-code/markdown-files/Sprint Goals - [YYYY-MM-DD].md`
> To share: copy the file or paste the Leadership Summary above into your standup doc / Slack.
