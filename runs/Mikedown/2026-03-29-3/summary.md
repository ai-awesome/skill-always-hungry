# Scouting Run 2026-03-29 (Run 3) — Mikedown

## Stats
- Repos scanned: 23
- Repos with new content: 23
- Candidates found: 2
- Candidates passed: 1
- Candidates applied: 1

## Audit Score
- Baseline: 297/297 tests passing
- Final: 297/297 tests passing

## Applied

### Image paste from clipboard
- Source: https://github.com/kyleyoungblom/sidebar-notes
- Files changed: `src/components/Editor.tsx`, `src-tauri/capabilities/default.json`
- Key insight: In the CodeMirror paste handler, detect `image/*` clipboard items. Read as ArrayBuffer, write to `assets/` directory alongside the markdown file via Tauri's `writeFile`. Insert `![](assets/img-timestamp.ext)` at cursor. Required adding `fs:allow-write-file`, `fs:allow-mkdir`, `fs:allow-exists` capabilities.

## Failed

### Enhanced editor comfort (bracket matching, auto-close, indent, selection highlights)
- Source: Standard CodeMirror 6 packages
- Reason: Tests failed — `@codemirror/language`, `@codemirror/autocomplete`, `@codemirror/search` are transitive deps but pnpm strict mode prevents importing them without direct dependency declaration. Would require adding new dependencies.

## Skipped Repos
- purocean/yn — Electron + Monaco + Vue, different stack
- foambubble/foam — VS Code extension, not a standalone editor
- Milkdown/milkdown — ProseMirror WYSIWYG framework, different paradigm
- mdSilo/mdSilo-app — Tauri + ProseMirror, useful patterns but too architecturally different
- mdx-editor/editor — Lexical-based React component, different editor core
- uiwjs/react-md-editor — textarea-based, no CodeMirror
- hengvvang/typoly — Similar image paste pattern, used sidebar-notes' cleaner approach instead
