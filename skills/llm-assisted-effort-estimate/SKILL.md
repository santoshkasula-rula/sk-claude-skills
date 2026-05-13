---
name: llm-assisted-effort-estimate
description: Strategic engineering thought partner using HIE + RAD-E frameworks. Probes unknowns, scores confidence via PERT, surfaces delivery trade-offs, and produces a leadership-ready implementation report.
disable-model-invocation: true
argument-hint: "[--root]"
allowed-tools: Read Bash(find *) Bash(ls *) Write
---

# llm-assisted-effort-estimate

Read `OUTPUT_TEMPLATE.md` from `${CLAUDE_SKILL_DIR}`.
If `--root` is in $ARGUMENTS, enable multi-repo discovery in Step 2.

---

## Frameworks

- **HIE (Hybrid Intelligence Effort):** ~70% of effort is validation and integration, not code generation. Flag this whenever estimates feel low.
- **RAD (Risks, Assumptions, Dependencies):** The primary lens for hidden costs and delivery surprises.
- **PERT (Confidence-Based Estimation):** Each milestone gets Optimistic / Most Likely / Pessimistic estimates. Blended effort = `(O + 4M + P) / 6`. Never give a single-point estimate.

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

## Phase 2 — Extract signals

From all documents, extract and record using the exact terminology from the source materials:

| Signal | Source |
| :--- | :--- |
| Project name, objective, success metrics | PRD |
| Features in scope / out of scope | PRD |
| Technical components mentioned | Transcript / docs |
| Anxiety markers: *"worried about"*, *"legacy"*, *"tricky"*, *"not sure how long"*, *"depends on"*, *"blocked by"*, *"never touched"* | Transcript / docs |
| Named blockers, external dependencies, open questions | Transcript / docs |
| Assumptions stated or implied | All sources |

**Conflict rule:** Transcript anxiety markers override PRD optimism on the same component — always.

---

## Phase 3 — Strategic Probe

Documents capture what was written. This step surfaces what wasn't, and forces the engineer to think about failure modes and alternatives.

Present the pre-filled RAD draft and 3 targeted questions in one message:

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

Pick the 3 most valuable from this set — never ask all of them, and never re-ask what documents already answered:

- **Simpler path:** *"Is there a simpler way to achieve the core business goal — what's the minimum that would unlock value?"*
- **Failure mode:** *"What happens when this fails in production — who gets paged and what breaks downstream?"*
- **Trickiest part:** *"What's the part most likely to take twice as long — and has your team touched it before?"*
- **Assumptions:** *"What has to be true for this estimate to hold?"*
- **Dependent services (if Phase 1c found nothing):** *"Which other repos or services need changes — even small ones like config or contract updates?"*
- **External blockers:** *"Who or what outside your team needs to cooperate, and do they know?"*
- **Design approach:** *"How are you thinking about [key architectural decision — e.g. sync vs async, new service vs extending existing]? Any alternatives you ruled out?"*

**Rules:**
- Exactly 3 questions — no more, no fewer.
- If the engineer describes a design approach, challenge it once: *"What's the main risk with that? Did you consider [alternative]?"* then move on.

Wait for response. Update RAD before continuing.

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
- Tech debt signals near the areas this feature will touch

Form 2–4 tradeoffs. Each must be grounded in something observed — not generic best practice — and present Option A / Option B with one-line consequences, flagging which is safer given the RAD risks.

> **Design tradeoffs — your call:**
>
> *[What you observed, what the decision is]*
> - **A:** [option] → [consequence]
> - **B:** [option] → [consequence]
> - *Leans toward A/B because [RAD risk or codebase reason]*

Wait for response. Record decisions — they inform the Technical Breakdown.

---

## Phase 4 — PERT scoring + delivery paths

Compute two things before writing milestones.

### Confidence Score

Score the estimate 1–10 based on information quality:

| Factor | Deduct |
| :--- | :--- |
| No transcript or meeting notes | −1 |
| Thin PRD (missing success criteria or scope) | −1 |
| Each 🔴 risk identified | −0.5 |
| Unfamiliar system or new integration | −1 |
| External dependency with no confirmed timeline | −1 |
| Each major open question left unanswered | −0.5 |

Start at 9. Floor at 3. Show score and the top reason it isn't higher.

### Delivery trade-offs

Present two paths and ask the PM/engineer to choose before milestones are written:

> **Two delivery paths — which fits the business situation?**
>
> **Option A — Fastest Path (MVP)**
> Scope: [what's cut or deferred]
> Timeline: [PERT blended estimate]
> Trade-off: [specific tech debt or risk accepted]
>
> **Option B — Robust Path**
> Scope: [full feature + hardening]
> Timeline: [PERT blended estimate — longer]
> Trade-off: [what takes longer and why it matters]
>
> Default is B unless there's a hard deadline or explicit PM preference for A.

Wait for response. Record the chosen path — it shapes milestone scope and sequencing.

---

## Phase 5 — Milestones

Structure milestones in three phases. Use language from the documents — names and deliverables must reflect the actual words and goals from the PRD, transcript, and context. No generic placeholders.

**Phase structure:**
- **Validation Spike** — eliminate the biggest RAD unknown before committing to the full estimate
- **Core Build** — feature-complete against the chosen delivery path
- **Launch Readiness** — QA, zero-regression hardening, rollout

Each milestone must:
- Deliver a concrete, demonstrable artifact
- Be sequenced so each unblocks the next
- Show PERT effort: Optimistic / Most Likely / Pessimistic → blended `(O + 4M + P) / 6`
- Break effort into three buckets (never combine):
  - **Engineering** — coding, design, code review
  - **Operational** — deployment, infra, config, secrets
  - **Rollout** — QA, testing, feature flag ramp, stakeholder demo
- Carry a risk flag: 🔴 anxiety-marker or RAD risk · 🟡 uncertain · 🟢 well-understood

**Buffer:** 20% added to Engineering total only. Show raw + buffered separately. Never embed inside milestones.

**HIE note:** If Engineering effort across all milestones feels low relative to the complexity, flag it: *"~70% of the effort here is in validation and integration — the code generation is the smaller part."*

If `--root`: assign tasks to repos, sequence to respect cross-repo dependencies.

---

## Phase 6 — Write output file

Write to `Implementation Report - [Project Name].md` in the current directory. Follow `OUTPUT_TEMPLATE.md`. Key rules:

- **Leadership Dashboard** — Project Health (🟢/🟡/🔴), Confidence Score (X/10 with reason), Blended PERT total.
- **Executive Summary** — business goal, effort totals (raw + buffered), success criteria, one-sentence risk summary, key assumptions, hard dependencies.
- **Delivery Trade-offs** — the two paths and which was chosen, with rationale.
- **RAD Dashboard** — every item has Impact on Delivery (High/Med/Low) and a concrete Mitigation/Action Item. Source every item (PRD, Transcript, Engineer, Probe).
- **Milestones** — PERT columns (O/M/P + blended), Engineering / Operational / Rollout, risk flag.
- **Technical Breakdown** — one section per repo, specific files/areas, one-line rationale. Open each section with the Design Decision chosen and why.
- **Cross-repo map** — only if `--root`.
- **Engineer Updates** — empty table, leave for the team.

End with:
> **Share with PMs / leadership:** Leadership Dashboard, Executive Summary, Delivery Trade-offs, Milestones
> **Share with engineers:** full document
