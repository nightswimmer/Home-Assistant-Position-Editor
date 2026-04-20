# Home Assistant Position Editor — Project Instructions

A **browser-based visual editor** for Home Assistant `picture-elements` card YAML files, built as a **single self-contained HTML file** (no build step, no dependencies, no server).

The user loads a floor-plan / background image alongside a YAML configuration. Each `entity` in the YAML is rendered as a draggable icon overlaid on the image, at the `left`/`top` percentage position from the element's `style:` block. The user repositions elements visually, edits titles, and saves the result back out as YAML — with the **original YAML structure, comments, and key ordering preserved** (only `left`, `top`, and `title` lines are rewritten).

## Stack & constraints

- **Single file**: everything lives in [index.html](index.html) — HTML + inline CSS + inline JS. The file is named `index.html` so hosting providers (GitHub Pages, plain static servers, or just opening the folder in a browser) serve it automatically as the directory index.
- **No build tools, no npm, no frameworks**. Plain vanilla JS, plain CSS.
- **No external runtime dependencies** except Google Fonts (JetBrains Mono + Syne) loaded from a CDN in `<style>`.
- Target: modern Chromium-based browsers (uses the **File System Access API** — `showOpenFilePicker`, `FileSystemHandle.requestPermission`). Falls back to a hidden `<input type=file>` when unavailable.
- Persistence via **IndexedDB** (store: `pos-editor` / `files`) for file handles, filenames, and grid settings. No localStorage.

## File layout

```
/
├── index.html                            ← the entire app
├── PROJECT_INSTRUCTIONS.md               ← this file (context for Claude)
├── README.md                             ← GitHub landing page
├── LICENSE                               ← MIT
└── .claude/                              ← Claude Code workspace settings
```

There is intentionally only one source file. Do **not** split it into separate `.js` / `.css` files unless the user explicitly asks.

## Feature summary

**Loading**
- `Load Image` — opens a file picker for the background image (PNG/JPG/GIF/WEBP/SVG). Handle is persisted to IndexedDB.
- `Load YAML` — opens a file picker for a `.yaml` / `.yml` / `.txt` file. Handle is persisted.
- `Reopen Last` — appears on startup if handles are stored; re-requests permission and reloads both files.
- **Paste area** in the sidebar — pasting YAML text replaces the currently loaded YAML without a file picker (useful for quick iteration from the Home Assistant UI).

**Canvas / element rendering**
- The stage is sized to fit the canvas area preserving the image's aspect ratio; it reflows on window resize.
- Each element is absolutely positioned as a percentage of the stage (`left: X%; top: Y%;` with `translate(-50%, -50%)`).
- **Entity-type-aware icons**:
  - `sensor.*` → purple rectangular text badge showing a placeholder value (`28°C`).
  - everything else (treated as a light) → teal circular crosshair SVG.
- **Missing title** → element is drawn in red (the `.no-title` class) to flag entities that still need a label.

**Selection**
- Click an element or sidebar row → single selection.
- **Ctrl/Cmd + click** on an element or row → toggle it in/out of the current selection group.
- **Rubber-band box select** by clicking empty stage and dragging → elements whose centre falls inside the box are selected. Hold Ctrl to add to the existing group.
- The **reference element** (the last-added to the group) gets a white-ringed highlight; other group members get a dimmer yellow ring. The detail panel always shows the reference element's values.
- Clicking empty stage (without drag) → deselect all.

**Editing**
- **Drag** any selected icon → moves the whole selected group by the same delta (clamped 0–100% per axis).
- **Arrow keys** → nudge the whole group (step = grid size if snap is on, else 0.1%).
- **Detail panel inputs** (Title / Left / Top) — live-update the reference element only.
- Titles are stored trimmed; an empty title triggers the red `.no-title` styling.

**Grid / snap**
- `⊞ Snap` toggles snap-to-grid on/off.
- `⋮⋮ Lines` toggles the overlay grid canvas (only available when snap is on).
- Grid size dropdown: 0.1 / 0.25 / 0.5 / 1 / 2 / 5 / 10 (percent).
- Snap and grid size apply to drags AND arrow-key nudges.
- All three settings persist to IndexedDB.

**Output**
- `Save YAML` → downloads `map_updated.txt` (Blob download).
- `Copy YAML` → writes to the clipboard; button shows `✓ Copied!` for ~1.8s.
- `Reset` → re-parses the original YAML text (discarding all edits) after a confirm dialog.

## YAML parse / serialize design (important)

The app uses a **minimal, line-based, non-general-purpose YAML parser** specifically for the Home Assistant `picture-elements` structure. It is **not** a full YAML implementation.

**Parsing** ([index.html:744](index.html#L744)):
- Scans for the top-level `elements:` key.
- Each `- type: …` starts a new element record.
- Extracts `left`, `top`, `entity`, `title` by prefix match on trimmed lines.
- Everything else in the element is stashed in `_extraLines` but is **not** currently used on serialize — the serializer rewrites the *original* lines in place.
- The elements block is considered ended when a non-indented key (that isn't a list item) appears.

**Serialization** ([index.html:788](index.html#L788)):
- Walks the original YAML line-by-line and **only rewrites the `left:`, `top:`, and `title:` lines** belonging to elements inside the `elements:` block. Indentation is preserved by matching the leading whitespace of the original line.
- `left` / `top` are only rewritten when inside a `style:` sub-block of an element (to avoid touching `top:` fields in unrelated nested structures).
- If an element previously had no `title:` line and one has been added in the UI, a new `title:` line is injected immediately before the `entity:` line, using the same indent.
- If a title was cleared, the line is dropped.
- All other lines (comments, `tap_action`, `hold_action`, `style:` overrides, etc.) are passed through untouched.

**Implication for any changes**: do not introduce a real YAML library without the user's say-so — the whole point of this design is round-trip preservation without reformatting the user's file.

## Working conventions (from CLAUDE.md)

- Ask for clarification when requirements are ambiguous.
- When the user **asks a question** about a bug or feature, answer it — do not start editing. Only change code after the user confirms.
- Flag incomplete areas proactively and suggest improvements.
- Before big refactors / structural changes, remind the user to push to GitHub first.
- On "we're finishing / commit time / closing the chat": update **this file** and **README.md** to reflect the current state, then propose a commit name + description for the user to run themselves.

## Current state (as of 2026-04-20)

- Main app file has been renamed from `Home-Assistant-Position-Editor.html` to `index.html` so the folder can be opened / hosted directly (GitHub Pages, plain static servers, or `file://` on the directory) and resolve to the app automatically.
- All features listed above are implemented and functional.
- No known bugs tracked yet.

## Open ideas / possible next steps

These are *unprompted suggestions* — not commitments. Raise with the user before acting.

- **Undo/redo stack** — currently the only way to revert is the `Reset` button (all-or-nothing).
- **Direct-save via File System Access API** — right now `Save YAML` downloads a new file (`map_updated.txt`) instead of writing back to the opened handle. The handle is already stored, so `createWritable()` would enable true in-place save.
- **Sensor-value preview** — the `28°C` badge is hardcoded; could pull a user-supplied mock value per entity or show the entity ID instead.
- **Keyboard shortcut to delete an element** from the YAML entirely (currently edit-only, no add/remove).
- **Zoom / pan** the stage for dense floor plans.
- **Drag-and-drop** image/YAML onto the drop zone (the UI hints at it but it isn't wired up).
