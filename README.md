# Wellcome Timeline (v2.0 — React + TypeScript)

An interactive, density-adaptive historical timeline component originally developed by [The Wellcome Library](https://wellcomecollection.org/) and rewritten from the ground up in **React 18, TypeScript, and Vite**.

- **Live Demo (GitHub Pages):** [https://mattmcgrattan.github.io/timelinetest/](https://mattmcgrattan.github.io/timelinetest/)
- **Architecture & Engine Reference:** [`src/AGENTS.md`](./src/AGENTS.md)
- **License:** [MIT](./MIT-LICENSE.txt)

---

## Overview

The **Wellcome Timeline** visualizes historical events across centuries, decades, and individual years. As users pan and zoom across time, horizontal space expands and events dynamically resolve collisions and appear across a 3-tier vertical stack above an aggregated ruler bar.

This v2.0 rewrite removes all legacy dependencies (`jQuery 1.7.2`, `jQuery UI 1.10.1`, `iScroll v4`, `easyXDM`, `jquery.address`, `moment.js`, and `json2.js`) while preserving the original astronomical Julian Day calendar math, JSON data contracts, dual-level navigation UX, and URL hash deep-linking. The original 2013 ES5/jQuery codebase is archived in [`legacy/`](./legacy).

### Key Features

- **Modern React 18 + TypeScript Stack:** Scoped CSS Modules ([`WellcomeTimeline.module.css`](./src/components/WellcomeTimeline.module.css)) allow direct mounting on any host page without global CSS/JS collisions or cross-domain iframes.
- **Monet's Letters Visual Theme:** Styled with the typography (`Cormorant Garamond Variable` and `Work Sans Variable` variable `.woff2` fonts) and warm archival palette (`#fefbf4` paper, `#f6f3ec` section wash, `#ebe4db` stone panels, and `#19599a` / `#124574` Monet blues) from [Monet's Letters](https://monet-letters-dev.ch.digirati.io/en/people/141).
- **Astronomical Julian Day Math:** Pure TypeScript port of the Press et al. algorithm ([`julianDateConverter.ts`](./src/utils/julianDateConverter.ts)) supporting BCE/CE dates, the absence of Year 0, and the October 1582 Gregorian calendar transition, paired with `date-fns`.
- **Two-Pass Priority Collision Engine & Spillover:** Distributes event cards across 3 vertical stack levels (`eventStackSize = 3`), resolving horizontal overlaps in strict `(Priority, chronological index)` order (eliminating legacy "zombie suppression") and spilling colliding cards in dense decades into open adjacent rows.
- **Multi-Century Dynamic Resolution Bands:** Automatically transitions the bottom ruler between **Centuries**, **Decades**, and **Years** as zoom level increases, with clickable `.tickEvent` availability squares for every event in each interval.
- **Recursive Deep-Linking (`computeZoomUntilVisible`):** Navigating to `#<EventId>` (e.g. `#43515` or `#15452`) automatically steps through zoom levels until the target event card is free of collisions and centers it in the viewport.
- **Smooth Wheel, Trackpad & Kinetic Touch Scrolling:** Uses [`normalize-wheel-es`](https://www.npmjs.com/package/normalize-wheel-es) for cross-browser wheel/trackpad normalization alongside native React Pointer Events and `requestAnimationFrame` inertia decay.

---

## Quick Start

### Prerequisites
- **Node.js** `>= 18` and **npm**

### Installation & Local Development

```bash
# Install dependencies
npm install

# Start the local Vite development server on http://localhost:8003/
npm run dev
```

Open your browser to:
- **Monet Letters Collection (Default, 397 events):** `http://localhost:8003/`
- **Monet Letters Deep-Link Auto-Zoom:** `http://localhost:8003/#43515`
- **History of Genetics Dataset (141 events):** `http://localhost:8003/?dataUri=sample_data_2.js`
- **Genetics Deep-Link Auto-Zoom (Friedrich Miescher, 1871):** `http://localhost:8003/?dataUri=sample_data_2.js#15452`

### Testing, Linting & Building

```bash
# Run the Vitest unit & engine test suite
npm test

# Run ESLint across src/**/*.{ts,tsx}
npm run lint

# Build both the GitHub Pages static site (docs/) and ESM/UMD library bundles (dist/)
npm run build
```

---

## Usage & Embedding

### 1. As a React Component (ESM)

```tsx
import { WellcomeTimeline } from './src/lib';
import './dist/style.css';

export function App() {
  return (
    <div style={{ width: '100%', height: '750px' }}>
      <WellcomeTimeline
        dataUri="monet_letters.json"
        eventId={43515}
        syncUrlHash={true}
      />
    </div>
  );
}
```

### 2. Drop-In HTML Embed (UMD)

Include `dist/style.css` and `dist/wellcome-timeline.umd.cjs` on any page with React 18 loaded. The library automatically discovers all `.timeline` elements and mounts the component using their `data-uri` and `data-eventid` attributes:

```html
<link rel="stylesheet" href="dist/style.css" />

<div
  class="timeline"
  data-uri="monet_letters.json"
  data-eventid="43515"
  style="width: 100%; height: 700px; background-color: #fefbf4"
></div>

<script src="dist/wellcome-timeline.umd.cjs"></script>
```

---

## Included Datasets & JSON Format

| File | Events | Span | Description |
| :--- | :---: | :---: | :--- |
| [`src/monet_letters.json`](./src/monet_letters.json) | 397 | `1864–1926` | **Default dataset.** Claude Monet correspondence enriched with DLCS IIIF square thumbnails (`!100,100` and `!250,250`). |
| [`src/sample_data_2.js`](./src/sample_data_2.js) | 141 | `1859–2012` | Wellcome Library *History of Genetics: from Darwin to the 21st century* (includes UTF-8 BOM). |
| [`src/sample_data.js`](./src/sample_data.js) | 124 | `1859–2003` | Original Wellcome Library *History of Genetics* sample dataset. |

### JSON Schema Example

```json
{
  "Title": "Claude Monet Correspondence",
  "FeatureImagePath": "https://presentation-api.dlc.services/.../full/!250,250/0/default.jpg",
  "Body": "<p>Introductory HTML description displayed when no event is selected...</p>",
  "InitialCaption": "",
  "IsOpenEnded": false,
  "EarliestJulianDay": 2402064,
  "StartDateDisplay": "1864",
  "LastJulianDay": 2424732,
  "EndDateDisplay": "1926-08-04",
  "BackgroundEvents": [],
  "Events": [
    {
      "EventId": 43515,
      "Title": "Claude Monet to Frédéric Bazille (264)",
      "Body": "<p>15th July 1864 - Honfleur</p>",
      "Priority": 1,
      "Year": "1864",
      "StartDisplayYear": "1864",
      "StartDisplay": "15 July 1864",
      "EndDisplayYear": null,
      "EndDisplay": null,
      "JulianDayStart": 2402064,
      "JulianDayEnd": -9223372036854775808,
      "ThumbnailPath": "https://presentation-api.dlc.services/.../full/!100,100/0/default.jpg",
      "FeatureImagePath": "https://presentation-api.dlc.services/.../full/!250,250/0/default.jpg"
    }
  ]
}
```

> **Note on Point-in-Time Events:** Events without an end date use `-9223372036854775808` (minimum 64-bit integer, or any negative value `< 0`) for `JulianDayEnd`.

---

## Deploying to GitHub Pages

Running `npm run build` (or `npm run build:pages`) outputs a self-contained static site into [`docs/`](./docs) with relative asset and font URLs (`base: './'`) and a `docs/.nojekyll` file:

1. **From the `/docs` folder or GitHub Actions:**
   - Enable **Settings → Pages → Deploy from a branch → `/docs`** (or **Source: GitHub Actions** using [`.github/workflows/deploy-pages.yml`](./.github/workflows/deploy-pages.yml)).
2. **Syncing to a standalone GitHub Pages repository (e.g. `timelinetest`):**
   ```bash
   npm run build:pages
   cp -R docs/. /path/to/timelinetest/
   ```
