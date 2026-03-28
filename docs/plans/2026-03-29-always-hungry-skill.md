# `/always-hungry` Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a generic Claude Code skill that scouts open-source repos for improvements to whatever project you're working in, evaluates them, and applies what works.

**Architecture:** Single SKILL.md file with 3-stage pipeline (Scout → Evaluate → Apply). State stored in the skill repo (`state/`, `runs/`). Project detection via `$PWD/CLAUDE.md`. Profile auto-generation per target project.

**Tech Stack:** Claude Code skill (Markdown), GitHub MCP tools, git.

---

## File Structure

```
skill-always-hungry/
├── SKILL.md                    ← CREATE: full pipeline logic
├── CLAUDE.md                   ← CREATE: dev instructions for the skill
├── README.md                   ← MODIFY: add usage docs
├── state/
│   ├── seen.json               ← CREATE: empty initial state
│   └── profiles/
│       └── .gitignore          ← CREATE: ignore all profiles
└── runs/
    └── .gitignore              ← CREATE: ignore all run data
```

---

### Task 1: Set up repo structure (state dirs, gitignores, seen.json)

**Files:**
- Create: `state/seen.json`
- Create: `state/profiles/.gitignore`
- Create: `runs/.gitignore`

- [ ] **Step 1: Create state directory and empty seen.json**

```bash
cd ~/Engineering/skill-always-hungry
mkdir -p state/profiles runs
```

Write `state/seen.json`:
```json
{}
```

- [ ] **Step 2: Create gitignore for profiles**

Write `state/profiles/.gitignore`:
```
*
!.gitignore
```

- [ ] **Step 3: Create gitignore for runs**

Write `runs/.gitignore`:
```
*
!.gitignore
```

- [ ] **Step 4: Commit**

```bash
git add state/ runs/
git commit -m "feat: add state and runs directories with gitignores"
```

---

### Task 2: Create SKILL.md — frontmatter, argument parsing, and project detection

**Files:**
- Create: `SKILL.md`

- [ ] **Step 1: Write the SKILL.md with frontmatter, argument parsing, and project detection**

Write the following to `SKILL.md`:

````markdown
---
name: always-hungry
description: >
  Scouts the open-source community for improvements to the current project.
  Reads the project's CLAUDE.md to understand what it is, searches GitHub for
  relevant repos, evaluates whether their patterns would help, and applies what works.
  Use when the user says "always-hungry", "go learn", "scout for improvements",
  "find improvements", or wants to discover open-source patterns applicable to
  their project.
---

# Always Hungry

An autonomous improvement scout. Reads your project's CLAUDE.md, searches the open-source
community for relevant patterns, evaluates candidates against your project, and applies what works.

Works on any project — trading systems, web apps, CLI tools, plugins, anything.

## Paths

| Item | Path |
|---|---|
| Skill repo | ~/Engineering/skill-always-hungry |
| State file | ~/Engineering/skill-always-hungry/state/seen.json |
| Profiles | ~/Engineering/skill-always-hungry/state/profiles/ |
| Run logs | ~/Engineering/skill-always-hungry/runs/ |

## GitHub MCP

Use `mcp__github__*` tools for all GitHub API interactions:

| Tool | Purpose |
|---|---|
| `mcp__github__search_repositories` | Search repos by topic, stars, etc. |
| `mcp__github__search_code` | Search code within repos |
| `mcp__github__get_file_contents` | Read files/directories from a repo |
| `mcp__github__list_commits` | Get commit history |

## Argument Parsing

Parse the user's message for these optional arguments:

| Argument | Default | Description |
|----------|---------|-------------|
| `--scout-only` | false | Scout and report only, don't evaluate or apply |
| `--dry-run` | false | Show profile and search plan, don't execute |
| `--profile` | false | Regenerate the project profile |
| `--show-last-run` | false | Display most recent run summary for this project |

## Project Detection

Detect the target project from the current working directory:

1. Read `$PWD/CLAUDE.md` — primary source of project context
2. If not found, walk up parent directories until a `CLAUDE.md` is found (monorepo support)
3. Fall back to `$PWD/README.md` if no CLAUDE.md exists
4. If nothing found, ask the user to describe the project

