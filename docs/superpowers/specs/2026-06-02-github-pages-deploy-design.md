# Deploy Dyeing Tool to GitHub Pages

**Date:** 2026-06-02

## Goal

Publish the existing `index.html` dyeing process tool to a public URL via GitHub Pages so teammates can access it from a browser without any local setup.

## Architecture

The tool is a single static HTML file with no build step or server-side dependencies. Google Sheets integration is handled via a hardcoded Apps Script URL that posts to the owner's spreadsheet. All users share the same destination sheet.

GitHub Pages serves the `index.html` directly from the `main` branch root — no configuration files needed.

Final URL: `https://<github-username>.github.io/dyeing-tool/`

## Components

| Component | Description |
|---|---|
| GitHub repo `dyeing-tool` | Public repo; hosts the static file |
| GitHub Pages | Free static hosting; auto-deploys on push to `main` |
| `index.html` | Unchanged — already named correctly for Pages |
| Google Apps Script (existing) | Hardcoded URL; accepts POST from all users into owner's Sheet |

## Data Flow

1. User opens `https://<github-username>.github.io/dyeing-tool/`
2. Browser loads `index.html` — all logic runs client-side
3. User fills in dyeing stages, clicks "上傳至 Google Sheets"
4. Browser sends POST to the existing Apps Script URL
5. Apps Script writes data + chart image into owner's Google Sheet

## What Changes

- Nothing in `index.html` — no code edits required
- Local `.git` remote origin is pointed at the new GitHub repo
- GitHub Pages is enabled (Source: `main` branch, root `/`)

## Steps

1. Create new public GitHub repo named `dyeing-tool` (via GitHub web UI or `gh` CLI)
2. Add remote origin: `git remote add origin https://github.com/<username>/dyeing-tool.git`
3. Push: `git push -u origin master` (local branch is `master` per `.git/HEAD`)
4. Enable Pages: GitHub repo → Settings → Pages → Source: Deploy from branch → `master` → `/ (root)` → Save
5. Wait ~1 minute, then visit the published URL

## Out of Scope

- Custom domain
- Per-user Google Sheets (all users share one sheet by design)
- Any backend or authentication
