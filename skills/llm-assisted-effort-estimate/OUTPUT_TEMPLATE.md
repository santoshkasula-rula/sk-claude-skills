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
| **Confidence Score** | [X/10] — *[Top 1–2 reasons it isn't higher — e.g. "Bus factor risk unresolved; external API stability unconfirmed"]* |
| **Blended Effort (PERT)** | **[X days]** — *70% chance: [M] days · 95% chance: [P] days* |
| **Delivery Path Chosen** | Option A (Fastest MVP) / Option B (Robust) |
| **Spike Required?** | Yes — see M0 / No |

---

## Executive Summary

> Written for PMs and senior leadership. No repo names. No technical jargon.

[1–2 sentences: what is being built and why it matters to the business.]

| | |
| :--- | :--- |
| **Business Goal** | [e.g. Reduce checkout drop-off by 15%] |
| **Success Criteria** | [How we'll know this is done — e.g. "Checkout completion rate ≥ 85% in staging load test"] |
| **Business Systems Affected** | [Plain-language names — e.g. Checkout Flow, Payment Processing] |
| **Construction Effort** | [X days — raw LLM-assisted build time] |
| **Oversight Multiplier** | [e.g. 3.5× — cross-service integration + one high-risk auth module at 4.0×] |
| **Total Engineering Effort** | [Construction × Multiplier → blended PERT days] |
| **Operational Effort** | [X days — deployment, infra, config, secrets] |
| **Rollout Effort** | [X days — QA, testing, feature flag ramp, stakeholder demo] |
| **Total Effort** | [Engineering + Operational + Rollout] |
| **Risk Summary** | [e.g. "One milestone blocked pending spike on legacy auth layer — estimate locked only after spike completes."] |
| **Key Assumptions** | [e.g. "Staging access available; v2 payments endpoint is idempotent"] |
| **Hard Dependencies** | [e.g. "Payment service schema migration must complete before M2"] |

> ⚠️ *~70% of the effort above is in validation and integration. LLM-assisted construction is the smaller part — the Oversight Multiplier captures the review, hardening, and iteration overhead.*

---

## Delivery Trade-offs

> PM's choice. Both paths are valid — pick the one that fits the business situation.

### Option A — Fastest Path (MVP)

**What's included:** [scope]
**What's deferred:** [what's cut and when it can be picked up]
**Blended estimate:** [X days] — *70% at [M] days, 95% at [P] days*
**Oversight Multiplier:** [e.g. 2.5× — reduced integration surface]
**Trade-off accepted:** [specific tech debt, behavioral loss risk, or validation shortcut — be concrete]

### Option B — Robust Path *(default)*

**What's included:** [full scope + hardening]
**Blended estimate:** [X days] — *70% at [M] days, 95% at [P] days*
**Oversight Multiplier:** [e.g. 3.5× — full integration hardening]
**Trade-off accepted:** [longer timeline; lower behavioral loss and regression risk long-term]

**Chosen:** Option [A/B] — *[one sentence: why this path fits the current business situation]*

---

## Scope

### In Scope
- [Feature or capability included in this phase]

### Out of Scope
- [Adjacent thing explicitly excluded — e.g. "Admin reporting dashboard (Phase 2)"]

---

## RAD Dashboard

> Leadership focus: every item shows impact, confidence level, and a concrete action item. Behavioral Loss and Ecosystem Stability risks are called out explicitly.

### Risks

| Risk | Type | Impact | Confidence | Source | Affected Milestone | Mitigation / Action Item | Knowledge Owner |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- | :--- |
| [e.g. "Legacy auth layer has no test coverage"] | Behavioral Loss / Technical / Ecosystem | High / Med / Low | High / Low | Transcript / PRD / Probe | M[N] | [e.g. "Spike 1 day before locking M1. Owner: [name]"] | [who owns this area] |

### Assumptions

| Assumption | Type | Impact | Confidence | Source | Owner | Must be confirmed by |
| :--- | :---: | :---: | :---: | :---: | :--- | :--- |
| [e.g. "v2 payments API is stable — no breaking changes planned"] | Ecosystem Stability | High / Med / Low | High / Low | Probe | Platform team | Before M1 start |

### Dependencies

| Dependency | Type | Impact | Source | Blocks | Status | Human Bottleneck |
| :--- | :---: | :---: | :---: | :--- | :--- | :--- |
| [e.g. `billing-engine` schema migration] | Internal | Critical / Minor | Transcript | M2 | Waiting | [name — the one person who must approve] |
| [e.g. Stripe API key rotation] | External | Critical / Minor | Probe | M3 | Pending | [name] |

### Out of Scope (confirmed)

- [Item and rationale — e.g. "Admin dashboard: separate backlog item, no dependency on this work"]

---

## Milestones

> Three phases: Validate the biggest unknown → Build → Launch. Sequenced so each unblocks the next.
> Construction × Oversight Multiplier = Total Engineering per milestone.

| # | Phase | Milestone | Deliverable | O | M | P | Blended | Confidence | Construction | Multiplier | Engineering | Operational | Rollout | Risk |
| :---: | :--- | :--- | :--- | :---: | :---: | :---: | :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| M0 | Spike | [Name — only if Spike Rule triggered] | [e.g. "Spike report: legacy auth risk is/isn't a blocker — estimate unlocked or revised"] | — | — | — | 1 day | — | 1 day | 1.0× | 1 day | — | — | 🔴 |
| M1 | Validation | [Name] | [Concrete artifact] | [d] | [d] | [d] | [(O+4M+P)/6] | *70%: [M]d · 95%: [P]d* | [d] | [e.g. 2.5×] | [d] | [d] | [d] | 🔴/🟡/🟢 |
| M2 | Core Build | [Name] | [Concrete artifact — e.g. "Feature-complete behind flag, passing integration tests"] | [d] | [d] | [d] | [(O+4M+P)/6] | *70%: [M]d · 95%: [P]d* | [d] | [e.g. 3.5×] | [d] | [d] | [d] | 🔴/🟡/🟢 |
| M3 | Launch Readiness | [Name] | [Concrete artifact — e.g. "Zero-regression sign-off, 100% flag rollout"] | [d] | [d] | [d] | [(O+4M+P)/6] | *70%: [M]d · 95%: [P]d* | [d] | [e.g. 2.5×] | [d] | [d] | [d] | 🔴/🟡/🟢 |

**PERT:** O = Optimistic (AI-Accelerationist) · M = Most Likely · P = Pessimistic (Skeptical SRE) · Blended = (O + 4M + P) / 6

**Effort totals:**
- Engineering: [construction sum] × [weighted multiplier] = [total engineering days]
- Operational: [sum — no multiplier]
- Rollout: [sum — no multiplier]
- **Total (blended PERT): [all three combined]**
- **Full range: 70% confidence → [M total] days · 95% confidence → [P total] days**

**Risk legend:** 🔴 High · 🟡 Medium · 🟢 Low

---

## Information Entropy Summary

> Shows how complete the input context was. Lower entropy = higher confidence score.

| HIE Dimension | Score | Gap / Note |
| :--- | :---: | :--- |
| Reasoning Complexity | Complete / Partial / Missing | [e.g. "Cross-module integration confirmed in transcript"] |
| Context Completeness | Complete / Partial / Missing | [e.g. "Undocumented internal API — flagged in RAD"] |
| Transformation Impact | Complete / Partial / Missing | [e.g. "3 downstream services confirmed in Phase 1c"] |
| Verification Overhead | Complete / Partial / Missing | [e.g. "Auth module touched — 4.0× multiplier applied to M2"] |
| Iteration Cycles | Complete / Partial / Missing | [e.g. "No prior work in this area — 2 rework cycles estimated"] |

---

## Technical Breakdown

> For engineers. One section per repo. Enough context to start, not a full spec.

### `[repo-name]`

**Business system:** [Plain-language name — e.g. Payment Processing]
**Milestones:** M1, M2
**Oversight Multiplier:** [e.g. 4.0× — Auth module; 3:1 review ratio applies]
**Design decision:** [e.g. "Sync API call over event bus — existing service calls here are all synchronous and async infra isn't in place"]
**Knowledge Owner:** [who to loop in for this area — Bus Factor check]

| Area / File | Change needed | Verification Overhead | Rationale |
| :--- | :--- | :---: | :--- |
| `src/domain/service.go` | [What to add or change] | High / Med / Low | [Why — ties to PRD requirement or RAD risk] |
| `src/api/handler.go` | [What to add or change] | High / Med / Low | [Why] |

**Testing required:**
- [ ] Unit tests for [component]
- [ ] Integration test for [flow]
- [ ] Security / compliance review for [high-risk component — if multiplier ≥ 3.5×]

---

### `[repo-name-2]`

**Business system:** [Plain-language name]
**Milestones:** M2, M3
**Oversight Multiplier:** [e.g. 2.5×]
**Depends on:** `[repo-name]` completing M1 first
**Knowledge Owner:** [name]

| Area / File | Change needed | Verification Overhead | Rationale |
| :--- | :--- | :---: | :--- |
| `config/schema.sql` | [What to add] | Low | [Why] |

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

| Date | Engineer | Milestone | What changed / Why | Construction | Multiplier | Engineering | Operational | Rollout |
| :--- | :--- | :---: | :--- | :---: | :---: | :---: | :---: | :---: |
| | | | | | | | | |
