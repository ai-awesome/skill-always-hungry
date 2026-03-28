# Design: Generic `/always-hungry` Skill

**Date:** 2026-03-29
**Repo:** `git@github.com:ai-awesome/skill-always-hungry.git`
**Origin:** Evolved from `alwayshungry` (Feature_Crew-specific coach at `Agent/alwayshungry`)
**Inspiration:** [karpathy/autoresearch](https://github.com/karpathy/autoresearch) loop pattern

## Summary

A Claude Code skill that scouts the open-source community for improvements applicable to whatever project you're currently working in. It reads the project's CLAUDE.md to understand what the project is, auto-generates search queries, finds relevant repos, evaluates whether their patterns would improve the project, and applies what works. Generic — works on any project.

## Architecture

```
git@github.com:ai-awesome/skill-always-hungry.git
├── SKILL.md              ← pipeline logic (Claude Code skill)
├── CLAUDE.md             ← dev instructions for the skill itself
├── README.md
├── state/
│   ├── seen.json         ← global dedup (repo names + SHAs, tracked in git)
│   └── profiles/
│       └── .gitignore    ← profiles contain local paths, not tracked
└── runs/
    └── .gitignore        ← run data is user-private, not tracked
```

**Installation:**
```bash
ln -s /path/to/skill-always-hungry ~/.claude/skills/always-hungry
```

**Invocation:**
```bash
cd ~/Engineering/AIDreamWorks/AutoTrader
/always-hungry                  # full pipeline
/always-hungry --scout-only     # scout and report only
/always-hungry --dry-run        # show profile and search plan
/always-hungry --profile        # regenerate project profile
```

## How It Detects the Target Project

1. Read `$PWD/CLAUDE.md` — primary source of project context
2. If not found, walk up parent dirs until a `CLAUDE.md` is found (monorepo support)
3. Fall back to `$PWD/README.md` if no CLAUDE.md exists
4. If nothing found, ask the user to describe the project

The skill never runs from its own repo — always from inside the target project.

## Project Profile Auto-Generation

On first run in a new project, the skill reads the project's docs and generates a profile at `state/profiles/<project-name>.json`:

```json
{
  "name": "autotrader",
  "path": "~/Engineering/AIDreamWorks/AutoTrader",
  "description": "Autonomous trading agent with Tiger Brokers, Python, FastAPI",
  "search_queries": [
    "algorithmic trading python stars:>=200 sort:updated",
    "backtesting framework stars:>=300 sort:updated",
    "trading agent autonomous stars:>=100 sort:updated"
  ],
  "relevance_keywords": {
    "high": ["backtest", "trading", "risk management", "position sizing", "stop loss", "portfolio"],
    "medium": ["time series", "market data", "signal", "strategy"],
    "low": ["finance", "stock", "api"]
  },
  "triage_question": "Does this repo contain patterns that could improve AutoTrader's backtesting, risk management, execution, or learning loop?",
  "test_command": "cd trader && python3 -m pytest tests/ -v",
  "target_paths": ["trader/"]
}
```

**Profile generation process:**
1. Read CLAUDE.md — extract project description, tech stack, test commands, architecture
2. Identify the project's domain (trading, web app, CLI tool, plugin, etc.)
3. Generate 3-5 GitHub search queries targeting that domain's open-source ecosystem
4. Generate relevance keywords ranked high/medium/low based on the project's subsystems
5. Generate a triage question specific to the project's improvement areas
6. Extract test command and editable paths from CLAUDE.md

The profile is saved and reused on subsequent runs. `--profile` regenerates it.

## Pipeline

### Stage 1: Scout

**Goal:** Find repos from the open-source community that contain ideas to improve the target project.

**Step 1 — Load or generate profile:**

Read `state/profiles/<project-name>.json`. If it doesn't exist, generate it (see above).

**Step 2 — Fetch repos:**

Use the GitHub MCP `search_repositories` tool with each query from the profile's `search_queries`. Deduplicate results by full_name.

**Step 3 — Filter:**

- Remove forks
- Remove repos not updated in the last 90 days
- Keep at most 20 repos total

**Step 4 — Dedup against state:**

Read `state/seen.json`. For each repo, fetch its latest commit SHA. Skip repos where the SHA matches (no new commits since last scan).

**Step 5 — Triage:**

For each new/updated repo, fetch README.md, CLAUDE.md, and key files. Answer the profile's `triage_question` for each repo. Rank relevance using the profile's `relevance_keywords`.

**Step 6 — Extract candidates:**

For repos that pass triage, produce up to 5 improvement candidates:

```json
{
  "source_repo": "owner/repo",
  "source_url": "https://github.com/owner/repo",
  "category": "detected from project domain",
  "description": "What the improvement is",
  "target": ["files in the target project to modify"],
  "rationale": "Why this would improve the target project",
  "key_insight": "The specific technique or pattern to adopt"
}
```

**Step 7 — Write scout report:**

Save to `runs/<project-name>/YYYY-MM-DD/scout-report.json`.

If `--scout-only`, display the report and stop.

### Stage 2: Evaluate

**Goal:** Test each candidate against the target project.

For each candidate:

1. **Create branch:** `git checkout -b always-hungry/eval-YYYY-MM-DD-N` in the target project
2. **Apply the change:** Modify the target files with the candidate's key_insight
3. **Run tests:** Execute the profile's `test_command` — hard gate, must pass
4. **Score:** Run `/project-audit` methodology inline (skip user checkpoints). Compare roadmap before/after:
   - Score = (Must Do resolved x 3) + (Should Do resolved x 1)
   - **Pass** if score improves AND tests pass
   - **Fail** otherwise
5. **Keep or revert:**
   - Pass: commit on the branch
   - Fail: `git reset --hard`, delete branch
6. **Log results:** Save to `runs/<project-name>/YYYY-MM-DD/eval-results.json`

### Stage 3: Apply

**Goal:** Land passing candidates in the target project.

For each passing candidate:

1. **Merge:** `git checkout main && git merge always-hungry/eval-YYYY-MM-DD-N --no-ff`
2. **Cleanup:** Remove branch
3. **Update state:** Update `state/seen.json` with latest commit SHAs for all scanned repos
4. **Write summary:** Save `runs/<project-name>/YYYY-MM-DD/summary.md`
5. **Commit state:** In the skill repo, commit `state/seen.json` and push
6. **Display summary** to the user
7. **Push target:** Push the target project to its remote (respects project's CLAUDE.md git workflow conventions)

## State Management

### `state/seen.json` (tracked in git)

Global dedup across all projects. Contains repo full_name → latest commit SHA:

```json
{
  "owner/repo": "sha...",
  ...
}
```

A repo scanned for AutoTrader won't be re-scanned for Feature_Crew (the SHA hasn't changed). Different projects may find different candidates from the same repo — but the repo itself is only fetched once.

### `state/profiles/<name>.json` (gitignored)

Per-project profiles containing local paths. Auto-generated, user-editable. Not tracked because paths are machine-specific.

### `runs/<project-name>/YYYY-MM-DD/` (gitignored)

Per-project run data. Contains scout reports, eval results, summaries. User-private — may contain details about the user's codebase.

## Migration from Old alwayshungry

The old `Agent/alwayshungry` repo:
- `CLAUDE.md` pipeline logic → replaced by `SKILL.md` in the new repo
- `state/seen.json` → copy to new repo's `state/seen.json`
- `runs/` → stays in old repo as historical data (or copy to `runs/feature-crew/`)
- `benchmark/` → no longer needed (evaluation uses `/project-audit` scoring instead)
- Skills (`AH-go-learn`, `AH-scout-only`, `AH-show-last-run`) → replaced by `/always-hungry` and its flags

The old repo can be archived after migration.

## Skill Commands

| Command | Action |
|---------|--------|
| `/always-hungry` | Full pipeline: scout → evaluate → apply |
| `/always-hungry --scout-only` | Scout and report, don't implement |
| `/always-hungry --dry-run` | Show profile and search plan, don't execute |
| `/always-hungry --profile` | Regenerate the project profile |
| `/always-hungry --show-last-run` | Display most recent run summary for this project |

## What This Does NOT Do

- Does not run from its own repo — always from inside the target project
- Does not push to remote without user seeing the summary first
- Does not track run data or profiles in git (user-private)
- Does not modify files outside the profile's `target_paths`
- Does not add new dependencies to the target project
- Does not tackle changes that require L-sized refactors
