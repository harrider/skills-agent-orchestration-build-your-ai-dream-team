# Project Pulse — Implementation Plan

## Overview

Project Pulse is a lightweight, static, single-page web dashboard that gives delivery leads and stakeholders an at-a-glance view of project health across a portfolio. It loads a bundled JSON file (`app/project-data.json`) and renders one card per project showing status, owner, progress, priority, and last-updated timestamp. The target user is a PM or delivery lead who wants a fast, no-backend read-out of "what's green, what's at risk, what's blocked" without logging into a heavier PM tool. It is intentionally client-only (HTML + CSS + a small inline `<script>`), runs locally via VS Code, and is designed for easy hand-off between Designer and Coder specialists.

## File assignments

| File | Owner | Purpose |
|---|---|---|
| `app/index.html` | Coder (with class-name contract from Designer) | Semantic markup and structure of the dashboard: page landmarks (`header`, `main`, `footer`), a projects grid container, a project-card pattern, and an inline `<script>` that fetches and renders `project-data.json`. |
| `app/styles.css` | Designer | Full visual system: CSS custom properties (design tokens), layout grid, project card styling, status/priority indicator styles, typography, spacing scale, focus/hover states, and responsive breakpoints. |
| `app/project-data.json` | Coder | Sample static dataset consumed by the dashboard. Contains an array of project objects with a stable, documented shape. |
| `.vscode/launch.json` | Coder | Local run/debug configuration to launch `app/index.html` in a browser (Edge/Chrome debug config, or a static-server variant so `fetch('./project-data.json')` resolves over `http(s)://`). |

## Designer responsibilities

- **Visual system / design tokens** as CSS custom properties in `:root` inside `app/styles.css`:
  - Color palette: neutral surface/background, text primary/secondary, brand accent, and semantic status colors (on-track / at-risk / off-track / blocked / complete) plus priority accents (low / medium / high / critical). All chosen for WCAG AA contrast.
  - Typography scale: font family stack, base size, line-height, and a small type ramp (display, heading, body, caption).
  - Spacing scale: step-based spacing tokens used for card padding, gaps, and layout gutters.
  - Radius, elevation/shadow, and border tokens for cards and badges.
- **Layout**: responsive CSS grid card layout, header with dashboard title and optional summary metrics area, and a projects grid that reflows across mobile / tablet / desktop breakpoints.
- **Component styling**:
  - `.dashboard` shell and `.dashboard-header` region.
  - `.project-card` (surface, padding, radius, elevation, hover/focus).
  - `.status-badge` with modifier classes per status (`.status-badge--on-track`, `--at-risk`, `--off-track`, `--blocked`, `--complete`).
  - `.priority-indicator` with modifier classes per priority.
  - `.progress-bar` (track + fill) with an accessible label pattern.
  - Optional summary metrics tiles (counts by status).
- **Accessibility**:
  - Verified color contrast for all status/priority combinations.
  - Visible keyboard focus styles on all interactive elements.
  - Non-color-only signals for status (icon glyph or text label alongside color).
  - Respect `prefers-reduced-motion` for transitions.
- **Deliverable to Coder**: a locked **class-name contract** so the markup matches the stylesheet. Designer owns `app/styles.css` end-to-end and contributes semantic class-name / structural guidance for `app/index.html` but does not write the markup or JS.

## Coder responsibilities

- **`app/index.html`**:
  - Valid HTML5 with `<html lang>`, `<meta charset>`, `<meta name="viewport">`, and a descriptive `<title>`.
  - Semantic landmarks: `<header>`, `<main>`, optional `<footer>`; heading hierarchy starting at `<h1>`.
  - Projects grid container using the Designer's agreed class names, plus a project-card pattern (card → header with title + status badge → owner → progress bar with accessible label → priority indicator → last-updated timestamp).
  - Link to `styles.css`.
  - Inline `<script>` that fetches `./project-data.json`, iterates projects, renders one card per project using the class-name contract, formats `lastUpdated` for display, sets progress width and `aria-valuenow`, and handles empty and error states.
