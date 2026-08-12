---
name: weekly-eng-report
description: Generates a weekly Engineering Leader report organized by ENGKR initiative, showing features/Epics in flight, Epic due dates bucketed by month, story point health, and GitHub PR activity — across Partnerships, Deals, New Verticals, and MarTech teams.
allowed-tools:
  - Bash
  - mcp__atlassian__getAccessibleAtlassianResources
  - mcp__atlassian__searchJiraIssuesUsingJql
  - mcp__atlassian__getJiraIssue
  - Write
---

# weekly-eng-report

Generate a weekly Engineering Leader report organized by H2 Key Result (ENGKR initiative). For each KR, show which Epics (features) are in flight, when each Epic is due, story point health (estimated / done / this week / remaining), and supporting GitHub PR activity — so leadership can see at a glance what is being built, when it ships, and how fast the team is moving.

**Primary lens:** Key Results → Epics → delivery months.
**Secondary lens:** GitHub activity and blockers as supporting context under each KR.
**Target audience:** Engineering Leader (VP/Director level). Translate data into delivery insights. Do not produce a flat ticket list.

---

## Configuration

### GitHub Teams (org: `pathccm`)
| Slug | Coverage |
|------|----------|
| `partnerships` | PAR board contributors |
| `partnership-deals` | DEA board contributors |
| `partnership-new-verticals` | NEV board contributors |
| `martech` | MARTECH board contributors |

### Jira Boards
| Team | Project Key | Board ID |
|------|-------------|----------|
| Partnerships Core | PAR | 1205 |
| Partnership Deals | DEA | 1042 |
| New Verticals | NEV | 2103 |
| MarTech | MARTECH | 1073 |

Jira base URL: `https://rula.atlassian.net`

### H2 2026 Sprint Calendar
Sprints run 2 weeks. Use this table to name the current sprint and understand where the week falls in the H2 sequence.

| Sprint | Start | End | Notes |
|--------|-------|-----|-------|
| 14 | 2026-07-06 | 2026-07-19 | |
| 15 | 2026-07-20 | 2026-08-02 | |
| 16 | 2026-08-03 | 2026-08-16 | |
| 17 | 2026-08-17 | 2026-08-30 | |
| 18 | 2026-08-31 | 2026-09-13 | Labor Day week |
| 19 | 2026-09-14 | 2026-09-27 | |
| 20 | 2026-09-28 | 2026-10-11 | |
| 21 | 2026-10-12 | 2026-10-25 | |
| 22 | 2026-10-26 | 2026-11-08 | |
| 23 | 2026-11-09 | 2026-11-22 | |
| 24 | 2026-11-23 | 2026-12-06 | Thanksgiving week |
| 25 | 2026-12-07 | 2026-12-20 | |
| 26 | 2026-12-21 | 2027-01-03 | Christmas / Winter Break |

### H2 Strategic OKR Initiatives
Active H2 focus areas. Used as fallback descriptions if `getJiraIssue` fails for an ENGKR.

| Initiative ID | Fallback Description | Primary Teams |
|---------------|----------------------|---------------|
| ENGKR-53 | Partnerships Platform Scalability & Self-Serve | PAR |
| ENGKR-54 | Provider Network Expansion (New Verticals) | NEV |
| ENGKR-55 | MarTech Attribution & Campaign Infrastructure | MARTECH |
| ENGKR-56 | Deals & Contracting Automation | DEA |
| ENGKR-57 | Core Platform Reliability & Observability | PAR, DEA |

**H1 tail-end initiatives (historical context only):** ENGKR-6, ENGKR-7, ENGKR-8

OKR timeline: https://rula.atlassian.net/jira/plans/1256/scenarios/1256/timeline

### Epic → ENGKR project theme mapping
Use these themes to infer which ENGKR an Epic maps to when there is no explicit label:
- **EAP expansion** (Curalinc Basic, BHS, ComPsych, WPO) → ENGKR-56
- **EHR integrations** (Auto Trigger Referrals, Writeback, Chrome Extension) → ENGKR-54
- **PCP Referral flow** (Notifications, DBT, FSF) → ENGKR-53
- **MarTech attribution** (Segment Unify, Affiliate Tracking, Iterable) → ENGKR-55
- **Platform migrations** (EKS/ECS, eFax, DBT 2.0) → ENGKR-57
- **Kaiser integrations** (Referral Self-Serve, CO/HI Skip Reg) → ENGKR-53 / ENGKR-54
- **Employer portal / utilization metrics** → ENGKR-54
- **Attribution service / sales portal** → ENGKR-54

