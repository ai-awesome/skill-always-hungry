# Scouting Run 2026-03-29 — FeatureCrew

## Stats
- Repos scanned: 16
- Repos with new content: 16
- Candidates found: 5
- Candidates passed: 5
- Candidates applied: 5

## Audit Score
- Baseline: 82
- Final: 87

## Applied

### Add cross-model adversarial review to evaluator
- Source: https://github.com/dsifry/metaswarm
- Files changed: plugin/agents/evaluator.md
- Key insight: Writer should always be reviewed by a different model to eliminate shared systematic biases. Added recommended Sonnet/Opus pairings.

### Add post-ship knowledge extraction phase
- Source: https://github.com/dsifry/metaswarm
- Files changed: plugin/skills/feature-crew/SKILL.md
- Key insight: After features ship, extract recurring issues, effective strategies, spec ambiguities, and anti-patterns into a persistent knowledge.jsonl file. Future implementations prime from relevant past entries.

### Add plan verification step before implementation
- Source: https://github.com/gsd-build/get-shit-done
- Files changed: plugin/skills/feature-crew-implement/SKILL.md
- Key insight: Before dispatching implementer agents, verify every acceptance criterion is covered by at least one plan task. Catches the common failure mode where a plan omits an AC and every implementation attempt fails the same eval check.

### Add diagnose command for stuck features
- Source: https://github.com/gsd-build/get-shit-done
- Files changed: plugin/skills/feature-crew/SKILL.md
- Key insight: `/feature-crew diagnose {name}` provides forensic analysis of blocked features — score trends, root cause classification (persistent dimension failure, spec ambiguity, strategy exhaustion, time budget exceeded, git anomaly), and recommended next action.

### Add bug fix ordering to QA agent output
- Source: https://github.com/attilaszasz/sdd-pilot
- Files changed: plugin/agents/qa-agent.md
- Key insight: When multiple bugs are found, analyze dependencies between them and recommend optimal fix sequence. Group bugs with shared root causes. Prevents wasted effort from fixing symptoms before root causes.

## Skipped Repos
- affaan-m/everything-claude-code — broad optimization collection, no single pattern specific enough to apply surgically
- code-yeongyu/oh-my-openagent — agent harness for different architecture (TUI-based), patterns not directly transferable
- sengac/fspec — Gherkin/BDD-style specs not compatible with FeatureCrew's markdown spec approach
- fstandhartinger/ralph-wiggum — bash-loop outer controller approach already superseded by FeatureCrew's strategy ladder
- hesreallyhim/awesome-claude-code — curated list (reference only), no patterns to apply
- ComposioHQ/awesome-claude-skills — curated list (reference only)
- Gentleman-Programming/agent-teams-lite — archived repo
- Other repos — too broad or not relevant to FeatureCrew's specific agent orchestration domain
