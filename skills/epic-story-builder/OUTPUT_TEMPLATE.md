# Epics & Stories - [Project Name]

**Generated from:** [Effort Breakdown Analysis filename]
**Date:** [Today's Date]
**Status:** Draft — Awaiting team review

---

## Jira Import Data

> Structured data for `/jira-import`. Do not edit manually — update the markdown sections below and regenerate.

```yaml
project: "[Project Name]"
generated: "[Today's Date]"
epics:
  - id: "E1"
    name: "[Milestone Name]"
    goal: "[Business value — exact language from Effort Breakdown Analysis]"
    risk: "high | medium | low"
    labels: ["[milestone-slug]"]
    stories:
      - id: "1.1"
        title: "[Action-language story title]"
        type: "Engineering | Operational | Rollout"
        repo: "[repo-name]"
        effort_days: 0
        demo_scenario: "[What the engineer shows on Friday]"
        done_when: "[Acceptance criterion]"
        rad_flag: null
        labels: ["[type-slug]", "[repo-slug]"]
      - id: "1.2"
        title: "[Action-language story title]"
        type: "Engineering | Operational | Rollout"
        repo: "[repo-name]"
        effort_days: 0
        demo_scenario: "[What the engineer shows on Friday]"
        done_when: "[Acceptance criterion]"
        rad_flag: "[RAD item name]"
        labels: ["[type-slug]", "[repo-slug]"]
  - id: "E2"
    name: "[Milestone Name]"
    goal: "[Business value]"
    risk: "high | medium | low"
    labels: ["[milestone-slug]"]
    stories:
      - id: "2.1"
        title: "[Action-language story title]"
        type: "Engineering | Operational | Rollout"
        repo: "[repo-name]"
        effort_days: 0
        demo_scenario: "[Demo scenario]"
        done_when: "[Done when]"
        rad_flag: null
        labels: ["[type-slug]", "[repo-slug]"]
totals:
  engineering_days: 0
  operational_days: 0
  rollout_days: 0
  total_days: 0
```

---

## Summary

| | |
| :--- | :--- |
| **Total Epics** | [N] |
| **Total Stories** | [N] |
| **Engineering** | [X]d |
| **Operational** | [X]d |
| **Rollout** | [X]d |
| **Total Effort** | [X]d |

---

## Epic 1: [Milestone Name]

**Goal:** [Business value — exact language from the Effort Breakdown Analysis]
**Effort:** Engineering [X]d · Operational [Y]d · Rollout [Z]d
**Risk:** 🔴 / 🟡 / 🟢

| # | Story | Type | Repo | Effort | Demo Scenario | Done When | RAD Flag |
| :--- | :--- | :---: | :--- | :---: | :--- | :--- | :--- |
| 1.1 | [Action-language title] | Engineering | `repo-a` | [X]d | [What the engineer shows on Friday] | [Acceptance criterion] | — |
| 1.2 | [Action-language title] | Engineering | `repo-a` | [X]d | [What the engineer shows on Friday] | [Acceptance criterion] | ⚠️ [RAD item] |
| 1.3 | [Action-language title] | Operational | `infra` | [X]d | [e.g. "Show new env vars live in staging"] | [Acceptance criterion] | — |
| 1.4 | [Action-language title] | Rollout | — | [X]d | [e.g. "Walk through end-to-end flow in staging with PM"] | [Acceptance criterion] | — |

---

## Epic 2: [Milestone Name]

**Goal:** [Business value]
**Effort:** Engineering [X]d · Operational [Y]d · Rollout [Z]d
**Risk:** 🔴 / 🟡 / 🟢

| # | Story | Type | Repo | Effort | Demo Scenario | Done When | RAD Flag |
| :--- | :--- | :---: | :--- | :---: | :--- | :--- | :--- |
| 2.1 | [Action-language title] | Engineering | `repo-b` | [X]d | [Demo scenario] | [Done when] | — |

---

## Effort Totals

| Epic | Engineering | Operational | Rollout | Total |
| :--- | :---: | :---: | :---: | :---: |
| [Epic 1 name] | [X]d | [Y]d | [Z]d | [total]d |
| [Epic 2 name] | [X]d | [Y]d | [Z]d | [total]d |
| **Total** | **[X]d** | **[Y]d** | **[Z]d** | **[total]d** |

---

## RAD Flags Index

> Stories flagged with ⚠️ touch one of these items. Review before sprint planning.

| Flag | Type | Detail | Stories affected |
| :--- | :---: | :--- | :--- |
| [RAD item name] | Risk / Assumption / Dependency | [One-line description] | 1.2, 2.3 |

---

**Next step:** Review story list with the team, adjust sizing, then run `/jira-import` to push to Jira. *(coming soon)*
