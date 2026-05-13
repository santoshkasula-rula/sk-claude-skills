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
