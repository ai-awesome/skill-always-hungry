# Scouting Run 2026-03-29 (Run 2) — Mikedown

## Stats
- Repos scanned: 25
- Repos with new content: 16
- Candidates found: 3
- Candidates passed: 3
- Candidates applied: 3

## Audit Score
- Baseline: 297/297 tests passing
- Final: 297/297 tests passing

## Applied

### GitHub-style callout/admonition blocks
- Source: https://github.com/blueberrycongee/Lumina-Note
- Files changed: `src/lib/markdown.ts`, `src/index.css`
- Key insight: Post-process rendered HTML to detect blockquotes starting with `[!TYPE]`. Replace with colored `.callout` divs styled by type (note=blue, warning=yellow, tip=green, danger=red, etc.). Supports 15+ types matching GitHub/Obsidian standard.

### Clickable URLs in editor
- Source: https://github.com/unvalley/ephe
- Files changed: `src/lib/urlClickPlugin.ts` (new), `src/components/Editor.tsx`
- Key insight: CodeMirror ViewPlugin that decorates URLs and markdown links with underlines. On Cmd/Ctrl+mousedown, find URL at cursor position via regex and window.open(). Includes hoverTooltip showing modifier key hint.

### Synchronized editor-preview scrolling
- Source: https://github.com/blueberrycongee/Lumina-Note
- Files changed: `src/components/Editor.tsx`
- Key insight: CodeMirror domEventHandler for scroll events. Compute scroll percentage from editor's scrollDOM, apply proportionally to the preview's content-wrap element. Debounced with requestAnimationFrame.

## Skipped Repos
- MarkEdit-app/MarkEdit — Swift/macOS native, CodeMirror patterns embedded in Swift bridge layer
- athasdev/athas — Full IDE (LSP, terminal, Git), too complex for surgical adoption
- fastrepl/char — Meeting note-taking, different domain focus
- realdennis/md2pdf — PDF conversion, Mikedown already has PDF export via print
- pandao/editor.md — jQuery-based, old architecture
- laobubu/HyperMD — CodeMirror 5 (Mikedown uses CM6)
- All other Tauri+React repos — Not editor/markdown related (clipboard, music, chess, etc.)