### H2 2026 Team Roster & Project Sequencing
Use as **rough planning context** — not a hard schedule. Cross-reference against actual Jira data to identify divergence.

#### Core / PAR
| Person | Planned H2 Projects (rough sequence) |
|--------|--------------------------------------|
| Ashley Huntsman | DBT 2.0 (Sprint 14–15), then TBD |
| Diana Trifonova | PCP Comms Hypercare (carryover), PST exploration (Sprint 14–15) |
| Imran Mhlanga | EKS/ECS Migration (Sprint 14), Waiting Room (Sprint 14–15) |
| Kevin Talley | In-Person Psych Carry-Over (Sprint 14), Waiting Room (Sprint 15) |
| Lauren Wholey | EAP: DRB Update (Sprint 14) |
| Sahal Asghar | EAP: RefAuth Audit Logs (Sprint 14), New Payer Batch (Sprint 15+) |
| Jason Bennett | eFax Migration (Sprint 14–15) |

#### Deals / DEA
| Person | Planned H2 Projects (rough sequence) |
|--------|--------------------------------------|
| Lauren Fuller | BHS (Sprint 15–16), Redesigned Appt Confirmation Series w/ Provider Intros (Sprint 16–19) |
| Justin Choi | BHS (Sprint 14), Kaiser Referral Self-Serve (Sprint 15–18) |
| Ryan Quant | ECS/EKS Migration (Sprint 14–15), EAP: Curalinc Basic (Sprint 16–20) |
| Anton | Kaiser Referral Self-Serve (Sprint 14–18) |
| Josh | EAP: Curalinc Basic (Sprint 14–20) |

#### New Verticals / NEV
| Person | Planned H2 Projects (rough sequence) |
|--------|--------------------------------------|
| Jay Schaffer | Utilization Metrics in Employer Portal (carryover), PCP Referral Notification Expansion M2 (Sprint 14–15), PCP Referral Notification Preferences M3 (Sprint 16–17) |
| Alejandro | Patient Progress Report: Internal Tool M1 (Sprint 14–15), Patient Progress Report: Provider Comms M2 (Sprint 15–16) |
| Iris Lam | Patient Progress Report: Internal Tool M1 (carryover → Sprint 14), EHR Auto Trigger Referrals (Athena) (Sprint 15–18) |
| Mitch | Redesigned Appointment Reminder Series (carryover → Sprint 15), Kaiser CO/HI Skip Reg. (Sprint 16) |
| Dimas | EHR Writeback (carryover → Sprint 14), EHR Auto Trigger Referrals (Athena) (Sprint 15–18) |
| Masha | Type Ahead PCP in Reg Form (Sprint 14–16) |

#### MarTech / MARTECH
| Person | Planned H2 Projects (rough sequence) |
|--------|--------------------------------------|
| Jackson Gratwohl | Segment Unify carryover (Sprint 14–15), Real-Time Appt Availability in Iterable (Sprint 16–17) |
| Zach Herr | CMP carryover (Sprint 14), Dynamic Pixel Deployment (Sprint 15–16) |
| Arturo Campos | Iterable - Next Appointment Scheduled (Sprint 14–16), Real-Time Appt Avail in Iterable (Sprint 16–17) |
| Brunno Silva | Affiliate Publisher-Level Tracking & Attribution / Impact (Sprint 14–15), historicalCareTypes (Sprint 16–17) |
| Rafael Lima | Iterable Provider Catalog 2.0 (Sprint 14–15), Expose Bot Traffic (Sprint 15–17) |

### Output directory
`/Users/skasula/source-code/markdown-files/`

---

## Step 0 — Resolve Cloud ID and date context

1. Call `getAccessibleAtlassianResources` to get the Atlassian Cloud ID. Store it for all subsequent MCP calls.
2. Determine the report week: if the user passed `--week=N`, use that sprint number. Otherwise use today's date against the sprint calendar above to identify the current sprint. Calculate the trailing 7-day window (`SINCE_DATE` = today minus 7 days, formatted as `YYYY-MM-DD`; `TODAY` = today's date).

---

