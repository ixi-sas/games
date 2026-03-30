# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is a collection of browser-based HTML games, served as static files. There is no build system, no package manager, and no server-side code. Everything runs directly in the browser.

Current games:
- `public/mundial2026/` — A 12-level puzzle/trivia game themed around Argentina and the 2026 World Cup.

## Structure

```
public/
  index.html              # Landing page (currently empty/minimal)
  mundial2026/
    index.html            # Self-contained single-file game (HTML + CSS + JS)
    caricaturas/          # Transparent PNG caricatures of players (Messi, CR7, Neymar, Mbappé, Haaland, Vinicius)
```

## Architecture of mundial2026

The entire game lives in a single `index.html` file (~1500 lines). Key patterns:

- **Level system**: `const L = [...]` array defines all 12 levels. Each entry has `q` (question HTML), `hint`, `suc`/`sub` (success messages), and `fn` (level function like `lv1`, `lv2`, etc.).
- **State**: Global vars `cur` (current level index), `stars` (accumulated score), `attempts` (object keyed by level index), `done` (completed levels).
- **Drag engine**: `mkDrag(el, cb)` — a custom touch+mouse drag implementation using a fixed-position ghost clone. `cb.move(x,y)` fires during drag; `cb.end(x,y)` must return `true` to accept the drop.
- **Level lifecycle**: `loadLv(n)` calls `cleanupLevel()` (removes floating elements and event listeners) then calls `L[n].fn()`. Level completion goes through `lvComplete(delay)`.
- **Event listener cleanup**: All listeners added via `addLvListener()` so they're tracked in `lvListeners[]` and removed on `cleanupLevel()`. DOM elements added outside `#play` go into `floatingEls[]`.
- **Audio**: Web Audio API synthesized sounds — no external audio files. `sfxOk()`, `sfxNo()`, `sfxGoal()`, `sfxClick()`.
- **Player images**: Referenced via the `I` object (e.g. `I.messi`, `I.cr7C`) mapping short keys to `./caricaturas/*.png` paths.
- **Responsive**: Uses `clamp()` throughout for font sizes and layout. `mob()` helper detects mobile to adjust star display.

## Adding a new level

1. Add a new entry to the `L` array with `q`, `hint`, `suc`, `sub`, and `fn`.
2. Implement the level function (e.g. `function lv13(){...}`) following the same patterns: populate `#play`, use `addLvListener` for events, call `lvComplete()` on success and `wrong()` on failure.
3. Update the final screen if needed (`fin-year`, `CAMPEON` text).

## Deployment

Static files only — just serve the `public/` directory. No compilation needed.
