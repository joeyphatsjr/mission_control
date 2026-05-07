# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`space-explorer/index.html` — a self-contained space launch tracker called **Mission Control**. No build step, no dependencies to install. Open the file directly in a browser.

```
open space-explorer/index.html
```

## Architecture

The entire app is a single HTML file (~912 lines) with embedded CSS and JavaScript. No bundler, no framework, no npm.

**Four tabs / sections** (tab switching via `showTab()`):
- **Missions** — upcoming + recent launch cards with live countdowns
- **Timeline** — filterable table grouped by month
- **3D Globe** — interactive globe with launch sites, satellites, ISS live tracking
- **Companies** — static company cards (data hardcoded in JS)

**External libraries (CDN only):**
- `globe.gl@2.27.2` — 3D globe rendering

**External APIs:**
- `https://fdo.rocketlaunch.live/json/launches/next/15` and `/past/15` — launch data (cached in `sessionStorage` for 5 min)
- `https://api.wheretheiss.at/v1/satellites/25544` — ISS live position (polled every 5 s)

**Key functions:**
- `loadData()` — fetches and renders all launch data on page load
- `normalizeRL(launch, isPast)` — normalizes the RocketLaunch.live API shape into a common object
- `buildCard(launch, showCountdown)` — returns mission card HTML
- `buildTl(items)` — returns timeline HTML grouped by month
- `initGlobe()` — lazily initializes the globe on first tab visit
- `tickAll()` — countdown ticker, runs every 1 s via `setInterval`
- `orbPos(...)` — simple Keplerian orbital position math for satellite animation
- `vehicleImg(name, launchId)` — returns a Flickr CDN URL for the rocket vehicle; matches by substring against `VEHICLE_IMGS`

**CSS design tokens** are defined as CSS variables on `:root` (dark space theme, cyan accent `#22d3ee`). Font families: Oxanium (`--fd`), JetBrains Mono (`--fm`), DM Sans (`--fu`).

## Image CDN constraint

All rocket images must use `live.staticflickr.com` URLs. Wikimedia (`upload.wikimedia.org`) and NASA CDN (`images-assets.nasa.gov`) are blocked when the app is opened via `file://`. All `<img>` tags use `loading="eager"` — `loading="lazy"` is broken on `file://` because `IntersectionObserver` never fires. The `onerror` handler always sets `this.onerror=null` first to prevent infinite fallback loops.
