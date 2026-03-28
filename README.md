# always-hungry

A Claude Code skill that scouts the open-source community for improvements to whatever project you're working in.

Inspired by [karpathy/autoresearch](https://github.com/karpathy/autoresearch) — an autonomous loop that finds, evaluates, and applies improvements.

## How it works

1. **Detect** — reads your project's `CLAUDE.md` to understand what it is
2. **Scout** — searches GitHub for repos relevant to your project's domain
3. **Evaluate** — applies candidate improvements, runs tests, scores via project audit
4. **Apply** — keeps what improves the project, reverts what doesn't

## Install

```bash
git clone git@github.com:ai-awesome/skill-always-hungry.git
ln -s /path/to/skill-always-hungry ~/.claude/skills/always-hungry
```

## Usage

```bash
cd ~/your-project
/always-hungry                  # full pipeline: scout → evaluate → apply
/always-hungry --scout-only     # discover what's out there
/always-hungry --dry-run        # preview search plan without executing
/always-hungry --profile        # regenerate project profile
/always-hungry --show-last-run  # show most recent results
```

## Works on any project

The skill auto-generates a search profile from your project's documentation. Examples:

| Project | What it searches for |
|---------|---------------------|
| Trading bot | Backtesting frameworks, risk management, execution engines |
| Claude Code plugin | Agent patterns, evaluation, skill design, workflows |
| Web app | UI patterns, state management, testing, deployment |
| CLI tool | Argument parsing, output formatting, plugin systems |

## License

MIT
