# sk-claude-skills

Personal [Claude Code](https://claude.ai/code) skills.

## Structure

```
skills/
  <skill-name>/
    SKILL.md       # skill definition and prompt
    scripts/       # optional helper scripts
```

## Usage

To install a skill into a Claude Code project, copy the relevant `SKILL.md` into your project's `.claude/skills/` directory, or reference it in your global Claude settings.

## Skills

| Skill | Description |
|-------|-------------|
| [estimate-effort](skills/estimate-effort/SKILL.md) | Thought-partner for engineers — collects PRDs and transcripts, probes for unstated risks, design tradeoffs, and assumptions via targeted questions, then generates a PM-readable implementation plan with RAD |
| [epic-story-builder](skills/epic-story-builder/SKILL.md) | Turns an Effort Breakdown Analysis into weekly-demo-sized Epics and Stories for Jira |
| [llm-assisted-effort-estimate](skills/llm-assisted-effort-estimate/SKILL.md) | Strategic engineering thought partner using HIE + RAD-E + PERT frameworks — produces a leadership-ready report with confidence scores, delivery trade-offs, and probabilistic effort ranges |
| [sprint-goals](skills/sprint-goals/SKILL.md) | Generates a unified sprint goals summary across PAR, MARTECH, DEA, and NEV boards — synthesizes active sprint tickets into focus areas, blockers, and a leadership summary |
| [weekly-eng-report](skills/weekly-eng-report/SKILL.md) | Weekly Engineering Leader report aggregating GitHub PR velocity, Jira sprint health, H2 OKR alignment, and blockers across PAR, DEA, NEV, and MARTECH teams |
| [team-deployment-frequency](skills/team-deployment-frequency/SKILL.md) | Per-team merged PRs and pooled deployment frequency (DORA-style deploy counts) across all repos a GitHub team touches, plus a weekly PR/deploy correlation scatter chart with Pearson r per team |
| [incident-postmortem-summary](skills/incident-postmortem-summary/SKILL.md) | Summarizes Datadog incidents under Santosh's teams — root cause, lessons learned, and Jira-enriched action items per incident, plus a cross-incident executive summary |

## Plugins

Some skills are also packaged as installable plugins under `plugins/`, listed in `.claude-plugin/marketplace.json` (`weekly-eng-report`, `incident-postmortem-summary`).
