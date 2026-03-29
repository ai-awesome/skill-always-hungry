# Scouting Run 2026-03-29 — AutoTrader

## Stats
- Repos scanned: 18
- Repos with new content: 8
- Candidates found: 5
- Candidates passed: 3
- Candidates applied: 3

## Applied

### VaR/CVaR tail-risk metrics + Monte Carlo drawdown simulation
- Source: https://github.com/ranaroussi/quantstats
- Files changed: trader/risk_metrics.py
- Key insight: Historical VaR/CVaR at 95% confidence from daily returns, plus Monte Carlo shuffled-return simulation for drawdown distribution (p5/p50/p95) and bust probability

### Choppiness index for dual-confirmation regime detection
- Source: https://github.com/brndnmtthws/thetagang
- Files changed: trader/market_regime.py
- Key insight: CI = 100 * log10(sum_ATR / range) / log10(N). Values >61.8 = choppy. Used as dual-confirmation with existing ADX — both must agree for trending classification

### Half-Kelly criterion ceiling for position sizing
- Source: https://github.com/ranaroussi/quantstats
- Files changed: trader/position_sizing.py
- Key insight: Kelly = win_rate - (1-win_rate)/payoff_ratio. Half-Kelly applied as a ceiling on ATR-based position size when trade statistics are available

## Skipped

### Configurable post-open delay / pre-close cutoff
- Source: https://github.com/brndnmtthws/thetagang
- Reason: Redundant — AutoTrader already has configurable TRADING_BLACKOUT_WINDOWS and EOD timing constants

## Skipped Repos
- freqtrade/freqtrade — crypto-focused, edge analysis patterns too specific to crypto markets
- kernc/backtesting.py — backtesting library, AutoTrader already has custom backtester
- polakowo/vectorbt — vectorized backtesting, already have walk-forward
- je-suis-tm/quant-trading — educational, basic strategy implementations
- blankly-finance/blankly — generic multi-exchange framework, crypto-focused
- zvtvz/zvt — Chinese stock focused, different architecture
- OpenBB-finance/OpenBB — data platform, not directly applicable patterns
- microsoft/qlib — AI quant platform, too complex/different architecture
