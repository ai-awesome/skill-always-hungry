# Scouting Run 2026-03-29 — Mikedown

## Stats
- Repos scanned: 9
- Repos with new content: 9
- Candidates found: 5
- Candidates passed: 4
- Candidates applied: 4

## Audit Score
- Baseline: 297/297 tests passing
- Final: 297/297 tests passing

## Applied

### Table paste conversion (HTML/TSV/CSV to Markdown tables)
- Source: https://github.com/Bokuchi-Editor/bokuchi
- Files changed: `src/lib/tableConverter.ts` (new), `src/components/Editor.tsx`
- Key insight: Detect `text/html` with `<table>` tags in clipboard via DOMParser, and detect tab/comma-separated text. Convert to pipe-delimited Markdown tables on paste in the CodeMirror editor.

### Interactive checkboxes in preview
- Source: https://github.com/Bokuchi-Editor/bokuchi
- Files changed: `src/components/MarkdownRenderer.tsx`
- Key insight: Add click handler on `input[type=checkbox]` in rendered HTML. Find the Nth checkbox in markdown source via regex and toggle `[ ]`/`[x]`.

### Auto-save to file (3s debounce)
- Source: https://github.com/Bokuchi-Editor/bokuchi
- Files changed: `src/store/appStore.ts`
- Key insight: For tabs with a known `filePath`, debounce-write content to disk via `writeTextFile` after 3 seconds of inactivity. Marks tab clean on success. Non-fatal on failure.

### Session tab persistence
- Source: https://github.com/Bokuchi-Editor/bokuchi
- Files changed: `src/store/appStore.ts`, `src/App.tsx`
- Key insight: Persist tab file paths and active tab index to localStorage on every tab change. On app mount, re-read files from disk (for freshness) and restore the tab layout.

## Skipped

### Document outline / TOC navigation
- Source: https://github.com/Bokuchi-Editor/bokuchi
- Reason: Mikedown already has a fully functional `Toc` component with scroll-spy and heading navigation.

## Skipped Repos
- drl990114/MarkFlowy — uses ProseMirror (different editor core), patterns too divergent
- ajkdrag/otterly — Svelte frontend, architectural patterns not directly portable
- Gram-ax/gramax — docs-as-code platform, different scope
- lockedmutex/rhyolite — pure Rust frontend (Freya), different stack
- dvcrn/markright — Clojure, very old codebase
- SimonShiki/Typability — Milkdown/WYSIWYG, different editing paradigm
- opensourcecheemsburgers/ubiquity — Yew (Rust frontend), not applicable
- pheralb/typethings — Tiptap editor, work in progress, limited patterns
