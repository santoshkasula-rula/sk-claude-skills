# Effort Breakdown Analysis - [Project Name]

**Date:** [Today's Date]
**Status:** Draft — Awaiting Engineer Sign-off
**Prepared by:** [Author / Team]

---

## Executive Summary

> Written for PMs and senior leadership. No repo names. No technical jargon.

[1–2 sentences: what is being built, why it matters to the business, and which solution approach was chosen.]

| | |
| :--- | :--- |
| **Business Goal** | [e.g. Reduce checkout drop-off by 15%] |
| **Chosen Approach** | [Solution name — one sentence on what it is] |
| **Success Criteria** | [How we'll know this is done] |
| **Business Systems Affected** | [Plain-language names — e.g. Checkout Flow, Payment Processing] |
| **Engineering Effort** | [X days raw] → [X days + 20% buffer] |
| **Operational Effort** | [X days — deployment, infra, config, secrets] |
| **Rollout Effort** | [X days — QA, testing, feature flag ramp, stakeholder demo] |
| **Total Effort** | [Buffered Engineering + Operational + Rollout] |
| **Risk Summary** | [One sentence — e.g. "One milestone carries high risk due to legacy auth layer — see RAD."] |
| **Key Assumptions** | [e.g. "Staging access available; v2 payments endpoint is idempotent"] |
| **Hard Dependencies** | [e.g. "Payment service schema migration must complete before M2"] |

---

## Requirements

> The agreed-upon problem definition. Locked before solutions were evaluated.

### Use Cases

| Actor | Action | Goal |
| :--- | :--- | :--- |
| [e.g. Logged-in user] | [e.g. Saves payment method at checkout] | [e.g. Faster repeat purchases] |

### User Stories

- As a [role], I want to [action] so that [outcome].

### Acceptance Criteria

- [ ] [Specific, testable condition — e.g. "Payment method saved successfully shows confirmation within 2s"]
- [ ] [...]

### Success Metrics

- [Measurable outcome — e.g. "Checkout completion rate ≥ 85% in staging load test"]

### Constraints

- [Non-negotiable — e.g. "Must not store raw card data — use tokenization only"]

### Scope

**In scope:**
- [Feature or capability included in this phase]

**Out of scope:**
- [Adjacent thing explicitly excluded — e.g. "Admin reporting dashboard (Phase 2)"]

---

## Solution Options

> Each option is a distinct functional design. The chosen approach is marked.

### Option A — [Name] ✅ *(Chosen)*

[2–3 sentences: what this solution does and how it addresses the requirements.]

**Repo / service impact:**
- `[repo/service]` — [what changes]
- `[repo/service]` — [what changes]

**Key trade-off:** [The main thing accepted with this approach]

---

### Option B — [Name]

[2–3 sentences: what this solution does and how it addresses the requirements.]

**Repo / service impact:**
- `[repo/service]` — [what changes]

**Key trade-off:** [The main thing accepted with this approach]

---

### Option C — [Name] *(if applicable)*

[Description]

**Repo / service impact:**
- `[repo/service]` — [what changes]

**Key trade-off:** [The main thing accepted]

---

## RAD Analysis

> The "no surprises" section. Source column shows where each item came from. Solution column shows which option(s) it affects.

### Risks

| Risk | Source | Solution(s) Affected | Affected Milestone | Mitigation |
| :--- | :---: | :---: | :---: | :--- |
| [Description] | Transcript / PRD / Engineer | A / B / All | M[N] | [Concrete step — e.g. "Spike 1 day on legacy auth before M1 is locked"] |

### Assumptions

| Assumption | Source | Solution(s) Affected | Owner | Must be confirmed by |
| :--- | :---: | :---: | :--- | :--- |
| [e.g. "Staging environment accessible to all engineers"] | Engineer | All | Platform team | Before M1 start |

### Dependencies

| Dependency | Type | Source | Solution(s) Affected | Blocks | Status |
| :--- | :---: | :---: | :---: | :--- | :--- |
| [e.g. `billing-engine` schema migration complete] | Internal | Transcript | A, B | M2 | Waiting |
| [e.g. Stripe API key rotation approved] | External | Engineer | A | M3 | Pending |

### Out of Scope (confirmed)

- [Item and rationale — e.g. "Admin dashboard: separate backlog item, no dependency on this work"]

---

## Tradeoff Matrix

> Comparison of all solution options across the dimensions that matter for this project.

| | [Option A] | [Option B] | [Option C] |
| :--- | :--- | :--- | :--- |
| **Effort** | [Low/Med/High — brief reason] | ... | ... |
| **Risk exposure** | [RAD items inherited] | ... | ... |
| **Repo impact breadth** | [Narrow / Moderate / Broad] | ... | ... |
| **Meets all requirements** | Yes / Partially — [gap] | ... | ... |
| **Long-term maintainability** | [signal] | ... | ... |
| **Key trade-off** | [what you give up] | ... | ... |

[2–3 sentences of narrative: what the table doesn't capture — team familiarity, reversibility, sequencing implications.]

---

## Recommendation

**Chosen approach: [Option Name]**

[2–3 sentences: why this solution best fits the requirements given the constraints, risks, and team context. Be direct about the trade-off being accepted.]

**Main risk accepted:** [One honest sentence about what could still go wrong.]

---

## Milestones

> For the chosen solution. Each milestone delivers something demonstrable. Sequenced so each unblocks the next.

| # | Milestone | Deliverable | Acceptance Criterion Met | Engineering | Operational | Rollout | Risk |
| :---: | :--- | :--- | :--- | :---: | :---: | :---: | :---: |
| M1 | [Name] | [Concrete artifact — e.g. "API contract + DB schema merged to main"] | [Which AC from Requirements this satisfies] | [X days] | [Y days] | [Z days] | 🔴/🟡/🟢 |
| M2 | [Name] | [Concrete artifact] | [AC reference] | [X days] | [Y days] | [Z days] | 🔴/🟡/🟢 |
| M3 | [Name] | [Concrete artifact] | [AC reference] | [X days] | [Y days] | [Z days] | 🔴/🟡/🟢 |

**Effort totals:**
- Engineering: [raw sum] → [+20% buffer]
- Operational: [sum — no buffer]
- Rollout: [sum — no buffer]
- **Total: [all three combined]**

**Risk legend:** 🔴 High · 🟡 Medium · 🟢 Low

---

## Technical Breakdown

> For engineers. One section per repo for the chosen solution. Enough context to start, not a full spec.

### `[repo-name]`

**Business system:** [Plain-language name — e.g. Payment Processing]
**Milestones:** M1, M2
**Design decision:** [e.g. "Sync API call over event bus — existing service calls here are all synchronous and async infra isn't in place"]

| Area / File | Change needed | Rationale |
| :--- | :--- | :--- |
| `src/domain/service.go` | [What to add or change] | [Why — ties to requirement or RAD risk] |
| `src/api/handler.go` | [What to add or change] | [Why] |

**Testing required:**
- [ ] Unit tests for [component]
- [ ] Integration test for [flow]

---

### `[repo-name-2]`

**Business system:** [Plain-language name]
**Milestones:** M2, M3
**Depends on:** `[repo-name]` completing M1 first

| Area / File | Change needed | Rationale |
| :--- | :--- | :--- |
| `config/schema.sql` | [What to add] | [Why] |

**Testing required:**
- [ ] [Test type and scope]

---

## Cross-Repo Dependency Map

> Only present when multiple repos are in scope (`--root` mode).

```
M1: [repo-a] — [deliverable]
  └─► M2: [repo-b] — depends on [repo-a] M1 completing first
        └─► M3: [repo-a] + [repo-b] — [deliverable]
```

**Sequencing risk:** [Note any milestone where a delay in one repo cascades to others]

---

## Engineer Updates

> Living document. Engineers: log estimate changes here as you uncover more context. PMs: check this table for the most current delivery picture.

| Date | Engineer | Milestone | What changed / Why | Engineering | Operational | Rollout |
| :--- | :--- | :---: | :--- | :---: | :---: | :---: |
| | | | | | | |
