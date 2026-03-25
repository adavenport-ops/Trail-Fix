# TrailFix

**GPS post-processor for Bad Elf Flex Mini Standard** — get sub-meter accuracy without the $1,000 Extreme upgrade.

[**Live App →**](https://adavenport.dev/trailfix)

## What it does

TrailFix is a browser-based GPS post-processing tool built for the Bad Elf Flex Mini Standard receiver. It improves your track accuracy from ~1.5m toward ~0.5m using three techniques:

- **Position averaging** — detects stops and averages clustered readings to reduce scatter
- **Track smoothing** — Kalman filter removes GPS jitter from moving tracks
- **CORS differential correction** — downloads NOAA reference station data and applies code-phase DGPS corrections based on your distance from the nearest base station

### Who it's for

Anyone with a Bad Elf Flex Mini Standard (or any GNSS receiver outputting GPX/NMEA) who wants better accuracy without paying for RTK subscriptions or hardware upgrades. Built for hobby use — trail mapping, dirt bike routes, approximate property boundaries.

## Features

- Import GPX and NMEA files (multiple files, combined or separate)
- Speed filter to remove GPS spikes (configurable threshold)
- Point averaging at detected stops
- Kalman-style track smoothing (adjustable strength)
- CORS correction via NOAA reference stations with .gz auto-decompression
- Interactive satellite map (MapLibre free tiles or Mapbox with token)
- Canvas map with raw vs. processed overlay
- Export to KML (Google Earth), GPX, and GeoJSON
- US Imperial / Metric unit toggle
- Single HTML file — no build step, no server, no dependencies

## Quick start

### Use the hosted version
Go to [adavenport.dev/trailfix](https://adavenport.dev/trailfix) and drop a GPX file.

### Run locally
```bash
# Clone the repo
git clone https://github.com/adavenport-ops/Trail-Fix.git
cd Trail-Fix

# Open in browser — that's it, it's a single HTML file
open src/index.html
# or
python3 -m http.server 8000 -d src
```

### Deploy your own
Copy `src/index.html` anywhere that serves static files. No build step needed.

## CORS correction workflow

1. Load your GPX track
2. Click "Find nearby CORS stations"
3. Click **Download** on the nearest station — fetches the RINEX .obs.gz directly from NOAA's AWS mirror
4. Click **Apply corrections**
5. Export your improved track

Accuracy improvement depends on baseline distance to the CORS station:
| Distance | Expected accuracy |
|----------|------------------|
| < 20 km  | ~0.3–0.7 m       |
| < 50 km  | ~0.5–1.0 m       |
| < 100 km | ~0.7–1.2 m       |

## Project structure

```
Trail-Fix/
├── README.md
├── LICENSE
├── src/
│   └── index.html          # The complete app (single file)
├── docs/
│   ├── how-cors-works.md   # Technical explanation of DGPS corrections
│   └── bad-elf-comparison.md # Standard vs Extreme breakdown
└── assets/
    └── screenshot.png       # For README/social preview
```

## Deployment to adavenport.dev

This app is deployed as a static page at `adavenport.dev/trailfix`. See the deployment notes below depending on your site setup.

### If using GitHub Pages
The `src/index.html` can be served directly, or copy it to your site repo's `public/trailfix/index.html`.

### If using Vercel / Netlify
Add a redirect or copy the file into your site's static directory:
```
# vercel.json example
{
  "rewrites": [
    { "source": "/trailfix", "destination": "/trailfix/index.html" }
  ]
}
```

### If using Hugo / Astro / similar SSG
Copy `src/index.html` to `static/trailfix/index.html` (Hugo) or `public/trailfix/index.html` (Astro). It will be served as-is without being processed by the SSG.

## Tech stack

- Vanilla JS — no framework, no build step
- [pako](https://github.com/nickolai/pako) — gzip decompression for RINEX .gz files
- [MapLibre GL JS](https://maplibre.org/) — free satellite map (optional)
- [Mapbox GL JS](https://www.mapbox.com/mapbox-gljs) — premium satellite map (optional, requires token)
- NOAA CORS data via [AWS Open Data](https://noaa-cors-pds.s3.amazonaws.com/)

## Moving to Claude Code

This project was prototyped in Claude chat. For continued development:

```bash
# Install Claude Code
npm install -g @anthropic-ai/claude-code

# Work on this repo
cd Trail-Fix
claude
```

Planned improvements (good first tasks for Claude Code):
- [ ] Multi-pass track averaging (ride same trail N times, average for better accuracy)
- [ ] Elevation profile chart
- [ ] GPX timestamp-based speed/distance analytics
- [ ] RINEX parser improvements for RINEX 3.x long filenames
- [ ] Progressive Web App (PWA) with offline support
- [ ] Shareable map links (encode track in URL hash)

## License

MIT

## Background

The Bad Elf Flex Mini Standard and Extreme use identical hardware — same GNSS chip, same dual-frequency L1/L5 antenna, same everything. The $1,000 price difference is a firmware unlock that enables RTK and improved SBAS. TrailFix exists because for hobby use (trail mapping, boundary estimation), you can get most of the way there with post-processing and free CORS data.

See [docs/bad-elf-comparison.md](docs/bad-elf-comparison.md) for the full technical breakdown.
