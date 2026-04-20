# Home Assistant Position Editor

A browser-based visual editor for the `left` / `top` coordinates in a Home Assistant [`picture-elements`](https://www.home-assistant.io/dashboards/picture-elements/) card YAML. Load your floor plan image and your YAML side-by-side, drag each entity icon to where it belongs on the plan, and save the updated YAML — with your original structure, comments, and key order preserved.

The whole app is a **single self-contained HTML file** with zero dependencies and no build step. Open it in a browser and go.

## Features

- **Load any image** (PNG, JPG, GIF, WEBP, SVG) as the background floor plan.
- **Load any `picture-elements` YAML** — the parser extracts each entity's `left`, `top`, `entity`, and `title` and renders an icon for it.
- **Entity-type-aware icons** — `sensor.*` entities show as purple text badges, everything else renders as a teal crosshair (lights / switches / etc.). Elements with no `title:` are highlighted in red so nothing gets shipped unlabelled.
- **Drag to reposition** — or use the **arrow keys** for pixel-precise nudges. All movement is stored as percentages so it survives image resizes.
- **Multi-select** — Ctrl/Cmd+click to build a group, or rubber-band select by dragging over empty canvas. Dragging or arrow-keying any member moves the whole group together.
- **Snap to grid** with a configurable step (0.1% → 10%) and an optional overlay so you can see the grid lines while you work.
- **Live title editing** in the sidebar detail panel — fixes the red "no title" warning the moment you type.
- **Paste area** — paste YAML from the clipboard (e.g. straight out of the Home Assistant UI) to load it without a file picker.
- **Round-trip preservation** — saves only rewrite the `left:`, `top:`, and `title:` lines. Comments, `tap_action` / `hold_action`, `style:` overrides, and everything else come through untouched.
- **Save as file** or **copy to clipboard** — whichever fits your workflow.
- **Session persistence** — your image + YAML file handles and grid settings are remembered via the File System Access API and IndexedDB. One click on **Reopen Last** and you're back where you left off.

## Usage

1. Open [`index.html`](index.html) in a modern Chromium-based browser (Chrome, Edge, Brave, Opera). Because the file is named `index.html`, you can also just open the folder directly (or host it on GitHub Pages / any static server) and the app loads automatically.
2. Click **Load Image** and pick your floor plan.
3. Click **Load YAML** and pick your `picture-elements` YAML (or paste it into the sidebar).
4. Drag icons, tweak titles, hit **Save YAML** (downloads `map_updated.txt`) or **Copy YAML**.
5. Next time you open the file, **Reopen Last** will reload both files after a permission prompt.

### Controls

| Action | Shortcut / gesture |
| --- | --- |
| Select one element | Click icon or sidebar row |
| Add/remove from selection | Ctrl/Cmd + click |
| Box-select | Click + drag on empty canvas (hold Ctrl to add) |
| Nudge selection | Arrow keys (step = grid size when snap is on, else 0.1%) |
| Deselect | Click empty canvas |
| Toggle snap | **⊞ Snap** button |
| Toggle grid lines | **⋮⋮ Lines** button (requires snap on) |
| Reset all edits | **Reset** button |

## Browser support

Designed for browsers with the **File System Access API** — Chromium-family (Chrome, Edge, Brave, Opera). Firefox and Safari will still work via a fallback `<input type=file>` for loading, but **Reopen Last** and persistent file handles are unavailable there.

## Project files

- [`index.html`](index.html) — the app.
- [`PROJECT_INSTRUCTIONS.md`](PROJECT_INSTRUCTIONS.md) — internal context document used when developing with Claude Code.
- [`LICENSE`](LICENSE) — MIT.

## License

MIT © 2026 nightswimmer — see [LICENSE](LICENSE).