- **`app/project-data.json`** — proposed data shape:
  - Top-level object with `generatedAt` (ISO timestamp) and `projects` (array).
  - Each project: `id` (string), `name` (string), `owner` (string), `status` (`on-track` | `at-risk` | `off-track` | `blocked` | `complete`), `priority` (`low` | `medium` | `high` | `critical`), `progress` (integer 0–100), `lastUpdated` (ISO 8601), `summary` (string, optional).
  - Include ~5–8 sample projects covering every status and priority value.
- **`.vscode/launch.json`**:
  - Launch config that opens `app/index.html` locally. Recommended: Chrome/Edge debug config with `cwd` set to `${workspaceFolder}/app`, OR a config paired with a static-server extension so `fetch('./project-data.json')` resolves over `http(s)://` rather than `file://`.
  - Coder documents which launch style is used and ensures the data fetch works under that mode.

## Dependencies

| # | Depends on | Blocked task |
|---|---|---|
| 1 | — | Lock **JSON data shape** (Coder proposes; Orchestrator approves). |
| 2 | — | Lock **class-name contract & structural pattern** (Designer proposes; Orchestrator approves). |
| 3 | 1 | Coder authors `app/project-data.json` sample data. |
| 4 | 2 | Designer authors `app/styles.css` against the agreed class names. |
| 5 | 1 + 2 | Coder authors final `app/index.html` markup + inline render script. |
| 6 | 5 | Coder authors `.vscode/launch.json` — served path / launch mode must match how `index.html` loads `project-data.json`. |
| 7 | 3 + 4 + 5 + 6 | Integration validation (see Validation expectations). |

## Parallel work decisions

- **Parallel (non-overlapping files):**
  - Once Dependencies 1 and 2 are locked, Designer on `app/styles.css` runs in parallel with Coder on `app/project-data.json`. Different files, different specialists.
  - `.vscode/launch.json` scaffolding can be drafted in parallel with `styles.css`, but final `url`/`file`/`cwd` values must be reconciled after `index.html` is finalized.
- **Sequential (shared contract or shared file):**
  - Data-shape lock (Dep 1) and class-name contract lock (Dep 2) must happen **before** `app/index.html` is finalized, because the markup encodes both contracts.
  - `app/index.html` is authored **after** both contracts are locked to avoid markup and script rework.
  - `.vscode/launch.json` is finalized **after** `app/index.html`'s served path and fetch strategy are settled, so the debug target resolves the JSON fetch.
  - Integration validation runs **last**, after all four runtime files exist together.

## Validation expectations

- `app/index.html` opens via the `.vscode/launch.json` target and renders with **no console errors or warnings**.
- `app/project-data.json` parses as valid JSON and is successfully fetched at runtime (200 response, correct MIME).
- For **every** project in the JSON, the DOM shows: name, owner, status badge, priority indicator, progress bar with correct percentage, and formatted `lastUpdated`. Empty and error states render when forced.
- `app/styles.css` is applied on first paint (no unstyled flash, no 404 on the stylesheet), and every status and priority modifier class has visible, distinct styling.
- `.vscode/launch.json` launches the dashboard locally with a single "Run" action and the fetch of `project-data.json` succeeds under the chosen launch mode.
- **Accessibility**:
  - Semantic landmarks present (`header`, `main`); single `<h1>`; logical heading order.
  - Keyboard focus visible on all focusable elements; sensible tab order.
  - Status not conveyed by color alone (text label or icon accompanies color).
  - Automated pass (Lighthouse or axe) reports no critical a11y violations and AA contrast passes on status/priority styles.
- **Responsive check**: layout is usable and visually intact at ~360px, ~768px, and ~1280px widths (cards reflow, no horizontal scroll, no clipped text).

## Open questions

1. **Launch strategy for `.vscode/launch.json`**: static-server (Live Server / `serve`) vs. Edge/Chrome `file://` launch? Recommend static server; needs confirmation.
2. **Summary metrics in the header**: should the dashboard show aggregate status tiles in addition to the card grid?
3. **Sorting / filtering controls**: MVP is read-only render — confirm no interactive filters in v1.
4. **Theming**: light-only, or include a `prefers-color-scheme: dark` variant via tokens?
5. **Icon set**: inline SVG (Coder) vs. CSS-only glyphs (Designer) for status/priority?
6. **Data volume**: is 5–8 sample projects sufficient, or seed ~20 to stress the responsive grid?
