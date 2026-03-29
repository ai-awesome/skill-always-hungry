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

Resolve the skill repo location dynamically:

```bash
SKILL_ROOT=$(readlink ~/.claude/skills/always-hungry || echo ~/.claude/skills/always-hungry)
```

| Item | Path |
|---|---|
| Skill repo | `$SKILL_ROOT` |
| State file | `$SKILL_ROOT/state/seen.json` |
| Profiles | `$SKILL_ROOT/state/profiles/` |
| Run logs | `$SKILL_ROOT/runs/` |

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

Write the profile to `$SKILL_ROOT/state/profiles/<project-name>.json`:

```json
{
  "name": "<project-name>",
  "path": "<absolute path to project root>",
  "description": "<1-2 sentence project description>",
  "github_topics": ["<2-3 GitHub topics most relevant to this project>"],
  "awesome_lists": ["<1-2 awesome-list repos relevant to this project, e.g. sindresorhus/awesome-nodejs>"],
  "search_queries": [
    "<query 1 — primary domain, stars:>=200 sort:stars>",
    "<query 2 — key subsystem, stars:>=100 sort:stars>"
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

---

## Stage 1: Scout

**Goal:** Find repos from the open-source community that contain ideas to improve the target project.

### Step 1 — Load or generate profile

Read `$SKILL_ROOT/state/profiles/<project-name>.json`.
If it doesn't exist, run Profile Generation (see above) first.

### Step 2 — Fetch repos (prioritized sourcing)

Source repos in priority order. Stop early once you have **20 repos** after filtering. Higher tiers yield higher-quality candidates.

**Tier 1 — Top repos by GitHub topic (highest signal)**

Search by the profile's `github_topics`, sorted by stars descending:

```
mcp__github__search_repositories(query: "topic:<topic> stars:>=500 sort:stars", perPage: 10)
```

These are the most-starred repos in the project's domain — proven, high-quality code.

**Tier 2 — Awesome lists (curated)**

For each repo in the profile's `awesome_lists`, fetch its README:

```
mcp__github__get_file_contents(owner: "...", repo: "...", path: "README.md")
```

Parse the README for repo links (GitHub URLs). Extract up to 10 repos from each list. These are community-curated and pre-vetted.

**Tier 3 — Keyword search (broad discovery)**

Only if Tiers 1-2 yielded fewer than 20 repos. Use the profile's `search_queries`:

```
mcp__github__search_repositories(query: "<search_query>", perPage: 10)
```

Deduplicate all results across tiers by `full_name`.

### Step 3 — Filter

From the combined results:
- Remove forks (check `fork` field)
- Remove repos not updated in the last 90 days (check `updated_at` field)
- Keep at most 20 repos total, preferring higher tiers

### Step 4 — Dedup against state

Read `$SKILL_ROOT/state/seen.json`. For each repo, compare its `updated_at` field from the search results against the timestamp stored in seen.json. Skip any repo where `updated_at` hasn't changed since the last scan.

No additional API calls needed — `updated_at` is already in the search results.

### Step 5 — Triage

For each new/updated repo, fetch only its README:

```
mcp__github__get_file_contents(owner: "...", repo: "...", path: "README.md")
```

Use the README plus the repo's description and topics from the search results to answer the profile's `triage_question`. Rank relevance using the profile's `relevance_keywords` (high → medium → low).

Do NOT fetch additional files at this stage — deeper reading happens only in Stage 2 for candidates that pass triage.

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
mkdir -p $SKILL_ROOT/runs/<project-name>/$(date +%Y-%m-%d)
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

---

## Stage 2: Evaluate

**Goal:** Test each candidate improvement against the target project.

Read the scout report from `runs/<project-name>/YYYY-MM-DD/scout-report.json`.

### Step 1 — Baseline audit

Run one audit before applying any candidates. Fetch the audit methodology:

```
mcp__github__get_file_contents(owner: "ai-awesome", repo: "skill-audit-project", path: "SKILL.md")
```

Run Phases 0–6 inline against the target project. Skip user checkpoints — the loop must be autonomous. Save the baseline result (overall score, Must Do count, Should Do count).

### Step 2 — Apply and test each candidate

For each candidate:

1. **Branch:**
   ```bash
   cd <project-root>
   git checkout -b always-hungry/eval-YYYY-MM-DD-N
   ```

2. **Apply:** Modify target files with the candidate's `key_insight`.
   - Read the existing target files first. Understand their structure.
   - Make surgical, focused changes — don't rewrite entire files.
   - Follow existing code patterns and naming conventions.
   - Only modify files within the profile's `target_paths`.
   - Do not add new dependencies.

3. **Test:** Run `<test_command>` from the profile.
   - **If tests fail:** revert and skip:
     ```bash
     git checkout main
     git branch -D always-hungry/eval-YYYY-MM-DD-N
     ```
     Log as "fail" with reason "tests failed". Continue to next candidate.

   - **If tests pass:** commit and keep:
     ```bash
     git add -A && git commit -m "always-hungry: <candidate description>"
     ```

### Step 3 — Final audit and scoring

After all candidates have been applied (or failed), run one final audit using the same methodology from Step 1.

Score = (Must Do resolved × 3) + (Should Do resolved × 1)

Compare baseline vs final:
- If score improved → all passing candidates are **accepted**
- If score worsened or unchanged → revert all candidates (reset to main)

### Step 4 — Log results

Write `runs/<project-name>/YYYY-MM-DD/eval-results.json`:

```json
{
  "baseline_score": { "overall": <N>, "must_do": <N>, "should_do": <N> },
  "final_score": { "overall": <N>, "must_do": <N>, "should_do": <N> },
  "candidates": [
    {
      "candidate_index": 1,
      "source_repo": "owner/repo",
      "description": "...",
      "verdict": "pass|fail",
      "reason": "..."
    }
  ]
}
```

---

## Stage 3: Apply

**Goal:** Land passing candidates in the target project permanently.

For each candidate that passed evaluation:

### Step 1 — Merge to main

For each passing candidate branch:

```bash
cd <project-root>
git checkout main
git merge always-hungry/eval-YYYY-MM-DD-N --no-ff -m "always-hungry: <description>

Source: <source_url>"
```

### Step 2 — Cleanup branches

```bash
git branch -d always-hungry/eval-YYYY-MM-DD-N
```

### Step 3 — Update state

In the skill repo, update `$SKILL_ROOT/state/seen.json` with the `updated_at` timestamp for every repo that was scanned in this run (not just ones that produced candidates).

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

## Audit Score
- Baseline: <baseline_overall>
- Final: <final_overall>

## Applied

### <description>
- Source: <source_url>
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
cd $SKILL_ROOT
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
