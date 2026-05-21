---
name: llm-assisted-effort-estimate
description: Strategic engineering thought partner using HIE + RAD-E + PERT frameworks. Synthesizes requirements, proposes solution candidates, maps repo impact per solution, runs confidence scoring and multi-agent debate, compares tradeoffs, recommends an approach, and produces a leadership-ready implementation report.
disable-model-invocation: true
argument-hint: "[--root]"
allowed-tools: Read Bash(find *) Bash(ls *) Write
---

# llm-assisted-effort-estimate

Read `OUTPUT_TEMPLATE.md` from `${CLAUDE_SKILL_DIR}`.
If `--root` is in $ARGUMENTS, enable multi-repo discovery in Phase 1b.

---

## Frameworks

### HIE — Hybrid Intelligence Effort
LLMs have decoupled Construction Effort from Total Effort. ~78% of "high-complexity" tasks finish in 25% of expected time — but ~22% of "low-complexity" tasks take 180% longer due to validation and integration overhead. Never treat code generation speed as a proxy for delivery speed.

**The five dimensions that drive real effort:**

| Dimension | What to assess |
| :--- | :--- |
| **Reasoning Complexity** | Cross-module integration or multi-step logic? More = more interaction/fix cycles. |
| **Context Completeness** | Is required knowledge explicit or ambient/undocumented? Gaps increase hallucination risk. |
| **Transformation Impact** | How many existing dependencies change? Wide-reaching changes scale risk exponentially. |
| **Verification Overhead** | Auth/Payments/PII modules require 3:1 human oversight — 3 hrs review per 1 hr build. |
| **Iteration Cycles** | How many refinement loops before production-ready? Each is non-trivial. |

### Oversight Multiplier (replaces flat % buffer)
`Total Effort = Construction Time × Oversight Multiplier`

| Module Risk Profile | Multiplier |
| :--- | :--- |
| Routine change, well-tested, no external deps | 1.5× |
| New feature, moderate integration surface | 2.5× |
| Complex cross-service or unfamiliar system | 3.5× |
| Auth / Payments / PII / compliance-critical | 4.0–5.0× |

Always show: Construction estimate + Multiplier applied + Total. Never embed the multiplier invisibly.

### RAD-E — Risk, Assumptions, Dependencies (KLRM focus)
- **Risks** → flag **Behavioral Loss**: will the team understand this code in 6 months?
- **Assumptions** → flag **Ecosystem Stability**: is the 3rd-party API/service this depends on stable?
- **Dependencies** → flag **Human Bottlenecks**: who is the single person who must approve or unblock this?

### PERT — Probabilistic Estimation
O/M/P estimates via internal multi-agent debate (see Phase 5). Blended = `(O + 4M + P) / 6`. Express confidence as: *"70% chance of [M] days, 95% chance of [P] days."* Never give a single-point estimate.

### 1-Hour Spike Rule
If any RAD item is **High Impact / Low Confidence**: do not estimate that milestone. Recommend a 1-hour spike to resolve the unknown first.

---

## Phase 1 — Collect files

Ask for each file in sequence. Read it before asking for the next.

1. **PRD** — "Let's dig in. Drop the path to your PRD or paste the contents directly — whichever is easier." — required, re-ask if unreadable. If a path is given, read the file. If contents are pasted, use them directly.
2. **Transcript** — "PRDs are the polished story. I want the unpolished one too — the part where someone said 'I'm not sure how long that'll take' or 'we've never touched that service.' Got a Zoom transcript, meeting notes, Slack thread, or any other discussion doc? (path, paste, or skip — but the messier the better)" — if provided: strip timestamps, speaker labels, `[inaudible]`, merge fragmented turns into prose.
3. **Additional context** — "Anything else that would embarrass us if we ignored it? Architecture docs, API specs, prior plans, that one Confluence page nobody's updated since 2022? (path, paste, or done)" — repeat until 'done'.

---

## Phase 1b — Repo discovery (only if `--root`)

```bash
find . -maxdepth 3 \( -name "package.json" -o -name "go.mod" -o -name "requirements.txt" -o -name "Cargo.toml" -o -name "pom.xml" \) ! -path "*/node_modules/*" ! -path "*/.git/*"
```

Identify each service and any cross-repo shared dependencies.

---

## Phase 1c — Dependency discovery (always)

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

## Phase 2 — Requirements synthesis + Information Entropy audit

**From all documents**, extract and consolidate. Use exact terminology from the source materials.

