# Implementation Report - [Project Name]

**Date:** [Today's Date]
**Status:** Draft — Awaiting Engineer Sign-off
**Prepared by:** [Author / Team]

---

## Leadership Dashboard

> One-glance status for PMs and Directors.

| | |
| :--- | :--- |
| **Project Health** | 🟢 On Track / 🟡 At Risk / 🔴 High Uncertainty |
| **Confidence Score** | [X/10] — *[Top reason it isn't higher — e.g. "External dependency timeline unconfirmed"]* |
| **Blended Effort (PERT)** | **[X days]** *(O: [X] · M: [X] · P: [X])* |
| **Delivery Path Chosen** | Option A (Fastest) / Option B (Robust) |

---

## Executive Summary

> Written for PMs and senior leadership. No repo names. No technical jargon.

[1–2 sentences: what is being built and why it matters to the business.]

| | |
| :--- | :--- |
| **Business Goal** | [e.g. Reduce checkout drop-off by 15%] |
| **Success Criteria** | [How we'll know this is done — e.g. "Checkout completion rate ≥ 85% in staging load test"] |
| **Business Systems Affected** | [Plain-language names — e.g. Checkout Flow, Payment Processing] |
| **Engineering Effort** | [X days raw] → [X days + 20% buffer] |
| **Operational Effort** | [X days — deployment, infra, config, secrets] |
| **Rollout Effort** | [X days — QA, testing, feature flag ramp, stakeholder demo] |
| **Total Effort** | [Buffered Engineering + Operational + Rollout] |
| **Risk Summary** | [e.g. "One milestone is high-risk due to legacy auth layer — see RAD Dashboard."] |
| **Key Assumptions** | [e.g. "Staging access available; v2 payments endpoint is idempotent"] |
| **Hard Dependencies** | [e.g. "Payment service schema migration must complete before M2"] |

---

## Delivery Trade-offs

> PM's choice. Both paths are valid — pick the one that fits the business situation.

### Option A — Fastest Path (MVP)

**What's included:** [scope]
**What's deferred:** [what's cut and when it can be picked up]
**Blended estimate:** [X days]
**Trade-off accepted:** [specific tech debt, risk, or quality shortcut — be concrete]

### Option B — Robust Path *(default)*

**What's included:** [full scope + hardening]
**Blended estimate:** [X days — longer than A]
**Why it takes longer:** [what the extra time buys — scalability, coverage, reduced future debt]

**Chosen:** Option [A/B] — *[one sentence: why this path was selected]*

---

## Scope

### In Scope
- [Feature or capability included in this phase]

### Out of Scope
- [Adjacent thing explicitly excluded — e.g. "Admin reporting dashboard (Phase 2)"]

---

## RAD Dashboard

> Leadership focus: every item shows impact on delivery and a concrete action item. No surprises.

### Risks

| Risk | Impact on Delivery | Source | Affected Milestone | Mitigation / Action Item |
| :--- | :---: | :---: | :---: | :--- |
| [Description — e.g. "Legacy auth layer has no test coverage"] | High / Med / Low | Transcript / PRD / Probe | M[N] | [Concrete step — e.g. "Spike 1 day before locking M1 estimate. Owner: [name]"] |

### Assumptions

| Assumption | Impact on Delivery | Source | Owner | Must be confirmed by |
| :--- | :---: | :---: | :--- | :--- |
| [e.g. "Staging environment accessible to all engineers"] | High / Med / Low | Probe | Platform team | Before M1 start |

### Dependencies

| Dependency | Type | Impact | Source | Blocks | Status | Owner |
| :--- | :---: | :---: | :---: | :--- | :--- | :--- |
| [e.g. `billing-engine` schema migration] | Internal | Critical / Minor | Transcript | M2 | Waiting | [name] |
| [e.g. Stripe API key rotation] | External | Critical / Minor | Probe | M3 | Pending | [name] |

### Out of Scope (confirmed)

- [Item and rationale — e.g. "Admin dashboard: separate backlog item, no dependency on this work"]

---

## Milestones

> Three phases: Validate the biggest unknown → Build → Launch. Sequenced so each unblocks the next.

| # | Phase | Milestone | Deliverable | O | M | P | Blended | Engineering | Operational | Rollout | Risk |
| :---: | :--- | :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| M1 | Validation Spike | [Name] | [Concrete artifact — e.g. "Spike report: legacy auth layer is/isn't a blocker"] | [d] | [d] | [d] | [(O+4M+P)/6] | [d] | [d] | [d] | 🔴/🟡/🟢 |
| M2 | Core Build | [Name] | [Concrete artifact — e.g. "Feature-complete behind flag, passing integration tests"] | [d] | [d] | [d] | [(O+4M+P)/6] | [d] | [d] | [d] | 🔴/🟡/🟢 |
| M3 | Launch Readiness | [Name] | [Concrete artifact — e.g. "Zero-regression sign-off, 100% flag rollout"] | [d] | [d] | [d] | [(O+4M+P)/6] | [d] | [d] | [d] | 🔴/🟡/🟢 |

**PERT columns:** O = Optimistic · M = Most Likely · P = Pessimistic · Blended = (O + 4M + P) / 6

**Effort totals:**
- Engineering: [raw sum] → [+20% buffer]
- Operational: [sum — no buffer]
- Rollout: [sum — no buffer]
- **Total (blended PERT): [all three combined]**

> ⚠️ *~70% of the effort above is in validation and integration — code generation is the smaller part.*

**Risk legend:** 🔴 High · 🟡 Medium · 🟢 Low

---

## Technical Breakdown

> For engineers. One section per repo. Enough context to start, not a full spec.

### `[repo-name]`

**Business system:** [Plain-language name — e.g. Payment Processing]
**Milestones:** M1, M2
**Design decision:** [e.g. "Sync API call over event bus — existing service calls here are all synchronous and async infra isn't in place"]

| Area / File | Change needed | Rationale |
| :--- | :--- | :--- |
| `src/domain/service.go` | [What to add or change] | [Why — ties to PRD requirement or RAD risk] |
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

> Living document. Engineers: log estimate changes here as you uncover more context. PMs: check this table for the latest delivery picture.

| Date | Engineer | Milestone | What changed / Why | Engineering | Operational | Rollout |
| :--- | :--- | :---: | :--- | :---: | :---: | :---: |
| | | | | | | |
