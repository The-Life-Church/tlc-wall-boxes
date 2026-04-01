# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file static web application for HSL Wall Boxes at The Life Church. It provides an interactive floor plan reference tool for broadcast/live event technicians to look up wall box and stage box connections and wiring.

## Development

No build system, package manager, or dependencies. Open `index.html` directly in a browser — that's it.

## Agent Guidance

- Treat `main` as the only active branch. Do not recreate or rely on a `stage` branch unless the user explicitly asks for it.
- This repo deploys automatically to Firebase Hosting from GitHub Actions on pushes to `main`.
- The live public app URL is `https://hsl-wall-boxes.web.app`.
- The Firebase project ID is `tlc-production-apps-2026`.
- The Firebase Hosting site target in `firebase.json` is `hsl-wall-boxes`. Keep workflow config aligned with that site if Hosting settings change.
- The repo remote is `https://github.com/The-Life-Church/HSL-Wall-Boxes.git`. If the repository name changes again, update the local `origin` URL and confirm the GitHub Actions still point at the correct repo.
- Firebase Auth is enabled for Google sign-in and currently restricted in `index.html` to approved `@thelifechurch.com` users. Be careful when editing auth-domain checks because both Firebase Auth config and client-side checks affect access.
- The Firebase web app config is embedded directly in `index.html`. Do not replace it with placeholders unless the user explicitly wants to disconnect the deployed app.
- Keep `.firebase/` and `firebase-debug.log` out of commits. `.gitignore` already excludes them.
- For local verification, a browser open of `index.html` is enough for layout work, but authentication behavior should be validated against the deployed Hosting URL because Google sign-in depends on Firebase Auth configuration and authorized domains.
- If changing Firebase Hosting targets or URLs, update both `firebase.json` and `.github/workflows/firebase-hosting-merge.yml` plus `.github/workflows/firebase-hosting-pull-request.yml` so GitHub Actions keep deploying to the intended site.

## Architecture

The entire application lives in `index.html` (~557 lines). There are no external scripts, frameworks, or stylesheets (only Google Fonts via CDN).

### Core Data Structures

- **`boxes`** — JavaScript object with 22 box entries, each containing `name`, `type`, `label`, and a `sections` array of connection groups (audio, video, data, fiber, RF, etc.)
- **`destinations`** — Object mapping `"boxId||portName"` keys to user-assigned destination strings, persisted in LocalStorage under key `tlc_dest`

### Box Types & Colors

| Type | CSS var | Examples |
|------|---------|---------|
| Wall Box (WB) | `--accent-amber` (#d4b74a) | Stage sides, walls, camera platform |
| Stage Floor Box (FBST) | `--accent-purple` | Microphone/guitar positions |
| Floor Box (FB) | `--accent-green` | Camera monitoring positions |
| FOH | `--accent-red` | Front-of-house booth positions |

### Key UI Components

1. **SVG Floor Plan** — 680×560 canvas with clickable box buttons; clicking selects a box and populates the detail panel
2. **Detail Panel** — Shows port/connection info for the selected box, with color-coded tags per connection type
3. **Accordion/List View** — All boxes expandable, shows assigned vs. total port counts; driven by the same `boxes` data
4. **Search** — Real-time filter across box IDs, names, and port names; highlights matches
5. **Modal Editor** — Edit destination text for any port; saves to LocalStorage and updates `tlc_updated` timestamp

### LocalStorage Keys

- `tlc_dest` — JSON object of user-edited destinations (`"boxId||portName"` → string)
- `tlc_updated` — ISO timestamp of last save