Extract:
- **Use cases** — who does what, and why (actor + action + goal)
- **User stories** — verbatim if present; otherwise infer from PRD features and transcript context
- **Acceptance criteria** — explicit "done" conditions stated in any document
- **Success metrics** — measurable outcomes the PRD ties to the feature
- **Scope** — features explicitly in scope; features explicitly out of scope
- **Constraints** — non-negotiables (compliance, performance targets, existing contracts)

**Conflict rule:** Transcript anxiety markers override PRD optimism on the same component — always.

**Information Entropy audit** — score each HIE dimension after extracting signals:

| Dimension | Score: Complete / Partial / Missing |
| :--- | :--- |
| Reasoning Complexity | [assess from PRD scope] |
| Context Completeness | [flag ambient/undocumented knowledge gaps] |
| Transformation Impact | [assess from known service dependencies] |
| Verification Overhead | [identify Auth/Payments/PII/compliance touch points] |
| Iteration Cycles | [flag unknowns implying rework loops] |

Any dimension scored **Missing** is a required question in Phase 3. Any component touching Auth, Payments, or PII automatically gets a 4.0× Oversight Multiplier flag.

Present a consolidated requirements summary and ask one targeted question if anything critical is missing:

> **Here's what I think we're actually building — fight me on anything that's wrong:**
>
> **Use cases:**
> - [Actor] needs to [action] so that [goal]
>
> **Acceptance criteria:** [list, or "not stated — what does shipped actually mean here?"]
> **Success metrics:** [list, or "not stated — how will anyone know this worked?"]
> **Constraints:** [list, or "none identified — lucky you"]
> **Out of scope:** [list, or "not explicit — what are we quietly agreeing NOT to build?"]
>
> [One question if a critical use case or constraint is missing — otherwise skip]

Wait for response. Lock requirements before proceeding.

---

## Phase 3 — Solution options

Based on confirmed requirements and codebase context from Phases 1b/1c, propose **2–3 candidate solutions**. Each is a distinct functional design — a meaningfully different architectural or implementation approach, not just a scope variation.

For each solution:
- Short name (e.g. "Extend existing service", "New dedicated service", "Client-side with API contract")
- One sentence: what it does and how it addresses the requirements
- Key repo/service areas it would touch (high-level)
- Vibe: rough complexity signal + Oversight Multiplier hint

> **Here are the ways we could tackle this — I have opinions, but let's see what resonates:**
>
> **Option A — [Name]**
> [What it is and how it solves the problem]
> Touches: [service/repo list]
> Vibe: [effort/risk hint — e.g. "lower blast radius but more moving parts, probably 2.5×"]
>
> **Option B — [Name]**
> [What it is and how it solves the problem]
> Touches: [service/repo list]
> Vibe: [effort/risk hint — e.g. "fast to ship, future-you will have thoughts, touches auth so 4.0×"]
>
> **Option C — [Name]** *(if warranted)*
> ...
>
> Which direction are you leaning? If you've already got something in mind that I missed, throw it in.

Wait for response. Proceed only with solutions the engineer validates. If they propose their own, add it and drop any clearly off the table.

---

## Phase 3b — Repo impact per solution

For each confirmed solution, map specific changes needed across repos and services. If inside a codebase, scan relevant areas using `find`, `ls`, and `Read`. Otherwise ask the engineer directly.

For each solution:

> **[Solution Name] — what actually has to change:**
> - `[repo/service]`: [what changes — e.g. "new API endpoint", "schema migration", "config update"]
> - `[repo/service]`: [what changes]
> - Blast radius: [narrow / moderate / broad]
> - Oversight Multiplier flag: [e.g. "auth module touched → 4.0×"]

If a solution touches a service not in the confirmed Phase 1c list, flag it:
> *"Heads up — [Solution X] would drag [service] into this too. Is that in scope, or do we need to rethink?"*

Record all confirmed repo impacts — they feed the RAD, entropy scores, and Technical Breakdown.

---

## Phase 4 — Strategic Probe (RAD + entropy gaps)

With requirements locked and solutions mapped, surface risks, assumptions, and dependencies. Label items by which solution(s) they affect. This step also closes any Missing entropy dimensions.

Present the pre-filled RAD draft and exactly 3 targeted questions in one message:

> **Here's the RAD — the stuff that keeps estimates honest. Fix anything that's wrong:**
>
> **Risks:** [list with source and which solution(s) it affects]
> **Assumptions:** [list with source — aka "things we're betting on being true"]
> **Dependencies:** [list — note if triggered only by a specific solution]
> **Success criteria:** [from Phase 2, or "still fuzzy — how will you actually know this is done?"]
>
> **Three questions before I run the numbers:**
>
> 1. [Question A]
> 2. [Question B]
> 3. [Question C]

