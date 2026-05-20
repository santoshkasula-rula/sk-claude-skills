---
name: estimate-effort
description: Guides engineers through a structured design and estimation process — synthesizes requirements, proposes solution candidates, maps repo impact per solution, surfaces RAD, compares tradeoffs, recommends an approach, then writes a PM-readable implementation plan.
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

1. **PRD** — "Let's dig in. Drop the path to your PRD or paste the contents directly — whichever is easier." — required, re-ask if unreadable. If a path is given, read the file. If contents are pasted, use them directly.
2. **Transcript** — "PRDs are the polished story. I want the unpolished one too — the part where someone said 'I'm not sure how long that'll take' or 'we've never touched that service.' Got a Zoom transcript, meeting notes, Slack thread, or any other discussion doc? (path, paste, or skip — but the messier the better)" — if provided, normalize it: strip timestamps, speaker labels, `[inaudible]`, and merge fragmented turns into prose.
3. **Additional context** — "Anything else that would embarrass us if we ignored it? Architecture docs, API specs, prior plans, that one Confluence page nobody's updated since 2022? (path, paste, or done)" — repeat until 'done'.

---

## Step 2 — Repo discovery (only if `--root`)

```bash
find . -maxdepth 3 \( -name "package.json" -o -name "go.mod" -o -name "requirements.txt" -o -name "Cargo.toml" -o -name "pom.xml" \) ! -path "*/node_modules/*" ! -path "*/.git/*"
```

Identify each service and any cross-repo shared dependencies.

---

## Step 2b — Dependency discovery (always)

Detect whether running inside a real repo:

```bash
git rev-parse --show-toplevel 2>/dev/null && basename $(git rev-parse --show-toplevel)
```

If inside a repo, read the relevant manifest(s) — use only what exists:
- `package.json` → `dependencies` + `devDependencies` + `workspaces`
- `go.mod` → `require` block
- `docker-compose.yml` / `docker-compose.yaml` → `services` keys
- `requirements.txt` or `pyproject.toml` → top-level package names

Extract internal services only (signals: org-scoped packages, docker-compose services, local `replace` directives). Present and confirm in one message:

