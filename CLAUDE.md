# skill-always-hungry

A generic Claude Code skill that scouts open-source repos for improvements to any project.

## Development

This repo contains:
- `SKILL.md` — the skill logic (installed via symlink to `~/.claude/skills/always-hungry`)
- `state/seen.json` — global dedup of scanned repos (gitignored, user-private)
- `state/profiles/` — per-project profiles (gitignored, machine-specific)
- `runs/` — per-project run data (gitignored, user-private)

## Installation

```bash
ln -s /path/to/skill-always-hungry ~/.claude/skills/always-hungry
```

## Usage

```bash
cd ~/your-project
/always-hungry                  # full pipeline
/always-hungry --scout-only     # scout only
/always-hungry --dry-run        # show profile
/always-hungry --profile        # regenerate profile
/always-hungry --show-last-run  # show last run summary
```

## Conventions

- Use `mcp__github__*` tools for all GitHub API interactions
- Never hardcode project-specific logic in SKILL.md — everything comes from the profile
- Profiles are auto-generated from the target project's CLAUDE.md
- Run data and profiles are never committed to git (user-private)