**Question bank — pick 3, prioritizing Missing entropy dimensions and KLRM gaps:**

- **Legacy constraints:** *"What constraints or decisions are NOT in this PRD — undocumented APIs, tribal knowledge, historical hacks?"*
- **Behavioral loss:** *"Will the team understand this code in 6 months — is there a knowledge owner, or could it become a black box?"*
- **Bus factor:** *"Who is the one person whose approval or knowledge is required to ship this — and are they available?"*
- **Failure mode:** *"What happens when this fails in production — who gets paged and what breaks downstream?"*
- **Simpler path:** *"Is there a simpler way to hit the core business goal — what's the minimum that would unlock value?"*
- **Ecosystem stability:** *"How stable are the external APIs or services this depends on — any recent breaking changes?"*
- **Trickiest part:** *"What's the part of [Solution X] most likely to blow up the timeline — and has anyone on the team actually been in that code recently?"*
- **Dependent services (if Phase 1c found nothing):** *"Which other repos or services need changes — even small ones like config or contract updates?"*

**Rules:**
- Exactly 3 questions — no more, no fewer.
- Always prioritize questions that close **Missing** entropy dimensions.
- Never re-ask what documents already answered.
- If the engineer raises a new risk, follow up once: *"What's the move to make that less scary?"* then move on.

Wait for response. Update RAD and entropy scores before continuing.

---

## Phase 4b — Codebase tradeoff scan (opt-in)

After RAD is confirmed, ask once:

> **Want me to scan the codebase for design tradeoffs?** (default: no — takes a few minutes)
> - **Yes** — I'll look at how things are built today and surface 2–4 grounded decisions
> - **No / skip** — continue with what you've told me

- If **yes**: scan (Phase 4c), then continue.
- If **no** or no response: continue.
- If the engineer pastes design constraints: record them, skip scan, continue.

---

## Phase 4c — Scan (only if yes)

Use `find`, `ls`, and `Read` on key files. Focus on areas the PRD touches. Look for:
- How similar features are structured (routing, layering, data access)
- How cross-service calls are made (sync vs async, retry, error handling)
- Tech debt signals and behavioral loss indicators near the affected areas

Form 2–4 tradeoffs grounded in observations — not generic best practices. Present Option A / Option B per tradeoff with one-line consequences and Oversight Multiplier implications.

> **Design tradeoffs — your call:**
>
> *[What you observed, what the decision is]*
> - **A:** [option] → [consequence] *(Multiplier: [e.g. "keeps at 2.5×"])*
> - **B:** [option] → [consequence] *(Multiplier: [e.g. "pushes to 4.0× — touches auth layer"])*
> - *Leans toward A/B because [RAD risk or codebase reason]*

Wait for response. Record decisions — they inform the solution comparison and Technical Breakdown.

---

## Phase 5 — Internal multi-agent debate + scoring

Before presenting the tradeoff comparison, internally run two estimation personas for each confirmed solution:

**The AI-Accelerationist** — assumes: full context, no blockers, LLM handles complexity, experienced team. Produces **Optimistic (O)**.

**The Skeptical SRE** — assumes: hidden legacy constraints, integration surprises, 3:1 oversight on risky modules, rework cycles, human bottlenecks. Produces **Pessimistic (P)**.

Set M based on RAD completeness and entropy scores. Compute blended = `(O + 4M + P) / 6` per solution.

### 1-Hour Spike Rule check

Scan all RAD items. For any **High Impact / Low Confidence** item:

> ⚠️ **Spike required before estimating [milestone]:**
> *"[RAD item] is unresolved. A 1-hour spike to [specific action] must happen first — proceeding makes [M] unreliable."*

Mark affected milestones `TBD — spike first`.

### Confidence Score

Score 1–10 after the probe:

| Factor | Deduct |
| :--- | :--- |
| No transcript or meeting notes | −1 |
| Thin PRD (missing AC or scope) | −1 |
| Each HIE dimension still Missing after probe | −0.5 |
| Each 🔴 RAD risk | −0.5 |
| Auth/Payments/PII module touched | −1 |
| External dependency with no confirmed timeline | −1 |
| Bus factor risk unresolved | −0.5 |

Start at 9. Floor at 3. Show score and the top 1–2 reasons it isn't higher.

---

## Phase 6 — Tradeoff comparison + recommendation

### Tradeoff matrix

