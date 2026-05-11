# Implementation Plan: [Project Name]

**Date:** [Today's Date]
**Status:** Draft — Awaiting Engineer Sign-off
**Prepared by:** [Author / Team]

---

## Executive Summary

> For PMs and stakeholders. No technical jargon.

[1–2 sentence description of what is being built and why it matters to the business.]

| | |
| :--- | :--- |
| **Business Goal** | [e.g. Reduce checkout drop-off by 15%] |
| **Total Estimate** | [X weeks] (includes 20% risk buffer) |
| **Impacted Systems** | `repo-a`, `repo-b`, `repo-c` |
| **Target Start** | [Date or TBD] |

---

## Milestones

> Each milestone delivers something demonstrable. Earlier milestones unblock later ones.

| # | Milestone | Deliverable | Why It Matters | Effort | Risk |
| :---: | :--- | :--- | :--- | :---: | :---: |
| M1 | [Name] | [Concrete artifact] | [Business value] | [X days] | 🔴 / 🟡 / 🟢 |
| M2 | [Name] | [Concrete artifact] | [Business value] | [X days] | 🔴 / 🟡 / 🟢 |
| M3 | [Name] | [Concrete artifact] | [Business value] | [X days] | 🔴 / 🟡 / 🟢 |

**Risk legend:** 🔴 High (anxiety marker or RAD flag) · 🟡 Medium · 🟢 Low

---

## RAD Analysis

> The "no surprises" section. Engineers: update this as you learn more.

### Risks

| Risk | Source | Affected Milestone | Mitigation |
| :--- | :--- | :--- | :--- |
| [Description] | Transcript / RAD interview | M[N] | [Step to reduce impact] |

### Assumptions

| Assumption | Owner | Must be true by |
| :--- | :--- | :--- |
| [e.g. Staging environment is accessible] | [Team / Person] | [Milestone or Date] |

### Dependencies

| Dependency | Type | Blocks | Status |
| :--- | :--- | :--- | :--- |
| [e.g. `billing-engine` schema migration] | Internal | M2 | Waiting |
| [e.g. Stripe API key rotation] | External | M3 | Pending approval |

---

## Technical Breakdown

> For engineers. One section per repo. Enough context to start, not a full spec.

### `[repo-name]`

**Milestones:** M1, M2

| Area / File | Change needed | Rationale |
| :--- | :--- | :--- |
| `src/domain/service.go` | Add [X] business logic | [Why] |
| `src/api/handler.go` | Update request validation for [Y] | [Why] |

---

### `[repo-name-2]`

**Milestones:** M2, M3
**Depends on:** `[repo-name]` completing M1 first

| Area / File | Change needed | Rationale |
| :--- | :--- | :--- |
| `config/schema.sql` | Add [table/column] | [Why] |

---

## Cross-Repo Dependency Map

> Only present when multiple repos are in scope.

```
M1: [repo-a] — API contract + DB schema
  └─► M2: [repo-b] — consumes new [repo-a] endpoint
        └─► M3: [repo-a] + [repo-b] — hardening and load testing
```

---

## Engineer Updates

> Engineers: use this table to log estimate changes as you uncover more context. This is a living document.

| Date | Engineer | Milestone | Update / Pivot | Revised Estimate |
| :--- | :--- | :--- | :--- | :--- |
| | | | | |