## Step 1 — Fetch GitHub team members

For each of the four GitHub teams, resolve current membership:

```bash
gh api orgs/pathccm/teams/partnerships/members --jq '[.[].login]'
gh api orgs/pathccm/teams/partnership-deals/members --jq '[.[].login]'
gh api orgs/pathccm/teams/partnership-new-verticals/members --jq '[.[].login]'
gh api orgs/pathccm/teams/martech/members --jq '[.[].login]'
```

If a team returns 404 or empty, note it and continue — do not abort.

Build a lookup map: `github_login → team_name`. For members on multiple teams, assign to all applicable teams.

---

## Step 2 — Fetch GitHub PR activity (trailing 7 days)

Search for PRs authored by team members updated within the trailing window. Run two queries per member (open and closed) and combine:

```bash
for state in open closed; do
  gh search prs --author=LOGIN --state=$state \
    --updated="SINCE_DATE..TODAY" \
    --json number,title,state,url,author,repository,createdAt,updatedAt,closedAt,isDraft,commentsCount \
    --limit 50 2>/dev/null
  sleep 0.3
done
```

**Filter:** Discard PRs where `repository.nameWithOwner` does not start with `pathccm/`.

**Deduplicate by Jira key:** When the same ticket key appears in multiple PRs, use the non-draft, non-reverted PR as canonical.

**Detect reverts:** Flag PRs whose title starts with `Revert "` or contains `Revert [`. Track these separately as a quality signal.

**Build output maps:**
- `jira_key → [merged PRs this week]` — extracted from PR titles (e.g. `NEV-1009`, `PAR-234`)
- `author_login → team_name` — for per-team GitHub metrics

**Aggregate per team for the Metrics Snapshot:**
- PRs merged this week (non-draft, non-revert, `state=closed`)
- Reverts / rollbacks count
- Avg PR cycle time (`createdAt → closedAt`, report `< 1 day` if under 24 hours)
- Active contributors (unique authors with ≥1 non-draft PR event)
- PRs open >3 days with no close (blocker candidates)

---

## Step 3 — Fetch Jira sprint tickets and Epic data

### 3a — Sprint tickets

Call `searchJiraIssuesUsingJql` with `maxResults: 75` for each board. Request fields: `summary, status, priority, customfield_10016, assignee, parent, issuetype, labels, resolutiondate, updated`.

```jql
project = PAR AND sprint in openSprints() ORDER BY priority DESC
project = DEA AND sprint in openSprints() ORDER BY priority DESC
project = NEV AND sprint in openSprints() ORDER BY priority DESC
project = MARTECH AND sprint in openSprints() ORDER BY priority DESC
```

Also run a stuck-ticket query per board:

```jql
project = NEV AND sprint in openSprints() AND statusCategory != Done
  AND (status = "In Review" OR status = "Blocked" OR labels = "blocked")
  AND updated <= -3d ORDER BY updated ASC
```
(Repeat for PAR, DEA, MARTECH)

### 3b — Fetch Epic metadata

Collect all unique Epic keys from the `parent` field of sprint tickets (these are the direct parent Epics). For each unique Epic key, call `getJiraIssue` with fields: `["summary", "duedate", "labels", "status", "parent"]`.

### 3c — Aggregate per Epic

For each Epic, compute:
- `sp_estimated`: sum of `customfield_10016` across all sprint tickets under this Epic. If **all** tickets have `null` story points, mark as `no_sp: true` (will render as `—` in the table, not `0`).
- `sp_done`: sum of SP where ticket's `statusCategory.name == "Done"`.
- `sp_this_week`: sum of SP where `resolutiondate >= SINCE_DATE` AND ticket's `statusCategory.name == "Done"`.
- `sp_remaining`: `sp_estimated − sp_done` (omit if `no_sp: true`).
- `active_teams`: distinct Jira project keys from sprint tickets (e.g. `["NEV", "PAR"]`).
- `ticket_keys`: list of sprint ticket keys under this Epic.
- `due_month`: extract month name from Epic's `duedate` field (e.g. `"July"`, `"October"`). If `duedate` is null → `"No Due Date"`.

### 3d — Map Epics to ENGKR

For each Epic, determine its ENGKR using this priority order:
1. Explicit ENGKR label on the Epic (e.g. label `ENGKR-54`)
2. Explicit ENGKR label on the Epic's parent (if Epic has a parent)
3. Project theme mapping in the configuration table above (match Epic summary keywords)
4. If no match after 1–3: assign to `Unmapped`