> **Alright, I poked around and found these internal services. Which ones are coming along for the ride?**
>
> - `[service-name]` — [inferred purpose]
> - *(nothing internal detected — I'll ask you directly later)*

Wait for response. Record confirmed services — they seed the repo impact analysis and RAD.
If no git root found: skip silently.

---

## Step 3 — Requirements synthesis

**From all documents**, extract and consolidate. Use exact terminology from the source materials.

Extract:
- **Use cases** — who does what, and why (actor + action + goal)
- **User stories** — if present verbatim; otherwise infer from PRD features and transcript context
- **Acceptance criteria** — explicit "done" conditions stated in any document
- **Success metrics** — measurable outcomes the PRD ties to the feature
- **Scope** — features explicitly in scope; features explicitly out of scope
- **Constraints** — non-negotiables (compliance, performance targets, existing contracts)

**Conflict rule:** Transcript anxiety markers override PRD optimism on the same component — always.

Present a consolidated requirements summary and ask one targeted question if anything critical is missing or ambiguous:

> **Here's what I think we're actually building — fight me on anything that's wrong:**
>
> **Use cases:**
> - [Actor] needs to [action] so that [goal]
> - ...
>
> **Acceptance criteria:** [list, or "not stated — what does shipped actually mean here?"]
> **Success metrics:** [list, or "not stated — how will anyone know this worked?"]
> **Constraints:** [list, or "none identified — lucky you"]
> **Out of scope:** [list, or "not explicit — what are we quietly agreeing NOT to build?"]
>
> [One question if a critical use case or constraint seems missing — otherwise skip]

Wait for response. Lock the requirements before proceeding.

---

## Step 4 — Solution options

Based on the confirmed requirements and any codebase context from Steps 2/2b, propose **2–3 candidate solutions**. Each solution is a distinct functional design — a meaningfully different architectural or implementation approach, not just a scope variation.

For each solution:
- Give it a short name (e.g. "Extend existing service", "New dedicated service", "Client-side with API contract")
- One sentence: what it does and how it addresses the requirements
- Key repo/service areas it would touch (high-level, before deep analysis)
- One-line effort hint: rough complexity signal (e.g. "lower risk, more scope", "faster to build, harder to maintain")

> **Here are the ways we could tackle this — I have opinions, but let's see what resonates:**
>
> **Option A — [Name]**
> [What it is and how it solves the problem]
> Touches: [service/repo list]
> Vibe: [effort/risk hint — e.g. "lower risk but more moving parts", "fast to ship, future-you will have thoughts"]
>
> **Option B — [Name]**
> [What it is and how it solves the problem]
> Touches: [service/repo list]
> Vibe: [effort/risk hint]
>
> **Option C — [Name]** *(if warranted)*
> ...
>
> Which direction are you leaning? If you've already got something in mind that I missed, throw it in.

Wait for response. Record the confirmed solution set — proceed only with solutions the engineer validates. If they propose their own approach, add it and drop any that are clearly off the table.

---

## Step 4b — Repo impact per solution

For each confirmed solution, map the specific changes needed across repos and services. If inside a codebase, scan relevant areas using `find`, `ls`, and `Read`. Otherwise, ask the engineer directly.

For each solution, produce:

> **[Solution Name] — what actually has to change:**
> - `[repo/service]`: [what changes — e.g. "new API endpoint", "schema migration", "config update"]
> - `[repo/service]`: [what changes]
> - Blast radius: [narrow / moderate / broad]

If a solution touches a repo or service that wasn't in the confirmed list from Step 2b, flag it:
> *"Heads up — [Solution X] would drag [service] into this too. Is that in scope, or do we need to rethink?"*

Record all confirmed repo impacts — they feed the RAD and the Technical Breakdown.

---

## Step 5 — RAD thought-partner

With requirements locked and solutions mapped, surface risks, assumptions, and dependencies. Some items will apply to all solutions; some are solution-specific — label them.

Present a pre-filled RAD draft and 2–3 targeted questions in one message:

> **Here's the RAD — the stuff that keeps estimates honest. Fix anything that's wrong:**
>
> **Risks:** [list with source and which solution(s) it affects]
> **Assumptions:** [list with source — aka "things we're betting on being true"]
> **Dependencies:** [list — note if a dependency is only triggered by a specific solution]
> **Success criteria:** [from Step 3, or "still fuzzy — how will you actually know this is done?"]

Then ask 2–3 targeted questions — pick the ones that would genuinely change the estimate:

- If a solution touches legacy or unfamiliar code: *"What's the part of [Solution X] most likely to blow up the timeline — and has anyone on the team actually been in that code recently?"*
- If assumptions are thin: *"What has to quietly be true for any of these estimates to hold? What's the thing nobody's said out loud yet?"*
- If external coordination is needed: *"Who outside your team needs to say yes or do work for each option — and have you talked to them?"*
- If scope boundaries are unclear: *"What's the thing someone will definitely assume is in scope, that we haven't agreed to build?"*
- If a solution is new ground: *"Has your team shipped something like [Solution X] before, or is this 'we'll figure it out as we go' territory?"*

**Rules:**
- Never ask more than 3 questions.
- Never re-ask what documents already answered.
- If the engineer raises a new risk, follow up once: *"What's the move to make that less scary?"* then move on.

Wait for response. Update RAD before continuing.

---

## Step 6 — Tradeoff analysis

Compare all confirmed solutions across the dimensions that matter most given the RAD. Present as a structured comparison — not a generic pros/cons list, but grounded in the specific requirements, risks, and repo impacts identified.

> **The showdown — let's see how these options actually stack up:**
>
> | | [Solution A] | [Solution B] | [Solution C] |
> | :--- | :--- | :--- | :--- |
> | **Effort** | [Low/Med/High — brief reason] | ... | ... |
> | **Risk exposure** | [RAD items it inherits] | ... | ... |
> | **Blast radius** | [breadth of change] | ... | ... |
> | **Covers all requirements** | Yes / Partially — [gap] | ... | ... |
> | **Future-you will thank you?** | [maintainability signal] | ... | ... |
> | **The thing you give up** | [key trade-off] | ... | ... |

Follow the table with 2–3 sentences of honest narrative — what the table can't capture (team familiarity, how reversible this is, sequencing landmines).

---

## Step 7 — Recommendation

Based on the requirements, RAD, and tradeoff analysis, propose one solution as the recommended approach. Be direct — don't hedge if the data points clearly to one option.

> **My call: [Solution Name]**
>
> [2–3 sentences: why this option wins given what we know — requirements, constraints, risks, and the team that actually has to build it. Name the trade-off being accepted, without apologizing for it.]
>
> *The honest risk: [one sentence about what could still bite us.]*
>
> Does this land, or is there something pulling you toward a different option?

If the engineer pushes back, ask: *"What's the pull toward [other option] — is there a constraint we haven't put on the table?"* Record their reasoning, accept their call, and move on without relitigating it.

---

## Step 8 — Design milestones

Create 3–5 milestones for the **chosen solution**. Use the language from the documents — names, deliverables, and business value must reflect the actual words and goals from the PRD, transcript, and requirements synthesis. No generic placeholders.

Each milestone must:
- Deliver a concrete, demonstrable artifact
- Be sequenced so each unblocks the next
- Map to a specific business value or acceptance criterion from Step 3
- Break effort into three buckets (never combine):
  - **Engineering** — coding, design, code review
  - **Operational** — deployment, infra, config, secrets
  - **Rollout** — QA, testing, feature flag ramp, stakeholder demo
- Carry a risk flag: 🔴 anxiety-marker or RAD risk · 🟡 uncertain · 🟢 well-understood

**Buffer:** 20% added to Engineering total only. Show raw + buffered separately. Never embed inside milestones.

If `--root`: assign tasks to repos, sequence to respect cross-repo dependencies.

---

## Step 9 — Write output file

Write to `Effort Breakdown Analysis - [Project Name].md` in the current directory, using the actual project name from the documents. Follow `OUTPUT_TEMPLATE.md`. Key rules:

- **Executive Summary** — plain language, no repo names: goal, chosen solution (one sentence), effort totals (raw + buffered), success criteria, risk summary (one sentence if any 🔴 milestone), key assumptions, hard dependencies.
- **Requirements** — use cases, acceptance criteria, success metrics, constraints, in/out of scope.
- **Solution Options** — one section per candidate with description, repo impact, and key trade-off. Mark the chosen one.
- **RAD** — every item cites source (PRD, Transcript, or Engineer) and which solution(s) it affects.
- **Tradeoff Matrix** — the comparison table from Step 6.
- **Recommendation** — chosen solution, rationale, main risk accepted.
- **Milestones** — deliverable, business value, Engineering / Operational / Rollout effort, risk flag.
- **Technical Breakdown** — one section per repo for the chosen solution, specific files/areas, one-line rationale each.
- **Cross-repo map** — only if `--root`.
- **Engineer Updates** — empty table, leave for the team.

End with:
> **Share with PMs / leadership:** Executive Summary, Requirements, Recommendation, Milestones
> **Share with engineers:** full document