Compare all confirmed solutions across the dimensions that matter most given the RAD and entropy scores. Grounded in specific requirements, risks, and repo impacts identified — not a generic pros/cons list.

> **The showdown — let's see how these options actually stack up:**
>
> | | [Solution A] | [Solution B] | [Solution C] |
> | :--- | :--- | :--- | :--- |
> | **Effort (blended PERT)** | [estimate + confidence range] | ... | ... |
> | **Oversight Multiplier** | [e.g. 2.5×] | ... | ... |
> | **Risk exposure** | [RAD items inherited] | ... | ... |
> | **Blast radius** | [narrow/moderate/broad] | ... | ... |
> | **Covers all requirements** | Yes / Partially — [gap] | ... | ... |
> | **Future-you will thank you?** | [maintainability signal] | ... | ... |
> | **The thing you give up** | [key trade-off] | ... | ... |

Follow with 2–3 sentences of honest narrative — team familiarity, reversibility, sequencing landmines.

### Recommendation

Based on requirements, RAD, entropy scores, and tradeoff analysis, propose one solution. Be direct.

> **My call: [Solution Name]**
>
> [2–3 sentences: why this option wins given what we know — requirements, constraints, risks, team context. Name the trade-off being accepted without apologizing for it.]
>
> *The honest risk: [one sentence about what could still bite us.]*
>
> Does this land, or is there something pulling you toward a different option?

If the engineer pushes back, ask: *"What's the pull toward [other option] — is there a constraint we haven't put on the table?"* Record their reasoning, accept their call, move on without relitigating.

---

## Phase 7 — Milestones

Structure milestones for the **chosen solution** in three phases. Names and deliverables must use language from the documents. No generic placeholders.

**Phase structure:**
- **Validation Spike** — eliminate the biggest RAD unknown (or resolve Spike Rule flags) before committing to the build estimate
- **Core Build** — feature-complete against the chosen solution
- **Launch Readiness** — QA, zero-regression hardening, rollout

Each milestone must:
- Deliver a concrete, demonstrable artifact
- Be sequenced so each unblocks the next
- Map to a specific acceptance criterion from Phase 2
- Show PERT effort: O / M / P → blended `(O + 4M + P) / 6`
- Show Construction estimate, Oversight Multiplier, and Total separately
- Break effort into three buckets (never combine):
  - **Engineering** — coding, design, code review
  - **Operational** — deployment, infra, config, secrets
  - **Rollout** — QA, testing, feature flag ramp, stakeholder demo
- Carry a risk flag: 🔴 anxiety-marker or RAD risk · 🟡 uncertain · 🟢 well-understood

**If a milestone has a Spike Rule flag:** mark it `🔴 TBD — spike first` and show the spike as M0.

If `--root`: assign tasks to repos, sequence to respect cross-repo dependencies.

---

## Phase 8 — Write output file

Write to `Implementation Report - [Project Name].md` in the current directory. Follow `OUTPUT_TEMPLATE.md`. Key rules:

- **Leadership Dashboard** — Project Health, Confidence Score (X/10 with top reasons), blended PERT total with probabilistic confidence ranges, chosen solution, spike required flag.
- **Executive Summary** — business goal, chosen solution (one sentence), effort totals with Oversight Multipliers, success criteria, one-sentence risk summary, key assumptions, hard dependencies.
- **Requirements** — use cases, acceptance criteria, success metrics, constraints, in/out of scope.
- **Solution Options** — one section per candidate with description, repo impact, Oversight Multiplier, and key trade-off. Mark the chosen one.
- **RAD Dashboard** — every item has: Type (Behavioral Loss / Ecosystem Stability / Technical), Impact, Confidence, Source, Solution(s) Affected, Mitigation/Action Item, Knowledge Owner.
- **Tradeoff Matrix** — the comparison table from Phase 6.
- **Recommendation** — chosen solution, rationale, main risk accepted.
- **Milestones** — PERT columns (O/M/P + blended + confidence range), Construction + Multiplier + Total, Engineering / Operational / Rollout, AC reference, risk flag. M0 spike row if triggered.
- **Information Entropy Summary** — five HIE dimension scores and gaps.
- **Technical Breakdown** — one section per repo for the chosen solution, Oversight Multiplier, Knowledge Owner, Verification Overhead per file.
- **Cross-repo map** — only if `--root`.
- **Engineer Updates** — empty table, leave for the team.

End with:
> **Share with PMs / leadership:** Leadership Dashboard, Executive Summary, Requirements, Recommendation, Milestones
> **Share with engineers:** full document