Build final structure: `ENGKR-XX → [list of Epic objects with aggregated data]`

---

## Step 4 — Fetch ENGKR initiative titles

Call `getJiraIssue` for each of ENGKR-53, ENGKR-54, ENGKR-55, ENGKR-56, ENGKR-57 to retrieve their `summary`. Use the summary as the initiative title in the report. If a fetch fails, fall back to the description in the H2 Strategic OKR Initiatives table above.

---

## Step 5 — Synthesize and write the report

Write to `/Users/skasula/source-code/markdown-files/Weekly Eng Report - YYYY-MM-DD.md`.

Write for an Engineering Leader who is not reading Jira or GitHub. Translate data into delivery insights. The primary question to answer per KR: *"What features are being built, when do they land, and how fast is the team moving?"*

```markdown
# Weekly Engineering Report — Sprint [N], [Date Range]
Generated: [YYYY-MM-DD] | Reporting window: [SINCE_DATE] → [TODAY]

---

## Table of Contents
- [Executive Summary](#executive-summary)
- [Key Results — Delivery View](#key-results--delivery-view)
  - [ENGKR-53](#engkr-53--title)
  - [ENGKR-54](#engkr-54--title)
  - [ENGKR-55](#engkr-55--title)
  - [ENGKR-56](#engkr-56--title)
  - [ENGKR-57](#engkr-57--title)
  - [Unmapped Work](#unmapped-work)
- [Blockers & Risks](#blockers--risks)
- [Metrics Snapshot](#metrics-snapshot)

---

## Executive Summary

- **KR Progress:** [Which KRs had the most SP completed this week. Name the Epics that moved.]
- **Delivery risk:** [Any Epic with a due date within 4 weeks that still has significant SP remaining. Be specific: "Epic X is due July 31 with N SP remaining."]
- **Quality signal:** [Revert count and affected tickets. Any Epic with zero SP progress this week despite being in-flight.]
- **Watch item:** [One action item — a blocker to unblock, a due date at risk, or a planning divergence leadership should acknowledge.]

---

## Key Results — Delivery View

> **How to read this table:**
> Each row is an Epic (feature) currently in active sprints. Epics are grouped by their due month.
> - **SP Est.** = total story points estimated for active sprint tickets under this Epic
> - **SP Done** = story points on tickets in Done status
> - **SP This Week** = story points on tickets moved to Done during [SINCE_DATE → TODAY]
> - **Remaining** = SP Est. − SP Done
> `—` means story points are not estimated for this Epic's tickets.

---

### [ENGKR-53](https://rula.atlassian.net/browse/ENGKR-53) — [title from ENGKR-53]

#### July
| Epic | Teams | SP Est. | SP Done | SP This Week | Remaining |
|------|-------|---------|---------|--------------|-----------|
| [Epic summary](https://rula.atlassian.net/browse/EPIC-KEY) | PAR | N | N | N | N |

#### August
| Epic | Teams | SP Est. | SP Done | SP This Week | Remaining |
|------|-------|---------|---------|--------------|-----------|
| ... | ... | ... | ... | ... | ... |

#### No Due Date
| Epic | Teams | SP Est. | SP Done | SP This Week | Remaining |
|------|-------|---------|---------|--------------|-----------|
| ... | ... | ... | ... | ... | ... |

**GitHub this week:** [N PRs merged, N still open] — [Name 2–3 most significant ticket titles merged this week that map to this KR.]

**Observations:** [1–2 sentences: pacing vs. due dates, any Epic with zero activity, divergence from H2 plan.]

---

### [ENGKR-54](https://rula.atlassian.net/browse/ENGKR-54) — [title from ENGKR-54]

#### [Month]
| Epic | Teams | SP Est. | SP Done | SP This Week | Remaining |
|------|-------|---------|---------|--------------|-----------|
| ... | ... | ... | ... | ... | ... |

#### No Due Date
| Epic | Teams | SP Est. | SP Done | SP This Week | Remaining |
|------|-------|---------|---------|--------------|-----------|
| ... | ... | ... | ... | ... | ... |

**GitHub this week:** [summary]

**Observations:** [1–2 sentences]

---

### [ENGKR-55](https://rula.atlassian.net/browse/ENGKR-55) — [title from ENGKR-55]

[same structure]

> Note: MarTech has a hard external deadline — LMT contract ends Sprint 17 (2026-08-30). Flag any MarTech Epic with due date at or before this date that has significant remaining SP.

---

### [ENGKR-56](https://rula.atlassian.net/browse/ENGKR-56) — [title from ENGKR-56]

[same structure]

---

### [ENGKR-57](https://rula.atlassian.net/browse/ENGKR-57) — [title from ENGKR-57]

[same structure]

---

### Unmapped Work

Epics or tickets with no ENGKR mapping. Include a note on why each is unmapped (e.g. "no Epic parent", "Epic label not set").

#### No Due Date
| Epic | Teams | SP Est. | SP Done | SP This Week | Remaining |
|------|-------|---------|---------|--------------|-----------|
| ... | ... | ... | ... | ... | ... |

---

## Blockers & Risks

> Anything stuck >3 days in Blocked or In Review.

### Jira — Stuck Tickets

| Ticket | Epic | Team | Status | Days Stuck | Assignee | Link |
|--------|------|------|--------|-----------|---------|------|
| NEV-NNN | [Epic summary] | NEV | In Review | N days | @name | [link] |

_If none: "No tickets stuck >3 days this week."_

### GitHub — PRs Open >3 Days Without Merge

| PR | Title | Author | Team | Repo | Open Days | Comments | Link |
|----|-------|--------|------|------|-----------|---------|------|
| #NNN | [title] | @login | NEV | repo | N days | N | [link] |

_If none: "No PRs stalled >3 days this week."_

### Pattern Flags
[Only include if a real pattern exists. Check for:
- **Revert cycles** — feature merged and reverted within the window (or same ticket reverted multiple times)
- **Review bottlenecks** — same reviewer on 3+ stalled PRs
- **Initiative blackout** — ENGKR initiative with no Sprint tickets in Done and no PRs merged this week
- **Epics approaching due date with high remaining SP** — flag specifically by Epic name and SP count
- **External deadline risk** — LMT contract ends Sprint 17; flag at-risk MarTech Epics
Omit entirely if none apply.]

---

## Metrics Snapshot

| Metric | PAR | DEA | NEV | MARTECH | Total |
|--------|-----|-----|-----|---------|-------|
| PRs Merged (week) | | | | | |
| Reverts / Rollbacks | | | | | |
| Avg PR Cycle Time | | | | | |
| Active Contributors | | | | | |
| Sprint Tickets Done | | | | | |
| SP Completed (week) | | | | | |
```

