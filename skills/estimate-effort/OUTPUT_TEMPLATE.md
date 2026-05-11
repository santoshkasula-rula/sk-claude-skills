# Implementation Plan: [Project Name]

**Date:** [Today's Date]
**Status:** Draft — Awaiting Engineer Sign-off
**Prepared by:** [Author / Team]

---

## Executive Summary

> Written for PMs and senior leadership. No repo names. No technical jargon.

[1–2 sentences describing what is being built and why it matters to the business.]

| | |
| :--- | :--- |
| **Business Goal** | [e.g. Reduce checkout drop-off by 15%] |
| **Success Criteria** | [How we'll know this is done — e.g. "Checkout completion rate ≥ 85% in staging load test"] |
| **Business Systems Affected** | [Plain-language names — e.g. Checkout Flow, Payment Processing, Customer Portal] |
| **Raw Estimate** | [X weeks] across [N] milestones |
| **Delivery Estimate** | [X weeks + 20% buffer] — Target completion: [Date] |
| **Risk Summary** | [e.g. "One milestone carries high risk due to legacy auth layer — see RAD section." or "No high-risk milestones identified."] |
| **Target Start** | [Date] |

---

## Scope

### In Scope
- [Feature or capability included in this phase]
- [Feature or capability included in this phase]

### Out of Scope
- [Adjacent thing explicitly excluded — e.g. "Admin reporting dashboard (Phase 2)"]
- [Adjacent thing explicitly excluded]

---

## Milestones

> Each milestone delivers something demonstrable. Sequenced so each one unblocks the next.

| # | Milestone | Deliverable | Why It Matters | Effort (feature + test/review) | Target Date | Risk |
| :---: | :--- | :--- | :--- | :---: | :---: | :---: |
| M1 | [Name] | [Concrete artifact — e.g. "API contract + DB schema merged to main"] | [Business value — e.g. "Unblocks parallel frontend and backend work"] | [X days + Y days] | [Date] | 🔴 / 🟡 / 🟢 |
| M2 | [Name] | [Concrete artifact] | [Business value] | [X days + Y days] | [Date] | 🔴 / 🟡 / 🟢 |
| M3 | [Name] | [Concrete artifact] | [Business value] | [X days + Y days] | [Date] | 🔴 / 🟡 / 🟢 |

**Effort totals:**
- Raw: [sum of all milestone days]
- With 20% buffer: [buffered total]
- Target completion: [Date]

**Risk legend:** 🔴 High · 🟡 Medium · 🟢 Low

---

## RAD Analysis

> The "no surprises" section. Source column shows where each item came from.

### Risks

| Risk | Source | Affected Milestone | Mitigation |
| :--- | :---: | :---: | :--- |
| [Description] | Transcript / PRD / RAD interview | M[N] | [Concrete step to reduce impact — e.g. "Spike 1 day on legacy auth layer before M1 estimate is locked"] |

### Assumptions

| Assumption | Source | Owner | Must be confirmed by |
| :--- | :---: | :--- | :--- |
| [e.g. Staging environment is accessible to all engineers] | RAD interview | Platform team | Before M1 start |

### Dependencies

| Dependency | Type | Source | Blocks | Status |
| :--- | :---: | :---: | :--- | :--- |
| [e.g. `billing-engine` schema migration complete] | Internal | Transcript | M2 | Waiting |
| [e.g. Stripe API key rotation approved] | External | RAD interview | M3 | Pending |

### Out of Scope (confirmed)

- [Item explicitly excluded and its rationale — e.g. "Admin dashboard excluded: separate backlog item, no dependency on this work"]

---

## Technical Breakdown

> For engineers. One section per repo. Enough context to start, not a full spec.

### `[repo-name]`

**Business system:** [Plain-language name — e.g. Payment Processing]
**Milestones:** M1, M2

| Area / File | Change needed | Rationale |
| :--- | :--- | :--- |
| `src/domain/service.go` | [What to add or change] | [Why — e.g. "Encapsulates new idempotency logic per PRD requirement"] |
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

> Living document. Engineers: log estimate changes here as you uncover more context. PMs: check this table for the most current delivery date.

| Date | Engineer | Milestone | What changed / Why | Revised Estimate | New Target Date |
| :--- | :--- | :---: | :--- | :---: | :---: |
| | | | | | |
