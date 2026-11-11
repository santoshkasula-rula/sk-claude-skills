---
name: incident-postmortem-summary
description: Summarizes Datadog incidents under Santosh's teams — what happened, lessons learned, and action items pulled from each incident's linked postmortem doc (Google Doc or other webpage), enriched with due dates/status from linked Jira tickets and process context pulled from the incident's Slack channel. Opens with a cross-incident lessons-learned executive summary for engineering leadership.
allowed-tools:
  - Bash
  - mcp__atlassian__getAccessibleAtlassianResources
  - mcp__atlassian__getJiraIssue
  - mcp__plugin_datadog_mcp__search_datadog_incidents
  - mcp__plugin_datadog_mcp__get_datadog_incident
  - mcp__slack__slack_search_channels
  - mcp__slack__slack_search_public_and_private
  - mcp__slack__slack_read_channel
  - mcp__slack__slack_read_thread
  - WebFetch
  - Write
---

# incident-postmortem-summary

Generate a per-incident summary for every Datadog incident assigned to Santosh's teams: what happened, the root cause, and the action items — read directly from each incident's linked postmortem doc, not just Datadog's own incident fields (which are often left blank). Action items that reference Jira tickets are enriched with the ticket's live status and due date. Each incident is also enriched with process context pulled from its Slack channel — how the incident was actually declared, escalated, diagnosed, and closed out, which the postmortem doc typically doesn't capture.

**Primary lens:** One card per incident — title (with incident + postmortem links), incident description, lessons learned, action items (Jira link, title, status, due date), and Slack-derived process context.
**Secondary lens:** A cross-incident lessons-learned executive summary for Engineering Leadership — operational effectiveness/inefficiencies and concrete opportunities to improve, not just a volume roll-up. This now spans both root-cause patterns (from postmortem docs) and incident-management process patterns (from Slack) — e.g. severity assigned reactively, coordination silos, undocumented high-stakes calls, no templated postmortem-to-Jira handoff.
**Target audience:** Santosh and Engineering Leadership, for team/leadership visibility into incident follow-through and systemic patterns — not a replacement for the postmortem doc itself, a digest of it.

---

## Configuration

### Datadog team tags (Santosh's teams)
Scope every incident query to this exact set, OR'd together — matches the incidents list Santosh reviews at `https://us5.datadoghq.com/incidents?query=teams%3A...`:

```
epd-partnerships, partner-growth-activation, partner-new-verticals, partner-onboarding-integrations, partner-platform, partnerships-core, partnerships-deals, martech, new-verticals
```

> Note: this matches `/eng-flow-health`'s Datadog team scope. If the two ever need to diverge, keep them independent — don't silently sync.

If a new team tag seems relevant when running this skill, ask before assuming the list above is exhaustive.

### Postmortem doc sources
Postmortem links vary by incident. Detect the link type from its URL and use the matching tool:
- **Google Docs** (`docs.google.com/...`) → `WebFetch`. Only works if the doc is shared "Anyone with the link can view" — if it 403s or redirects to a login page, treat as unfetchable (see Error handling).
- **Anything else** (Notion, a wiki, a plain webpage) → `WebFetch` as a best effort.

This skill does not read Confluence. If a postmortem link points at `*.atlassian.net/wiki/...`, treat it as `unfetchable (Confluence not supported)` rather than attempting to fetch it.

### Slack channel links
Every incident has a dedicated Slack channel named `incident-<public_id>` (e.g. incident 887 → `incident-887`). Build its link as:

```
https://slack.com/app_redirect?channel=incident-<public_id>&team=TFMMYKQU8
```

`public_id` is the numeric incident ID from Datadog (not the UUID `id` field, not the `IR-N` slug). This is a constructed link, not something read from the incident payload — Datadog's incident API has no field for the linked Slack channel.

