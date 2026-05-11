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
| [estimate-effort](skills/estimate-effort/SKILL.md) | Analyzes PRDs and Zoom transcripts to generate a multi-repo implementation plan with RAD analysis |
