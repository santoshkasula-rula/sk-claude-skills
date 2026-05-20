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

1. **PRD** — "Path to your PRD, or paste the contents directly?" — required, re-ask if unreadable. If a path is given, read the file. If contents are pasted, use them directly.
2. **Transcript** — "To think more deeply about risks, unstated assumptions, and design intent, it helps to have richer context beyond the PRD. Do you have any Zoom transcripts, meeting notes, Slack threads, or other discussion docs I can use? (path, paste, or skip)" — if provided, normalize it: strip timestamps, speaker labels, `[inaudible]`, and merge fragmented turns into prose.
3. **Additional context** — "Any other files? (architecture docs, API specs, prior plans — path or done)" — repeat until 'done'.

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

> **I found these internal services — which ones are in scope for this project?**
>
> - `[service-name]` — [inferred purpose]
> - *(none detected — I'll ask later)*

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

> **Here's my read of the requirements — correct anything before we look at solutions:**
>
> **Use cases:**
> - [Actor] needs to [action] so that [goal]
> - ...
>
> **Acceptance criteria:** [list, or "not stated — what does 'done' look like?"]
> **Success metrics:** [list, or "not stated — how will you measure success?"]
> **Constraints:** [list, or "none identified"]
> **Out of scope:** [list, or "not explicit — what should be excluded?"]
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

> **Here are the candidate approaches — do any match what you're thinking, or do you have a different direction?**
>
> **Solution A — [Name]**
> [What it is and how it solves the problem]
> Touches: [service/repo list]
> Signal: [effort/risk hint]
>
> **Solution B — [Name]**
> [What it is and how it solves the problem]
> Touches: [service/repo list]
> Signal: [effort/risk hint]
>
> **Solution C — [Name]** *(if warranted)*
> ...
>
> Which direction are you leaning? Add, remove, or correct any of these.

Wait for response. Record the confirmed solution set — proceed only with solutions the engineer validates. If they propose their own approach, add it and drop any that are clearly off the table.

---

## Step 4b — Repo impact per solution

For each confirmed solution, map the specific changes needed across repos and services. If inside a codebase, scan relevant areas using `find`, `ls`, and `Read`. Otherwise, ask the engineer directly.

For each solution, produce:

> **[Solution Name] — repo impact:**
> - `[repo/service]`: [what changes — e.g. "new API endpoint", "schema migration", "config update"]
> - `[repo/service]`: [what changes]
> - Estimated touch points: [narrow / moderate / broad]

If a solution touches a repo or service that wasn't in the confirmed list from Step 2b, flag it:
> *"[Solution X] would also require changes in [service] — is that in scope?"*

Record all confirmed repo impacts — they feed the RAD and the Technical Breakdown.

---

## Step 5 — RAD thought-partner

With requirements locked and solutions mapped, surface risks, assumptions, and dependencies. Some items will apply to all solutions; some are solution-specific — label them.

Present a pre-filled RAD draft and 2–3 targeted questions in one message:

> **Here's what I found — correct anything:**
>
> **Risks:** [list with source and which solution(s) it affects]
> **Assumptions:** [list with source]
> **Dependencies:** [list — note if a dependency is only triggered by a specific solution]
> **Success criteria:** [from Step 3, or "not confirmed — how will you know this is done?"]

Then ask 2–3 targeted questions — pick based on genuine gaps:

- If a solution touches legacy or unfamiliar code: *"What's the trickiest part of [Solution X] — what would make it take twice as long?"*
- If assumptions are thin: *"What has to be true for any of these estimates to hold?"*
- If external coordination is needed: *"Who outside your team needs to cooperate for each solution, and do they know?"*
- If scope boundaries are unclear: *"What's adjacent to this that someone might assume is included?"*
- If a solution is new ground: *"Has your team built something like [Solution X] before?"*

**Rules:**
- Never ask more than 3 questions.
- Never re-ask what documents already answered.
- If the engineer raises a new risk, follow up with one mitigation probe: *"What's the concrete step to reduce that risk?"* then move on.

Wait for response. Update RAD before continuing.

---

## Step 6 — Tradeoff analysis

Compare all confirmed solutions across the dimensions that matter most given the RAD. Present as a structured comparison — not a generic pros/cons list, but grounded in the specific requirements, risks, and repo impacts identified.

> **Solution comparison:**
>
> | | [Solution A] | [Solution B] | [Solution C] |
> | :--- | :--- | :--- | :--- |
> | **Effort** | [Low/Med/High — brief reason] | ... | ... |
> | **Risk exposure** | [RAD items it inherits] | ... | ... |
> | **Repo impact** | [breadth of change] | ... | ... |
> | **Meets all requirements** | Yes / Partially — [gap] | ... | ... |
> | **Long-term maintainability** | [signal] | ... | ... |
> | **Key trade-off** | [the one thing you give up] | ... | ... |

Follow the table with 2–3 sentences of narrative — what the table can't capture (team familiarity, reversibility, sequencing implications).

---

## Step 7 — Recommendation

Based on the requirements, RAD, and tradeoff analysis, propose one solution as the recommended approach. Be direct — don't hedge if the data points clearly to one option.

> **Recommendation: [Solution Name]**
>
> [2–3 sentences: why this solution best addresses the requirements given the constraints, risks, and team context. Call out the main thing being accepted as a trade-off.]
>
> *The main risk with this recommendation: [one honest sentence about what could still go wrong.]*
>
> Does this match your thinking, or do you want to go a different direction?

If the engineer disagrees, ask: *"What's driving the preference for [other solution] — is there a constraint we haven't surfaced?"* Record their reasoning. Accept their choice and proceed.

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