Extract from the detected file:
- **Project name**: directory name of the project root (e.g., `AutoTrader`)
- **Project root**: absolute path to the directory containing CLAUDE.md

The skill never runs from its own repo — always from inside the target project.
````

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: add SKILL.md with frontmatter, args, and project detection"
```

---

### Task 3: Add profile generation to SKILL.md

**Files:**
- Modify: `SKILL.md`

- [ ] **Step 1: Append profile generation section to SKILL.md**

Append after the Project Detection section:

````markdown

## Profile Generation

On first run in a new project (or when `--profile` is passed), generate a project profile.

### Step 1: Read project docs

Read the project's CLAUDE.md (and README.md if it exists). Extract:
- Project description and purpose
- Tech stack (languages, frameworks, key libraries)
- Architecture (main subsystems and their responsibilities)
- Test command
- Editable paths (where source code lives)

### Step 2: Generate profile

Based on what you learned, generate a profile JSON. Think about:
- What open-source ecosystem is this project part of?
- What GitHub topics and search queries would find repos with relevant patterns?
- What keywords indicate high relevance vs low relevance?
- What question should you ask when triaging a repo for this project?

Write the profile to `~/Engineering/skill-always-hungry/state/profiles/<project-name>.json`:

```json
{
  "name": "<project-name>",
  "path": "<absolute path to project root>",
  "description": "<1-2 sentence project description>",
  "search_queries": [
    "<query 1 — primary domain, stars:>=200 sort:updated>",
    "<query 2 — key subsystem, stars:>=200 sort:updated>",
    "<query 3 — tech stack + domain intersection, stars:>=100 sort:updated>"
  ],
  "relevance_keywords": {
    "high": ["<5-8 keywords from project's core subsystems>"],
    "medium": ["<3-5 keywords from adjacent concerns>"],
    "low": ["<2-3 broad domain keywords>"]
  },
  "triage_question": "Does this repo contain patterns that could improve <project-name>'s <key subsystems>?",
  "test_command": "<detected from CLAUDE.md>",
  "target_paths": ["<source directories from CLAUDE.md>"]
}
```

### Step 3: Dry-run exit

If `--dry-run` was specified, print the profile and stop:

```
## Dry Run — Project Profile for <project-name>

Description: <description>
Search queries:
  1. <query 1>
  2. <query 2>
  3. <query 3>

Relevance keywords:
  High: <list>
  Medium: <list>
  Low: <list>

Triage question: <question>
Test command: <command>
Target paths: <paths>
```

Then stop. Do not proceed to Stage 1.

### Step 4: Profile flag exit

If `--profile` was specified, print "Profile regenerated for <project-name>" and stop.
````

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: add profile generation with dry-run and profile-only modes"
```

---

### Task 4: Add Stage 1 — Scout

**Files:**
- Modify: `SKILL.md`

- [ ] **Step 1: Append Stage 1 to SKILL.md**

Append after Profile Generation:

````markdown

## Stage 1: Scout

**Goal:** Find repos from the open-source community that contain ideas to improve the target project.

### Step 1 — Load or generate profile

Read `~/Engineering/skill-always-hungry/state/profiles/<project-name>.json`.
If it doesn't exist, run Profile Generation (see above) first.

### Step 2 — Fetch repos

Use the GitHub MCP `search_repositories` tool with each query from the profile's `search_queries`:

```
mcp__github__search_repositories(query: "<search_query>", perPage: 20)
```

Run each query. Deduplicate results by `full_name`.

### Step 3 — Filter

From the combined results:
- Remove forks (check `fork` field)
- Remove repos not updated in the last 90 days (check `updated_at` field)
- Keep at most 20 repos total

### Step 4 — Dedup against state

Read `~/Engineering/skill-always-hungry/state/seen.json`. For each repo, fetch its latest commit SHA:

```
mcp__github__list_commits(owner: "...", repo: "...", perPage: 1)
```

Take the SHA from the first (most recent) commit. Skip any repo where the SHA matches what's in seen.json (no new commits since last scan).

### Step 5 — Triage

For each new/updated repo, fetch its key files using GitHub MCP tools:

```
mcp__github__get_file_contents(owner: "...", repo: "...", path: "")
mcp__github__get_file_contents(owner: "...", repo: "...", path: "README.md")
```

