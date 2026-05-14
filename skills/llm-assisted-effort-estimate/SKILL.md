---
name: llm-assisted-effort-estimate
description: Strategic engineering thought partner using HIE + RAD-E + PERT frameworks grounded in 2026 LLM-assisted estimation research. Audits context entropy, scores oversight multipliers per module risk, runs internal multi-agent debate for probabilistic ranges, and produces a leadership-ready implementation report.
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
| **Reasoning Complexity** | Does this require cross-module integration or multi-step logic? More = more interaction/fix cycles. |
| **Context Completeness** | Is all required knowledge explicit in documents, or ambient/undocumented? Gaps increase hallucination risk. |
| **Transformation Impact** | How many existing dependencies change? Wide-reaching changes scale risk exponentially. |
| **Verification Overhead** | High-risk modules (Auth, Payments, PII) require a 3:1 human oversight ratio — 3 hrs review per 1 hr build. |
| **Iteration Cycles** | How many refinement loops before production-ready? Each loop is non-trivial. |

### Oversight Multiplier (replaces flat % buffer)
`Total Effort = Construction Time × Oversight Multiplier`

| Module Risk Profile | Multiplier |
| :--- | :--- |
| Routine change, well-tested, no external deps | 1.5× |
| New feature, moderate integration surface | 2.5× |
| Complex cross-service or unfamiliar system | 3.5× |
| Auth / Payments / PII / compliance-critical | 4.0–5.0× |

Always show: Construction estimate + Multiplier applied + Total. Never embed the multiplier invisibly.

### RAD-E — Risk, Assumptions, Dependencies (2026 KLRM focus)
- **Risks** → flag **Behavioral Loss**: will the team understand this code in 6 months?
- **Assumptions** → flag **Ecosystem Stability**: is the 3rd-party API/service this depends on stable?
- **Dependencies** → flag **Human Bottlenecks**: who is the single person who must approve or unblock this?

### PERT — Probabilistic Estimation
O/M/P estimates generated via internal multi-agent debate (see Phase 4). Blended = `(O + 4M + P) / 6`. Express confidence as: *"70% chance of [M] days, 95% chance of [P] days."* Never give a single-point estimate.

### 1-Hour Spike Rule
If any RAD item is marked **High Impact / Low Confidence**: do not estimate that milestone. Instead, recommend a 1-hour time-boxed spike to resolve the unknown before locking numbers.

---

## Phase 1 — Collect files

Ask for each file in sequence. Read it before asking for the next.

1. **PRD** — "Path to your PRD?" — required, re-ask if unreadable.
2. **Transcript** — "Zoom transcript or meeting notes? (path or skip)" — if provided: strip timestamps, speaker labels, `[inaudible]`, merge fragmented turns into prose.
3. **Additional context** — "Any other files? (architecture docs, API specs, prior plans — path or done)" — repeat until 'done'.

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

