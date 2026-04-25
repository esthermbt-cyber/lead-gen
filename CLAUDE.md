# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file lead generation web app (`index.html`) that calls the Apify REST API directly from the browser. No build step, no dependencies, no server required — open the file in any browser.

## Architecture

Everything lives in `index.html` in three logical sections:

1. **`<style>`** — CSS custom properties (design tokens in `:root`) drive the dark theme. All colours, spacing, and radii reference these variables. UI components (`.card`, `.tag`, `.pill`, `.mode-btn`, etc.) are self-contained.

2. **`<body>` (HTML)** — Four stacked cards: settings panel (hidden by default), search form, progress card, results card. Visibility is toggled via `.visible` / `.open` classes added/removed in JS — no DOM nodes are created or destroyed for show/hide.

3. **`<script>`** — Vanilla JS, no frameworks. Key sections:
   - **State**: `currentMode` (`'industry'` | `'company'`), `leads[]`, `sortState`, and `tags` object (`industry`, `role`, `companyRole` arrays).
   - **Token**: read from / written to `localStorage` under key `apify_token`.
   - **Tag input**: `makeTagInput()` wires up a wrapper+input pair to a `tags[key]` array; `renderTags()` re-renders chips into the DOM. Remove handlers are attached as `window.removeTag_{key}` so inline `onclick` in injected HTML can reach them.
   - **Apify flow**: `startRun()` → `pollRun()` (5 s interval) → `fetchDataset()`. All three are plain `fetch` calls with the token in the query string.
   - **Field normalisation**: `normaliseLead()` maps actor-specific field names from both actors into a single `{ name, jobTitle, company, email, phone, linkedin }` shape before rendering.
   - **Render / sort**: `renderResults()` builds `innerHTML` from the normalised leads array. Sorting re-calls `renderResults()` with a sorted copy; the canonical `leads[]` array stays unsorted.
   - **CSV export**: builds a string in memory and triggers a download via `URL.createObjectURL`.

## Apify Actors

| Mode | Actor ID | Key inputs |
|------|----------|------------|
| By Industry | `code_crafter/leads-finder` | `jobTitle[]`, `industry[]`, `location`, `fetchCount` |
| By Company | `harvestapi/linkedin-company-employees` | `companyName`, `jobTitles[]`, `maxResults` |

API base: `https://api.apify.com/v2` — token passed as `?token=` query parameter on every request.

## Adding a New Actor / Data Source

1. Add a new mode button in the `.mode-toggle` div and corresponding `.form-section` fields.
2. Extend `setMode()` to show/hide the new section IDs.
3. Add a new branch in the `submit` handler to build `actorId` + `input`.
4. Extend `normaliseLead()` with any new field names the actor returns.

## Styling Conventions

- All colours via CSS variables — never hardcode hex values outside `:root`.
- Show/hide via class toggle (`.visible`, `.open`, `.active`) — never `style.display` except for the token status confirmation flash.
- `.form-section.active { display: contents }` — the active class on a grid child uses `contents` so grid layout flows through it.