---

## Step 6 — Post-write: print summary to chat

After writing the file, print the **Executive Summary** and **Blockers & Risks** sections directly in the chat.

Then output:
> Full weekly report written to `/Users/skasula/source-code/markdown-files/Weekly Eng Report - YYYY-MM-DD.md`

---

## Error handling

- **GitHub team 404:** Note "Team `X` not found in org `pathccm`" and fill GitHub columns with `N/A`.
- **Jira board empty sprint:** Note "No active sprint found for [PROJECT]" — do not fabricate counts.
- **GitHub rate limit:** Wait 60s, retry once. If still failing, mark "GitHub data partial — rate limited" in the report header.
- **ENGKR issue fetch fails:** Fall back to the description in the H2 Strategic OKR Initiatives table.
- **All SP null for an Epic:** Render `—` in all SP columns. Add footnote under the table: "_Story points not estimated for this Epic's sprint tickets._"
- **No PRs for a team member:** Normal — do not flag them as inactive unless the entire team has zero PR activity.
- **Epic maps to multiple ENGKRs** (e.g. Kaiser work → ENGKR-53 and ENGKR-54): show the Epic row under both KR sections with a note "(shared)".

---

## Invocation

The skill is invoked via `/weekly-eng-report`. Optional arguments:
- `--sprint=N` — generate the report for sprint N (defaults to current sprint based on today's date)
- `--since=YYYY-MM-DD` — override the start of the trailing window (defaults to 7 days ago)
- `--team=SLUG` — scope GitHub fetch to a single team slug (for dry runs; Jira still fetches all boards)

Examples:
```
/weekly-eng-report
/weekly-eng-report --sprint=14
/weekly-eng-report --since=2026-06-22 --team=partnership-new-verticals
```
