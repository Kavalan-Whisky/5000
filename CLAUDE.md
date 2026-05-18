# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page demo titled "God Mode 6.0: Omniscient" — a Traditional Chinese (`lang="zh-TW"`) cyberpunk-styled HTML toy that packs 240+ "hidden programs" (visual effects, Web API demos, an audio synth, a particle/neural-network canvas, generated buttons/filters, an infinite-scroll card grid) into one file. There is no application logic beyond browser-side demos.

## Repository layout

- `index.html` — the entire app: inline CSS in `<head>`, inline JS in a single `<script>` at the bottom. **All edits happen here.** There is no `src/`, no modules, no bundler.
- `.github/workflows/jekyll-gh-pages.yml` — Jekyll build + GitHub Pages deploy. Triggers on push to `main` only (plus manual `workflow_dispatch`). No Jekyll config exists, so Jekyll just publishes `index.html` as-is.

There is no `package.json`, no test runner, no linter, no README.

## Running / building

- **Local dev**: open `index.html` directly in a browser, or serve the directory (`python3 -m http.server`) — there is no build step.
- **Deploy**: merge to `main`; the workflow above publishes to GitHub Pages.
- **Browser requirements**: many features use modern/experimental Web APIs (`navigator.getBattery`, `requestMIDIAccess`, `EyeDropper`, `BarcodeDetector`, `navigator.contacts`, `getDisplayMedia`, `requestPictureInPicture`, Web Audio, Gamepad). Each call site guards for support; missing APIs should degrade silently, not throw.

## Code organization inside `index.html`

The single `<script>` is divided by Chinese comment banners into named regions. Preserve these banners when editing — they are the only structure:

- `5.0 舊有動畫庫` — base keyframes + `.eff-*` classes injected into `#dynamic-styles`.
- `[NEW: 151-200] Hyper-Filters` — 50 randomly-parameterized `.hf-1` … `.hf-50` filter classes injected into `#hyper-styles` at load.
- `5.0 核心監控` — HUD widgets (CPU, RAM, network, battery, APM, MIDI, gamepad, clock, uptime, mouse, keys, screen, orientation, scroll, idle).
- `[NEW: 101-150] Web Audio Synth Engine` — `audioCtx` is lazy (`initAudio`); `playRandomTone()` is called from clicks and from generated buttons.
- `5.0 舊有按鈕事件` — handlers for the static buttons declared in `#main-controls` (IDs `f-*` and `n-*`).
- `[NEW: 201-240] Hyper Commands` — a loop appends 40 buttons to `#main-controls`; the first ~7 come from the `hyperCommands` array, the rest are auto-generated "CMD_*" buttons that toggle a random `hf-*` class on a random card.
- `5.0 全域粒子引擎` — click-spawned particles on `#particle-canvas`; the same click handler also triggers `playRandomTone()`.
- `[NEW: 1-100] Neural Network Canvas` — 100 nodes on `#neural-canvas` with O(N²) edge drawing and mouse-gravity. This is the dominant CPU cost; be cautious adding work to its `requestAnimationFrame` loop.
- `Konami Code & 搜尋與卡片渲染` — `renderCards(filter)` rebuilds `#grid-container` from `cardsData`; the scroll listener at the bottom pushes new entries to `cardsData` and re-renders, producing infinite scroll.

## Conventions specific to this file

- `S = id => document.getElementById(id)` is the universal shorthand — use it instead of `document.getElementById`.
- `log(msg)` shows a transient toast **and** appends to the terminal; `termLog(msg)` only appends to the in-page `#terminal`. Pick the right one — UI-visible feedback uses `log`, diagnostic traces use `termLog`.
- Sections marked `未刪減` ("not removed") are intentionally preserved legacy code. Don't refactor or dedupe across the `5.0` ↔ `[NEW]` boundaries; the comment ranges (`1-100`, `101-150`, `151-200`, `201-240`) are the author's mental index for the 240 features.
- All CSS lives in three places: the `<style>` in `<head>` (handwritten), `#dynamic-styles` (animations), and `#hyper-styles` (50 generated filter classes). Generated classes are reseeded on every load — don't rely on a specific `.hf-N` looking the same twice.
- UI strings are Traditional Chinese. Match the existing language when adding buttons or toasts.
- New buttons added to the static control grid need both an `id` in the `#main-controls` HTML and a handler in the `5.0 舊有按鈕事件` block; dynamically-generated buttons go through the `hyperCommands` array or the auto-CMD loop instead.
