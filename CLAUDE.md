# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**High/Mark** — a self-contained, client-facing performance marketing presentation and dashboard, part of the broader **AIcraft** ecosystem. Anton Markhaichuk's tool for ad account audits and presale client presentations.

This repository consists of a **single compiled file**: `HighMark.html` (≈4.6 MB). There is no separate source tree, build system, or package manager. The HTML file IS the deliverable.

Parent ecosystem context: see `../Dashboard/CLAUDE.md`.

---

## Architecture: Custom Single-File Bundler

`HighMark.html` uses a custom in-browser bundler. The file structure:

```
HighMark.html
├── <head>  — loading UI, thumbnail SVG (High/Mark wordmark)
├── <script>  — bundler runtime (unpacks assets, replays scripts in order)
├── <script type="__bundler/manifest">  — JSON: UUID → { data: base64, compressed: bool, mimeType }
└── <script type="__bundler/template">  — JSON-escaped HTML of the actual React app
```

**Boot sequence** (executed on `DOMContentLoaded`):
1. Parse manifest → decode base64, decompress gzip (via `DecompressionStream`) → create `blob:` URLs
2. Parse template HTML → substitute UUID references with blob URLs
3. Re-create all `<script>` tags as live DOM elements, awaiting `onload` for external src scripts to preserve execution order: **React → ReactDOM → Babel → user JSX**
4. `text/babel` / `text/jsx` scripts with a `src` are fetched and inlined before Babel transforms them (required for `file://` origin compatibility)
5. `window.Babel.transformScriptTags()` triggered manually (DOMContentLoaded already fired before the template swap)

---

## Editing Workflow

Because there is no source directory, all edits happen inside `HighMark.html`:

- **JSX/React source** lives JSON-encoded inside the `__bundler/template` script block (line ~178, single very long line).
- **Assets** (images, fonts) live JSON-encoded inside the `__bundler/manifest` script block (line ~170).
- To read the embedded template, pipe through a JSON parser: the raw string is a full HTML document containing `<script type="text/babel">` tags with the app's JSX.
- Changes to the app logic require editing the JSON-escaped template content in place — no build step.

**To view/extract the embedded source** (PowerShell):
```powershell
$html = Get-Content "HighMark.html" -Raw
# Extract template block content
$match = [regex]::Match($html, '<script type="__bundler/template">(.*?)</script>', 'Singleline')
$template = $match.Groups[1].Value | ConvertFrom-Json
$template | Out-File "template_extracted.html" -Encoding utf8
```

---

## No Build or Test Commands

There are no build, lint, or test commands — open `HighMark.html` directly in a browser. Works from `file://` without a local server.

To preview changes: reload the file in Chrome/Edge. The bundler runtime handles all asset unpacking client-side.

---

## Relationship to AIcraft

| Project | Purpose | Status |
|---|---|---|
| `../Dashboard` | Agentic analytics hub (multi-source, AI alerts) | Early planning |
| `../High Mark` | Self-contained client presentation / audit report | Active deliverable |
| `../MA` | Related project | — |
