# Scouting Run 2026-03-29 — AI-Coding-Observability

## Stats
- Repos scanned: 18
- Repos with new content: 10
- Candidates found: 5 (1 already existed)
- Candidates passed: 4
- Candidates applied: 4

## Applied

### Engagement tier scoring
- Source: https://github.com/satomic/copilot-usage-advanced-dashboard
- Files changed: src/claude_analytics/reporter.py
- Key insight: Map efficiency_score to named engagement tiers (Power User, Strong, Productive, Developing, Low) with color-coded ANSI display, giving engineers an instant read on their AI workflow maturity.

### Streak tracking
- Source: https://github.com/muety/wakapi
- Files changed: src/claude_analytics/reporter.py
- Key insight: Compute consecutive-day coding streaks from activity block dates and display current/longest streaks in the report header — a motivating engagement metric.

### ASCII contribution heatmap
- Source: https://github.com/yihong0618/GitHubPoster
- Files changed: src/claude_analytics/reporter.py
- Key insight: Render a 7-row (Mon-Sun) x N-week ASCII heatmap using Unicode block characters at varying intensities, giving a visual overview of daily AI coding patterns.

### Week-over-week trend indicators
- Source: https://github.com/microsoft/copilot-metrics-dashboard
- Files changed: src/claude_analytics/reporter.py
- Key insight: Split the reporting period in half, compare totals, and display trend arrows (up/down/flat) next to total active time to show usage momentum.

## Skipped

### Top-project leaderboard (candidate 5)
- Source: copilot-usage-advanced-dashboard
- Reason: Already implemented as "Top Projects by Active Time" section

## Skipped Repos
- openlit/openlit — LLM API instrumentation, not session log analysis
- raga-ai-hub/RagaAI-Catalyst — Enterprise AI observability platform, too heavy/different domain
- syncora-ai/Synthetic-AI-Developer-Productivity-Dataset — Dataset only, no applicable code patterns
- rz1989s/claude-code-statusline — Shell-based statusline, different purpose
- HarmonicSecurity/claudit-sec — Security audit tool, not analytics
- spotify/XCMetrics — Xcode-specific build metrics, different stack
- anmol098/waka-readme-stats — GitHub readme stats, no applicable patterns beyond what wakapi offered
