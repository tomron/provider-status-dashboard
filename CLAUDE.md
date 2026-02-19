# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file static dashboard (`index.html`) that polls live status APIs for 35+ cloud, AI, and developer service providers and renders them in a categorized card grid. There is no build step, no bundler, no package manager, and no backend — everything runs in the browser.

## Development

Open `index.html` directly in a browser. No server required, though a local HTTP server avoids CORS issues when testing API fetches:

```bash
python3 -m http.server 8080
# or
npx serve .
```

There are no lint, test, or build commands.

## Git Workflow

Always create a feature branch and git worktree before making changes — never commit directly to `main`. Merging to `main` triggers the GitHub Actions workflow (`.github/workflows/deploy.yml`) which deploys the site to GitHub Pages automatically.

## Deployment

Pushing to `main` deploys automatically via `.github/workflows/deploy.yml` using GitHub Pages (`actions/deploy-pages`). No manual deploy step is needed.

## Architecture

Everything lives in `index.html` as a single self-contained file with three logical sections:

1. **`<style>`** — All CSS. Uses CSS custom classes (`.card.green/yellow/red/gray`) driven by the status indicator returned from each provider's API. The sticky filter bar and "issues only" toggle are implemented with CSS class toggling on `<body>`.

2. **`<body>`** — Minimal static HTML; the card grid is built entirely at runtime by JavaScript.

3. **`<script>`** — All application logic:
   - **SVG constants** (`SVG_GITHUB`, `SVG_OPENAI`, etc.) — inline path data from Simple Icons (MIT/CC0). `viewBox="0 0 24 24"` for all icons. Monday.com uses a special `SVG_MONDAY_DOTS` array of circle descriptors instead.
   - **`CATEGORIES`** array — defines the five display categories: `cloud`, `llm`, `devops`, `devtools`, `pm`.
   - **`PROVIDERS`** array — each entry has `name`, `category`, `apiUrl`, `statusPage`, `logo`, and an optional `parser` field.
   - **`fetchStatus(provider)`** — fetches each provider's status API. Most providers use the Atlassian Statuspage v2 format (`/api/v2/status.json` → `status.indicator`). Three providers need custom parsers set via the `parser` field:
     - `"gcp"` — parses Google Cloud's `incidents.json` format
     - `"slack"` — parses Slack's `/api/current` format
     - `"aws"` — parses AWS's RSS feed via `DOMParser`
   - **`indicatorToClass(indicator)`** — maps Atlassian indicator strings (`"none"`, `"minor"`, `"major"`, `"critical"`) to CSS classes.
   - **`buildCategories()` / `buildCard()` / `renderCard()`** — DOM construction and updates.
   - **`refreshAll()`** — fires all fetches in parallel via `Promise.allSettled`, updates cards, then updates the "last updated" timestamp. Called on load and every 60 seconds.

## Adding a Provider

1. Add an SVG path constant (from [simpleicons.org](https://simpleicons.org), `viewBox="0 0 24 24"`).
2. Add an entry to `PROVIDERS` with `name`, `category`, `apiUrl`, `statusPage`, and `logo`.
3. If the provider doesn't use Atlassian Statuspage v2, add a `parser` key and handle it in `fetchStatus`.

## URL Query Parameter

`?providers=github,openai` filters the dashboard to show only the listed providers (matched by slugified name). Useful for embedding a focused subset.

## OpenSpec

The `openspec/` directory contains workflow artifacts for structured changes (proposals, tasks, specs). These are managed by the `openspec-*` skills and do not affect the runtime application.