> **I found these internal services — which need changes for this project?**
>
> - `[service-name]` — [inferred purpose]
> - *(none detected — I'll ask in the probe step)*

Wait for response. Record confirmed services — they seed the Technical Breakdown and RAD dependencies.
If no git root found: skip silently.

---

## Phase 2 — Extract signals + Information Entropy audit

From all documents, extract and record using the exact terminology from the source materials.

**Standard signals:**

| Signal | Source |
| :--- | :--- |
| Project name, objective, success metrics | PRD |
| Features in scope / out of scope | PRD |
| Technical components mentioned | Transcript / docs |
| Anxiety markers: *"worried about"*, *"legacy"*, *"tricky"*, *"not sure how long"*, *"depends on"*, *"blocked by"*, *"never touched"* | Transcript / docs |
| Named blockers, external dependencies, open questions | Transcript / docs |
| Assumptions stated or implied | All sources |

**Conflict rule:** Transcript anxiety markers override PRD optimism on the same component — always.

**Information Entropy audit** — after extracting signals, score each of the five HIE dimensions:

| Dimension | Score: Complete / Partial / Missing |
| :--- | :--- |
| Reasoning Complexity | [assess from PRD scope] |
| Context Completeness | [flag any ambient/undocumented knowledge gaps] |
| Transformation Impact | [assess from known service dependencies] |
| Verification Overhead | [identify Auth/Payments/PII/compliance touch points] |
| Iteration Cycles | [flag unknowns that imply rework loops] |

Any dimension scored **Missing** is a required question in Phase 3. Any component touching Auth, Payments, or PII automatically gets a 4.0× Oversight Multiplier flag.

---

## Phase 3 — Strategic Probe

Documents capture what was written. This step closes information entropy gaps and surfaces failure modes, behavioral loss, and human bottlenecks.

Present the pre-filled RAD draft and exactly 3 targeted questions in one message:

> **Here's what I extracted — correct anything:**
>
> **Risks:** [list with source, or "none found"]
> **Assumptions:** [list with source, or "none found"]
> **Dependencies:** [list with source, or "none found"]
> **Out of scope:** [list, or "not stated"]
> **Success criteria:** [list, or "not stated"]
>
> **Three questions before I estimate:**
>
> 1. [Question A]
> 2. [Question B]
> 3. [Question C]

**Question bank — pick the 3 most valuable, prioritizing entropy gaps from Phase 2:**

- **Legacy constraints (Context Completeness gap):** *"What constraints or decisions are NOT in this PRD — undocumented APIs, tribal knowledge, historical hacks?"*
- **Behavioral loss (Risks — KLRM):** *"Will the team understand this code in 6 months — is there a knowledge owner for the area being modified, or could it become a black box?"*
- **Bus factor (Dependencies — KLRM):** *"Who is the one person whose approval or knowledge is required to ship this — and are they available?"*
- **Failure mode:** *"What happens when this fails in production — who gets paged and what breaks downstream?"*
- **Simpler path:** *"Is there a simpler way to hit the core business goal — what's the minimum that would unlock value?"*
- **Ecosystem stability (Assumptions — KLRM):** *"How stable are the external APIs or services this depends on — any recent breaking changes or deprecations?"*
- **Trickiest part:** *"What's most likely to take twice as long — and has your team touched it before?"*
- **Dependent services (if Phase 1c found nothing):** *"Which other repos or services need changes — even small ones like config or contract updates?"*
- **Design approach:** *"How are you thinking about [key architectural decision]? Any alternatives you've ruled out?"*

**Rules:**
- Exactly 3 questions — no more, no fewer.
- Always prioritize questions that close **Missing** entropy dimensions from Phase 2.
- Never re-ask something the documents already answered.
- If the engineer describes a design approach, challenge it once: *"What's the main risk with that? Did you consider [alternative]?"* then move on.

Wait for response. Update RAD and entropy scores before continuing.

---

## Phase 3b — Codebase tradeoff scan (opt-in)

After RAD is confirmed, ask once:

> **Scan the codebase for design tradeoffs?** (default: no — takes a few minutes)
> - **Yes** — I'll scan areas the PRD touches and surface 2–4 grounded decisions
> - **No / skip** — continue with what you've told me

- If **yes**: scan (Phase 3c), then continue.
- If **no** or no response: continue.
- If the engineer pastes design constraints: record them, skip scan, continue.

---

## Phase 3c — Scan (only if yes)

Use `find`, `ls`, and `Read` on key files. Focus on areas the PRD touches. Look for:
- How similar features are structured (routing, layering, data access)
- How cross-service calls are made (sync vs async, retry, error handling)
- Tech debt signals and behavioral loss indicators near the areas this feature will touch

Form 2–4 tradeoffs. Each must be grounded in something observed — not generic best practice — and present Option A / Option B with one-line consequences, flagging which is safer given the RAD risks and Oversight Multiplier implications.

> **Design tradeoffs — your call:**
>
> *[What you observed, what the decision is]*
> - **A:** [option] → [consequence] *(Multiplier implication: [e.g. "keeps multiplier at 2.5×"])*
> - **B:** [option] → [consequence] *(Multiplier implication: [e.g. "pushes multiplier to 4.0× — touches auth layer"])*
> - *Leans toward A/B because [RAD risk or codebase reason]*

Wait for response. Record decisions — they inform the Technical Breakdown and Oversight Multipliers.

---

## Phase 4 — Internal multi-agent debate + scoring

Before presenting anything to the engineer, internally run two estimation personas:

**The AI-Accelerationist** — assumes: full context, no blockers, LLM handles complexity, experienced team.
- Produces the **Optimistic (O)** estimate per milestone.

**The Skeptical SRE** — assumes: hidden legacy constraints, integration surprises, 3:1 oversight on risky modules, rework cycles, human bottlenecks cause waiting time.
- Produces the **Pessimistic (P)** estimate per milestone.

Use these two to anchor O and P. Set M based on the RAD completeness and entropy scores. Compute blended = `(O + 4M + P) / 6`.

**Express delivery confidence as:** *"70% chance of [M] days, 95% chance of [P] days."*

### 1-Hour Spike Rule check

Before presenting delivery paths, scan all RAD items. For any item where **Impact = High AND Confidence = Low**:

> ⚠️ **Spike required before estimating [milestone]:**
> *"[RAD item] is high-impact and unresolved. A 1-hour spike to [specific action] must happen before this milestone's estimate can be locked. Proceeding without it makes [M] unreliable."*

Flag it in the Leadership Dashboard as 🔴 and do not assign a milestone blended estimate — mark it `TBD — spike first`.

### Confidence Score

Score 1–10 based on information quality after the probe:

| Factor | Deduct |
| :--- | :--- |
| No transcript or meeting notes | −1 |
| Thin PRD (missing success criteria or scope) | −1 |
| Each HIE dimension still scored Missing after probe | −0.5 |
| Each 🔴 RAD risk | −0.5 |
| Auth/Payments/PII module touched (high Verification Overhead) | −1 |
| External dependency with no confirmed timeline | −1 |
| Bus factor risk (one person who must approve — availability unknown) | −0.5 |

Start at 9. Floor at 3. Show score and the top 1–2 reasons it isn't higher.

### Delivery trade-offs

Present two paths before milestones are written:

> **Two delivery paths — which fits the business situation?**
>
> **Option A — Fastest Path (MVP)**
> Scope: [what's cut or deferred]
> Estimate: [blended PERT] — 70% at [M], 95% at [P]
> Oversight Multiplier applied: [e.g. "2.5× — integration surface is moderate"]
> Trade-off accepted: [specific tech debt, behavioral loss, or validation shortcut]
>
> **Option B — Robust Path** *(default)*
> Scope: [full feature + hardening]
> Estimate: [blended PERT — longer] — 70% at [M], 95% at [P]
> Oversight Multiplier applied: [e.g. "3.5× — full integration hardening"]
> Trade-off accepted: [longer timeline, but lower behavioral loss and regression risk]
>
> Default is B unless there's a hard deadline or explicit PM preference for A.

Wait for response. Record chosen path — it shapes milestone scope, multipliers, and sequencing.

---

## Phase 5 — Milestones

Structure milestones in three phases. Names and deliverables must use language from the documents. No generic placeholders.

**Phase structure:**
- **Validation Spike** — eliminate the biggest RAD unknown (or resolve any 1-Hour Spike Rule flags) before committing to the build estimate
- **Core Build** — feature-complete against the chosen delivery path
- **Launch Readiness** — QA, zero-regression hardening, rollout

Each milestone must:
- Deliver a concrete, demonstrable artifact
- Be sequenced so each unblocks the next
- Show PERT effort: Optimistic / Most Likely / Pessimistic → blended `(O + 4M + P) / 6`
- Show Construction estimate, Oversight Multiplier, and Total separately
- Break effort into three buckets (never combine):
  - **Engineering** — coding, design, code review
  - **Operational** — deployment, infra, config, secrets
  - **Rollout** — QA, testing, feature flag ramp, stakeholder demo
- Carry a risk flag: 🔴 anxiety-marker or RAD risk · 🟡 uncertain · 🟢 well-understood

**If a milestone has a Spike Rule flag:** mark it `🔴 TBD — spike first` and show the spike as its own M0 milestone.

If `--root`: assign tasks to repos, sequence to respect cross-repo dependencies.

---

## Phase 6 — Write output file

Write to `Implementation Report - [Project Name].md` in the current directory. Follow `OUTPUT_TEMPLATE.md`. Key rules:

- **Leadership Dashboard** — Project Health, Confidence Score (X/10 with top reasons), blended PERT total with probabilistic confidence ranges, delivery path chosen.
- **Executive Summary** — business goal, effort totals with Oversight Multipliers shown, success criteria, one-sentence risk summary, key assumptions, hard dependencies.
- **Delivery Trade-offs** — both paths with multipliers, which was chosen and why.
- **RAD Dashboard** — every item has: Impact, Confidence (High/Low), Source, Mitigation/Action Item, Knowledge Owner. Behavioral Loss and Ecosystem Stability risks called out explicitly.
- **Milestones** — PERT columns (O/M/P + blended), probabilistic confidence line, Construction + Multiplier + Total, Engineering / Operational / Rollout, risk flag. Any Spike Rule items shown as M0.
- **Technical Breakdown** — one section per repo, specific files/areas, rationale, design decision chosen, Oversight Multiplier for each area.
- **Cross-repo map** — only if `--root`.
- **Engineer Updates** — empty table, leave for the team.

End with:
> **Share with PMs / leadership:** Leadership Dashboard, Executive Summary, Delivery Trade-offs, Milestones
> **Share with engineers:** full document