Also look for architecture docs, config files, and source code relevant to the profile's domain.

For each repo, answer the profile's `triage_question`. Rank relevance using the profile's `relevance_keywords` (high → medium → low).

### Step 6 — Extract candidates

For repos that pass triage, produce improvement candidates. Each candidate:

```json
{
  "source_repo": "owner/repo",
  "source_url": "https://github.com/owner/repo",
  "category": "<detected from project domain>",
  "description": "What the improvement is",
  "target": ["<files in the target project to modify>"],
  "rationale": "Why this would improve the target project",
  "key_insight": "The specific technique or pattern to adopt"
}
```

Extract at most **5 candidates** total across all repos. Prioritize by expected impact.

### Step 7 — Write scout report

Create the run directory:

```bash
mkdir -p ~/Engineering/skill-always-hungry/runs/<project-name>/$(date +%Y-%m-%d)
```

Write `runs/<project-name>/YYYY-MM-DD/scout-report.json`:

```json
{
  "date": "YYYY-MM-DD",
  "project": "<project-name>",
  "repos_scanned": <N>,
  "repos_with_new_content": <N>,
  "candidates": [ ...candidate objects... ]
}
```

If `--scout-only`, display the scout report to the user and stop here.
````

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: add Stage 1 Scout — search, filter, triage, extract candidates"
```

---

### Task 5: Add Stage 2 — Evaluate

**Files:**
- Modify: `SKILL.md`

- [ ] **Step 1: Append Stage 2 to SKILL.md**

Append after Stage 1:

````markdown

## Stage 2: Evaluate

**Goal:** Test each candidate improvement against the target project.

Read the scout report from `runs/<project-name>/YYYY-MM-DD/scout-report.json`.

For each candidate:

### Step 1 — Create evaluation branch

In the target project:

```bash
cd <project-root>
git checkout -b always-hungry/eval-YYYY-MM-DD-N
```

Where N is the candidate index (1, 2, 3...).

### Step 2 — Apply the change

Modify the target project files identified in the candidate's `target` field. Apply the `key_insight` from the candidate.

Rules:
- Read the existing target files first. Understand their structure.
- Make surgical, focused changes — don't rewrite entire files.
- Follow existing code patterns and naming conventions.
- Only modify files within the profile's `target_paths`.
- Do not add new dependencies.

### Step 3 — Run tests

Run the test command from the profile:

```bash
<test_command>
```

**If tests fail:** revert and skip this candidate:
```bash
git checkout main
git branch -D always-hungry/eval-YYYY-MM-DD-N
```
Log as "fail" with reason "tests failed" in eval-results.json. Continue to next candidate.

### Step 4 — Score

Run the `/project-audit` methodology inline against the target project (read `~/.claude/skills/project-audit/SKILL.md` and follow Phases 0–6). Skip user checkpoints — the loop must be autonomous.

**Scoring:**
- Before applying the candidate, use the baseline audit from the scout phase (or run one if not available)
- After applying, run a fresh audit
- Score = (Must Do resolved × 3) + (Should Do resolved × 1)
- **Pass** if score improves AND tests pass
- **Fail** otherwise

### Step 5 — Keep or revert

**Pass:**
1. Commit: `git add -A && git commit -m "always-hungry: <candidate description>"`
2. Stay on the evaluation branch

**Fail:**
1. `git checkout main`
2. `git branch -D always-hungry/eval-YYYY-MM-DD-N`

### Step 6 — Log results

Append to `runs/<project-name>/YYYY-MM-DD/eval-results.json`:

```json
[
  {
    "candidate_index": 1,
    "source_repo": "owner/repo",
    "description": "...",
    "baseline_score": { "overall": <N>, ... },
    "candidate_score": { "overall": <N>, ... },
    "verdict": "pass|fail",
    "diff_summary": "..."
  }
]
```
````

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: add Stage 2 Evaluate — branch, apply, test, score, keep/revert"
```

---

### Task 6: Add Stage 3 — Apply

**Files:**
- Modify: `SKILL.md`

- [ ] **Step 1: Append Stage 3 to SKILL.md**

Append after Stage 2:

````markdown

## Stage 3: Apply