### Slack process context
Beyond linking the channel, this skill also *reads* it (see Step 3.7) to extract incident-management process signal that postmortem docs almost never capture: how fast declaration/severity actually happened, whether diagnosis chased false leads, cross-team coordination friction, and how the postmortem-to-action-item handoff played out. If Slack tools aren't available in a given environment (no MCP connection, not authenticated), skip Step 3.7 entirely and omit the Slack Context bullet from affected incident cards — don't block the rest of the report on it.

### Jira ticket enrichment
Postmortems frequently link out to Jira tickets for action items (e.g. `rula.atlassian.net/browse/PAR-2543`). For every Jira ticket key or link found in an action item, fetch the ticket via `getJiraIssue` and pull:
- **Status** (e.g. To Do / In Progress / Done)
- **Due date**
- **Assignee** (used as owner if the doc itself doesn't name one)

Use these live Jira fields instead of stale doc text when both exist — a doc might say "Done" from months ago while Jira shows it reopened.

### Output directory
`/Users/skasula/source-code/markdown-files/`

---

## Step 0 — Resolve time window and Cloud ID

1. Default window: trailing **90 days** (`SINCE_DATE` = today minus 90 days, `TODAY` = today). Override with `--days=N` or `--since=YYYY-MM-DD`. Pass `--all` to remove the time filter entirely and pull every incident regardless of age (use when doing a first full sweep).
2. Call `getAccessibleAtlassianResources` to get the Atlassian Cloud ID — needed for any Jira ticket lookups in Step 3.5.

---

## Step 1 — Fetch incidents

Call `search_datadog_incidents`:

```
query: "teams:(epd-partnerships OR partner-growth-activation OR partner-new-verticals OR partner-onboarding-integrations OR partner-platform OR partnerships-core OR partnerships-deals OR martech OR new-verticals)"
from: SINCE_DATE   (omit if --all)
to: TODAY          (omit if --all)
sort: "-created"
```

Collect: `id`, `title`, `severity`, `state`, `created`, `resolved`, `customer_impacted`, `commander`, `services`, `teams`.

If zero incidents come back, stop and report "No incidents found for these teams in [window]." — do not fabricate.

---

## Step 2 — Fetch incident detail and postmortem link

For each incident from Step 1, call `get_datadog_incident` with `include_attachments: true` and `include_follow_ups: true`.

From the response, extract:
- **Postmortem link**: the attachment of type postmortem (or, if absent, any external link whose title/URL suggests a postmortem/RCA doc).
- **Datadog's own fields**: `root_cause`/summary notes if populated (used as fallback, see Step 4).
- **Follow-ups**: Datadog-native action items, if the team tracks them there instead of (or in addition to) inside the postmortem doc.

If an incident has no postmortem attachment and no usable external link, mark it `postmortem: missing` and skip Step 3 for it.

---

## Step 3 — Fetch and read each postmortem doc

For each incident with a postmortem link, fetch its content per the Postmortem doc sources rules above, then extract:
- **Incident description** — the incident summary/timeline narrative section (however the doc labels it: "Summary", "What happened", "Overview", "Timeline"). 2-4 sentences: what broke, what triggered it, what the impact was.
- **Lessons learned** — the retrospective learnings, not just a one-line root cause. Pull from whatever sections the doc uses for this ("Root Cause", "Why did it happen", "Compounding/Contributing Factors", "Lessons Learned", "What went well/poorly"). Synthesize into 2-4 sentences covering both the technical cause and any process/detection/response takeaway — this is the material the executive summary in Step 4.5 draws on, so don't compress it down to just a root-cause one-liner.
- **Action items** — the action items / follow-ups section, including owner and status if the doc tracks them (e.g. a table with Done/Not Started columns). If the doc links out to Jira tickets for action items, keep the ticket key/link — do not paraphrase it away, since Step 3.5 resolves it to live title/status/due date.

If the doc fetch fails (dead link, 404, permission-gated Google Doc, or a Confluence link), mark `postmortem: unfetchable` and note the reason — do not guess at contents.

If the doc has no distinct lessons-learned/root-cause section, use whatever causal explanation is present in the narrative and say so explicitly ("lessons learned not explicitly labeled in doc — inferred from timeline").

---

## Step 3.5 — Enrich action items with Jira data

Collect every distinct Jira ticket key found across all action items in Step 3 (from full URLs like `rula.atlassian.net/browse/PAR-2543` or bare keys like `PAR-2543`).

For each ticket key, call `getJiraIssue` with `fields: ["summary", "status", "duedate", "assignee"]`.

- If the ticket resolves: attach its **title** (`summary`), **status**, **due date**, and **assignee** to that action item (see Configuration — prefer Jira's live fields over doc text).
- If the ticket 404s, is permission-gated, or the key doesn't resolve: leave the action item as extracted from the doc and note `(Jira lookup failed)` next to it — do not guess at status, title, or due date.

---

## Step 3.6 — Compute MTTD and MTTR per incident

For each incident, compute:

- **MTTD (mean time to detect)** — time from actual onset to detection. Prefer, in order:
  1. An explicit onset time stated in the postmortem narrative (e.g. "issue began at X," a deploy/PR timestamp cited as the trigger, the first error log referenced in the timeline).
  2. The Datadog `detected` field.
  Datadog's `detected` timestamp frequently just equals `created` and does not capture delayed detection — an incident's true onset can predate its declaration by hours or days (e.g. a bad deploy that silently breaks something, only escalated to an incident once a customer/partner notices). If a monitor or synthetic fired automatically at or near onset, MTTD is ~0 — state that explicitly rather than computing a spurious nonzero delta from field rounding.

- **MTTR (mean time to resolve)** — time from detection/declaration to resolution. Prefer the postmortem's own stated resolution/mitigation timestamp or an explicit "Time to Resolution" field over Datadog's raw `resolved` field when the two diverge significantly. Datadog incidents are sometimes left open for admin/postmortem bookkeeping well after the actual fix landed (e.g. formally resolved days or weeks after the real mitigation) — don't let that bookkeeping gap masquerade as MTTR. If the postmortem timeline shows the incident reached "stable" well before it was formally marked "resolved," report **both** and label which is which: time-to-stable (active mitigation) and time-to-formal-resolution (administrative closure).

For every incident, add a short **Notes** callout flagging any data-quality caveat affecting the numbers above — onset time inferred rather than exact, the `resolved` field being unreliable for this incident, third-party confirmation lag inflating the resolution time, etc. Do not silently smooth over these; flag them so the reader knows which numbers to trust.

If neither the incident fields nor the postmortem narrative give enough information to compute MTTD (no monitor, no stated onset, detected by chance with no timestamp), say so explicitly — e.g. "Not cleanly isolable" or "Effectively unmeasurable" — rather than guessing a number.

After computing all rows, write one or two **Takeaway** sentences: do the worst MTTD/MTTR incidents correlate with a specific detection gap (e.g. all the slow-to-detect incidents lacked a matching monitor)? Ground it in the specific incidents, don't generalize past what the data shows.

---

## Step 3.7 — Pull Slack process context

For each incident, find and read its dedicated Slack channel (`incident-<public_id>`, per Configuration) to extract process context the postmortem doc doesn't capture. Postmortem docs describe *what broke*; Slack shows *how the response actually went* — declaration speed, false leads, coordination friction, and how (or whether) action items got a home.

**Locating the channel:** try `slack_search_public_and_private` with the incident's public_id and a keyword from its title first (e.g. `"887 therapyType"`) — this is more reliable than assuming the channel name resolves directly, since some incidents get triaged partly in a team channel before Datadog auto-creates the dedicated incident channel. Once found, use `slack_read_channel` (paginate with `cursor` for long-running incidents) and `slack_read_thread` on any threads with replies to get the full discussion, not just top-level messages.

If the incident spans multiple channels (e.g. informal triage in a team channel before the incident channel existed, or cross-posted to a partner channel), read all of them — the pre-incident-channel discussion is often where the most useful process signal lives (how long it took to recognize something as declaration-worthy).

**What to extract — look for, don't force all of these into every incident:**
- **Declaration/escalation speed and mechanism** — was it monitor-auto-declared, or did a human have to notice and act? How long between first signal and a formal incident channel/severity assignment? Severity assigned at declaration vs. retroactively once scope was understood is a meaningful signal on its own.
- **False leads or misdirected effort during diagnosis** — a plausible-but-wrong theory chased, a revert based on a hunch, confusion about which service/team owns the affected surface. Note how the mistake was caught and how much time it cost.
- **Coordination friction or silos** — duplicate work happening in parallel without either side aware (e.g. a support ticket and an incident channel fixing the same thing independently), a pager routed to the wrong team, an SRE/on-call pulled in without context.
- **High-stakes calls made without a visible approval trail** — a unilateral decision (e.g. flipping maintenance mode, reverting a deploy) made fast but with no manager/IC sign-off visible in the transcript. Speed isn't a problem; the missing decision trail for leadership to audit later is the signal worth flagging.
- **Postmortem-to-action-item handoff quality** — was there a clear "file tickets with owner + due date" step, or did action items drift with no home (a doc owner asking "where do we track this?" days later is a strong signal of a missing process step)? Did anyone push back on vague action items, and what happened as a result?
- **Anything that materially contradicts or adds nuance to the postmortem doc's account** — e.g. the doc says "detected via monitor" but Slack shows a human flagged it first; the doc says an action item was "Won't Fix" but no rationale appears anywhere in Slack either.

Synthesize into 2-5 sentences per incident — a tight, quotable executive paragraph, not a transcript dump. Where a specific quote or moment is the clearest evidence, cite it with its Slack permalink (`slack_read_channel`/`slack_read_thread` results include permalinks) — 1-3 permalinks per incident is usually enough. Don't restate the postmortem's root cause here; this section exists specifically to add what the doc doesn't say.

If the channel can't be found, is empty, or reading it fails, mark `Slack context: unavailable` for that incident and move on — do not guess at what might have happened.

---

## Step 3.8 — Synthesize the cross-incident process-pattern summary

Read across all the Step 3.7 findings for patterns that recur across ≥2 incidents — the same lens as Step 4.5's "Recurring themes," but for *process*, not root cause. Look specifically for things like:
- Severity being assigned reactively/retroactively rather than at declaration, across multiple incidents.
- No templated postmortem→Jira handoff — repeated instances of vague action items, no default ticketing step, or tickets closed with no rationale trail in Slack.
- Coordination silos — duplicate/parallel work, misrouted pages, ownership ambiguity playing out live in a channel rather than only being flagged after the fact.
- Fast, high-consequence decisions made with no visible approval or deliberation trail.

Only write this section if a real pattern recurs across multiple incidents — a single incident's process hiccup is a data point, not a theme, and belongs in that incident's Slack Context bullet, not here. Ground every claim in the specific incidents that demonstrate it, and close with 1-2 sentences on what this suggests for the operating model (not the code) — e.g. a templated ticket-filing step, a severity checklist at declaration, a cross-team "who else might be touching this" check before an incident-driven revert. This is placed in the Executive Summary (see Step 5 template) as a distinct paragraph, separate from the root-cause-driven bullets in Step 4.5 — don't blend the two.

If Slack context was unavailable for most/all incidents (Step 3.7 skipped or empty), omit this section rather than writing a pattern summary with no evidence behind it.

---

## Step 4 — Synthesize per-incident summary

For each incident, produce one card with:
- **Incident title**, linked to the Datadog incident, followed by the postmortem link and the Slack channel link — e.g. `## [Incident Title](datadog incident URL) ([Postmortem](postmortem URL), [Slack](slack link))`. If the postmortem is missing/unfetchable, write that in place of the link (see Step 5 template). The Slack link is always constructible (see Configuration — Slack channel links) and should always be present.
- Severity, team(s), state, declared/resolved dates, duration as a metadata line under the title.
- **Incident Description** — 2-4 sentences from Step 3 (or from Datadog's own summary field if no postmortem exists).
- **Lessons Learned from Postmortem** — 2-4 sentences from Step 3. If unavailable from any source, state `Lessons learned not documented`.
- **Action Items from Postmortem** — bulleted list, each item showing Jira link, title, status, and due date where known (from Step 3.5). Merge postmortem action items and Datadog follow-ups if both exist (dedupe by wording, don't list the same item twice). If none exist anywhere, state `No action items recorded`.
- **Slack Context** — the 2-5 sentence synthesis from Step 3.7, with permalinks, framed for engineering leadership driving incident-management process improvement (not a root-cause restatement). If Step 3.7 was skipped or the channel was unreadable, state `Slack context: unavailable` instead of omitting the bullet silently.

---

## Step 4.5 — Synthesize the cross-incident executive summary

This is a distinct synthesis step, not a mechanical roll-up of Step 4 — read across all the "Lessons Learned" you extracted in Step 3 and write for **Engineering Leadership**. The question the summary answers is: *are we detecting, responding to, and preventing incidents effectively, and where should we invest to get better?*

Ground every claim in specific incidents (cite incident titles/links) — do not generalize from a single incident into a "trend."

The cross-incident **process-pattern** paragraph from Step 3.8 (if written) is appended to the Executive Summary after **Follow-through risk** — see Step 5 template for its placement and heading. It answers a related but distinct question from the bullets below: *are we running incidents well operationally*, independent of what caused any single one.

The **Postmortem Quality, MTTD, MTTR** subsection is written here but placed in the report right after the "MTTD / MTTR by Incident" table (see Step 5), not inside Executive Summary — it's a direct analytical follow-through on that table's data:

- **Postmortem Quality** — how many postmortems have an explicit root-cause section vs. inferred; flag the best-documented incident and any incident whose raw Datadog timestamps are misleading; flag missing-Jira-ticket action items as a trackability gap.
- **MTTD** — split incidents into "monitor matched the failure mode → near-zero MTTD" vs. "monitor missing/mismatched → failed detection," state the ratio.
- **MTTR** — restore time once detected; call out inflation from third-party lag or cleanup-only tails; state plainly whether MTTR is the bottleneck.
- **Bottom line** — where the leverage is (MTTD vs. MTTR), caveated by postmortem-quality gaps, tied to 1-2 concrete actions.

The Executive Summary itself, only where the evidence actually supports it (omit any bullet with no real signal — do not pad):
- **Operational effectiveness** — what's working: fast detection, fast mitigation, good on-call response, effective use of monitors/runbooks. Cite the incidents that demonstrate it.
- **Operational inefficiencies** — recurring friction: slow detection (customer/partner reported before internal monitors fired), long time-to-mitigate, unclear ownership between teams, missing runbooks, monitors that exist but didn't cover the failure mode. Cite the incidents.
- **Systemic/recurring themes** — a root cause or contributing factor that shows up across ≥2 incidents (e.g. insufficient testing before deploy, lack of alerting on a specific class of failure, third-party API dependency fragility). Only include if it genuinely recurs — a single incident is an incident, not a theme.
- **Opportunities to improve** — concrete, prioritized recommendations that follow from the above (e.g. "add alerting on X", "assign clear ownership for Y service", "invest in integration test coverage for Z flow"). Tie each one back to the incident(s) that motivate it.
- **Follow-through risk** — action items with no owner, no due date, or stalled/overdue in Jira, especially on older incidents — this is leadership's visibility into whether lessons learned are actually being acted on.

---

## Step 5 — Write the report

Write to `/Users/skasula/source-code/markdown-files/Incident Summary - YYYY-MM-DD.md`.

```markdown
# Incident Postmortem Summary — [SINCE_DATE] → [TODAY]
Generated: [YYYY-MM-DD] | Teams: epd-partnerships, partner-growth-activation, partner-new-verticals, partner-onboarding-integrations, partner-platform, partnerships-core, partnerships-deals, martech, new-verticals

---

## Table of Contents
- [Executive Summary](#executive-summary)
- [MTTD / MTTR by Incident](#mttd--mttr-by-incident)
- [Postmortem Quality, MTTD, MTTR](#postmortem-quality-mttd-mttr)
- [Incidents](#incidents)
- [Missing / Unfetchable Postmortems](#missing--unfetchable-postmortems)
- [SEVs Summary](#sevs-summary)
- [SEV Action Items](#sev-action-items)

---

## Executive Summary

_For Engineering Leadership — lessons learned across incidents in this window, not a per-incident recap._

**Volume:** [N incidents in window — N SEV-1, N SEV-2, N SEV-3+. Postmortem coverage: N of N.]

**Operational effectiveness:** [What's working, citing specific incidents. Omit if no real signal.]

**Operational inefficiencies:** [Recurring friction — detection gaps, slow mitigation, ownership confusion — citing specific incidents. Omit if no real signal.]

**Recurring themes:** [Root causes/contributing factors shared by ≥2 incidents, cited by name. Omit if no genuine pattern.]

**Opportunities to improve:** [Concrete, prioritized recommendations tied back to the incidents that motivate them.]

**Follow-through risk:** [Action items with no owner/due date or stalled/overdue in Jira, especially on older incidents.]

**What the Slack transcripts reveal that the postmortem docs don't:** [The Step 3.8 process-pattern synthesis — recurring incident-management process gaps across ≥2 incidents (reactive severity assignment, no templated postmortem→Jira handoff, coordination silos, undocumented high-stakes decisions), each grounded in named incidents, closing with 1-2 sentences on the operating-model fix implied. Omit this entire paragraph if Step 3.7 was unavailable for most/all incidents or no real cross-incident pattern emerged — do not pad with a single incident's hiccup.]

---

## MTTD / MTTR by Incident

_Computed from Datadog incident fields (`created`/`detected`/`resolved`), cross-checked against postmortem timelines. Data-quality quirks are noted inline rather than smoothed over._

| Incident | Sev | MTTD (onset → detection) | MTTR (detection → resolution) | Notes |
|---|---|---|---|---|
| [Incident title (id)](datadog incident URL) | SEV-N | [value or "~0 (monitor)" or "Not cleanly isolable — reason"] | [value; report both time-to-stable and time-to-formal-resolution if they diverge] | [data-quality caveat, or omit if none] |
| ... | | | | |

**Takeaway:** [1-2 sentences connecting the worst MTTD/MTTR rows to a specific detection/response gap, grounded in the cited incidents — not a generic statement.]

---

## Postmortem Quality, MTTD, MTTR

**Postmortem Quality:** [How many postmortems have an explicit root-cause/contributing-factors section vs. inferred from narrative. Call out any doc that self-reports MTTD/MTTR explicitly (best-practice example). Call out any Datadog record whose raw fields are misleading (e.g. left open for bookkeeping) — don't let a dashboard built on raw fields inherit that error. Note where action items lack a Jira ticket ("documented" ≠ "trackable").]

**MTTD:** [Which incidents hit near-zero MTTD via a matched monitor vs. which failed and why (monitor covered wrong signal, no monitor existed, unmeasurable due to no monitor). State the ratio, e.g. "N of M already at elite MTTD; the rest are the entire problem."]

**MTTR:** [Once detected, how fast was restore for each — call out any MTTR inflated by third-party confirmation lag, multi-stage fixes, or cleanup-only tails rather than idle response. State plainly whether MTTR is or isn't the bottleneck in this window.]

**Bottom line:** [Where the real leverage is between MTTD and MTTR investment, and any caveat on trusting the numbers (e.g. weak-postmortem-quality incidents shouldn't anchor a baseline). Tie to one or two concrete next actions.]

---

## Incidents

## [Incident Title](datadog incident URL) ([Postmortem](postmortem URL) / Postmortem: Missing / Postmortem: Unfetchable ([reason]), [Slack](slack link))
**Severity:** SEV-N · **Team:** [team] · **State:** [state] · **Declared:** [date] · **Resolved:** [date or "Ongoing"] · **Duration:** [N hrs/days]

**Incident Description:** [2-4 sentences]

**Lessons Learned from Postmortem:** [2-4 sentences, or "Lessons learned not documented"]

**Action Items from Postmortem:**
- [ ] [Jira Title](Jira link) — [status], due [date or "no due date"]
- [ ] [item without a Jira ticket] — [status/owner if known from doc]
- ...
_or:_ No action items recorded.

**Slack Context (process, for engineering leadership):** [2-5 sentence synthesis from Step 3.7 — declaration/escalation speed, false leads, coordination friction, undocumented high-stakes calls, or postmortem-to-action-item handoff quality, with 1-3 Slack permalinks as evidence. Do not restate the postmortem's root cause here. _or:_ Slack context: unavailable.]

---

[Repeat per incident, newest first]

---

## Missing / Unfetchable Postmortems

| Incident | Team | Severity | State | Reason |
|----------|------|----------|-------|--------|
| [title](link) | ... | ... | ... | No postmortem attached / Doc returned 404 / Doc requires login / Confluence not supported |

_If none: "All incidents in this window have a readable postmortem."_

---

## SEVs Summary
List all SEVs for which your team was responsible.

| Link | Severity | Description | Status |
|---|---|---|---|
| [id](datadog incident URL) | SEV-N | [incident title] | [state] |
| ... | | | |

_Order rows by declared date, newest first — same order as the Incidents section above._

## SEV Action Items
List all action Items from the SEVs above. Copy all unfinished Action Items from previous review. Do not update their statuses in previous reviews.

| Link | Description | Incident | ✅ Done? | Due Date |
|---|---|---|---|---|
| [Jira link or "—"] | [action item text] | [id](datadog incident URL) | ✅ / ☐ | [date or "—"] |
| ... | | | | |

_Order rows by the parent incident's declared date, newest first. Skip incidents with no action items recorded._
```

---

## Step 6 — Post-write: print summary to chat

After writing the file, print the **Executive Summary** and **Missing / Unfetchable Postmortems** sections directly in the chat.

Then output:
> Full incident postmortem summary written to `/Users/skasula/source-code/markdown-files/Incident Summary - YYYY-MM-DD.md`

---

## Error handling

- **No incidents in window:** report "No incidents found for these teams in [window]." — do not fabricate.
- **Incident has no postmortem attachment:** mark `Missing`, list it in the Missing/Unfetchable table, still include the card using Datadog's own fields/follow-ups if any exist.
- **Postmortem link is a Confluence URL:** mark `Unfetchable (Confluence not supported)` — do not attempt to fetch it.
- **Postmortem link 404s or is permission-gated:** mark `Unfetchable (reason)`, do not guess contents.
- **Postmortem doc has no distinct lessons-learned/root-cause section:** infer from the narrative and say so explicitly rather than leaving it blank.
- **Duplicate action items between the postmortem doc and Datadog follow-ups:** dedupe by wording, don't double-list.
- **Jira ticket key doesn't resolve (404 / no access):** keep the action item as extracted from the doc, append `(Jira lookup failed)`, don't guess at status, title, or due date.
- **Executive summary bullet has no real supporting evidence:** omit that bullet entirely rather than padding it with a generic statement.
- **Slack MCP tools unavailable, unauthenticated, or the incident's channel can't be located/read:** mark `Slack context: unavailable` for that incident's card, skip it in Step 3.8's pattern synthesis, and continue with the rest of the report — never block the whole run on Slack access.
- **Step 3.8 process-pattern paragraph has no real cross-incident evidence:** omit the paragraph from the Executive Summary entirely rather than writing one from a single incident's hiccup.

---

## Invocation

The skill is invoked via `/incident-postmortem-summary`. Optional arguments:
- `--days=N` — override the window length (defaults to 90 days)
- `--since=YYYY-MM-DD` — override the window start explicitly (window end is today)
- `--all` — remove the time filter, pull every incident regardless of age

Examples:
```
/incident-postmortem-summary
/incident-postmortem-summary --days=30
/incident-postmortem-summary --all
```
