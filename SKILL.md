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