**Goal:** Land passing candidates in the target project permanently.

For each candidate that passed evaluation:

### Step 1 — Merge to main

```bash
cd <project-root>
git checkout main
git merge always-hungry/eval-YYYY-MM-DD-N --no-ff -m "always-hungry: <description>

Source: <source_url>
Score: <baseline_overall> → <candidate_overall>"
```

### Step 2 — Cleanup branch

```bash
git branch -d always-hungry/eval-YYYY-MM-DD-N
```

### Step 3 — Update state

In the skill repo, update `~/Engineering/skill-always-hungry/state/seen.json` with the latest commit SHA for every repo that was scanned in this run (not just ones that produced candidates).

Read current seen.json, merge in new entries, write back.

### Step 4 — Write run summary

Create `runs/<project-name>/YYYY-MM-DD/summary.md`:

```markdown
# Scouting Run YYYY-MM-DD — <project-name>

## Stats
- Repos scanned: <n>
- Repos with new content: <n>
- Candidates found: <n>
- Candidates passed: <n>
- Candidates applied: <n>

## Applied

### <description>
- Source: <source_url>
- Score: <baseline> → <candidate>
- Files changed: <list>
- Key insight: <what was learned>

## Failed

### <description>
- Source: <source_url>
- Reason: <why it failed>

## Skipped Repos
<list repos scanned but no candidates, with brief reason>
```

### Step 5 — Commit state

```bash
cd ~/Engineering/skill-always-hungry
git add state/seen.json
git commit -m "run: <project-name> YYYY-MM-DD — <n> applied, <n> failed"
git push origin main
```

### Step 6 — Display summary

Print the summary to the user.

### Step 7 — Push target

Push the target project to its remote:

```bash
cd <project-root>
git push origin main
```

### Show Last Run

If `--show-last-run` was specified, find the most recent directory in
`runs/<project-name>/` and display its `summary.md`. If no summary exists, show
`scout-report.json` instead. Then stop.
````

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: add Stage 3 Apply — merge, cleanup, state, summary, push"
```

---

### Task 7: Create CLAUDE.md for the skill repo

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: Write CLAUDE.md**

Write `CLAUDE.md`:

```markdown
# skill-always-hungry

A generic Claude Code skill that scouts open-source repos for improvements to any project.

## Development

This repo contains:
- `SKILL.md` — the skill logic (installed via symlink to `~/.claude/skills/always-hungry`)
- `state/seen.json` — global dedup of scanned repos (tracked)
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
```

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: add CLAUDE.md with dev instructions and usage"
```

---

### Task 8: Update README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Write README.md**

Write `README.md`:

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: update README with installation and usage"
```

---

### Task 9: Migrate state from old alwayshungry repo

**Files:**
- Modify: `state/seen.json`

- [ ] **Step 1: Copy seen.json from old repo**

Read `~/Engineering/AIDreamWorks/Agent/alwayshungry/state/seen.json` and write its contents to `~/Engineering/skill-always-hungry/state/seen.json`.

This preserves the dedup state so repos already scanned aren't re-scanned.

- [ ] **Step 2: Commit**

```bash
git add state/seen.json
git commit -m "feat: migrate seen.json from old alwayshungry repo"
```

---

### Task 10: Install symlink and verify skill is discoverable

**Files:**
- No files created/modified

- [ ] **Step 1: Create symlink**

```bash
ln -sf ~/Engineering/skill-always-hungry ~/.claude/skills/always-hungry
```

- [ ] **Step 2: Verify skill appears in list**

Check that `/always-hungry` appears in the available skills list. If a previous `always-hungry` symlink exists, the `-f` flag will overwrite it.

- [ ] **Step 3: Push everything**

```bash
cd ~/Engineering/skill-always-hungry
git push origin main
```

- [ ] **Step 4: Verify with dry-run**

Navigate to AutoTrader and run the skill in dry-run mode:

```bash
cd ~/Engineering/AIDreamWorks/AutoTrader
/always-hungry --dry-run
```

Verify that:
- The skill detects AutoTrader from `$PWD/CLAUDE.md`
- A profile is generated with sensible search queries for a trading project
- The profile is saved to `state/profiles/AutoTrader.json`
- The dry-run output shows the profile and stops
