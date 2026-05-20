# Trail-Fix — AGENTS.md

> Canonical operating context for any agent working on Trail-Fix. Brain surface at `~/Brain/01-Projects/trail-fix/AGENTS.md`. Vanilla JS, no framework, single-HTML-file. Hosted at adavenport.dev/trailfix.

## What this is

TrailFix is a browser-based GPS post-processing tool. It takes GPX/NMEA tracks from a Bad Elf Flex Mini Standard GNSS receiver and improves accuracy using position averaging, Kalman smoothing, and NOAA CORS differential corrections.

**The entire app is a single HTML file** at `src/index.html`. No framework, no build step, no npm. All JS is inline. External dependencies are loaded via CDN:
- pako (gzip) from cdnjs.cloudflare.com
- MapLibre GL JS from unpkg.com (optional, loaded on demand)
- Mapbox GL JS from api.mapbox.com (optional, loaded on demand)

## Architecture decisions

- **Single file by design** — the app should remain a single HTML file that anyone can download and open. Don't split into separate JS/CSS files unless there's a compelling reason.
- **No server required** — all processing happens client-side in the browser. The only network requests are optional: fetching CORS data from NOAA's AWS S3 bucket, and loading map tiles.
- **Vanilla JS** — no React, no framework. Keep it simple. The app is ~1700 lines and manageable.
- **Dark theme with Outfit + JetBrains Mono fonts** — this is the visual identity, don't change it.

## Key globals and state

- `rawPoints[]` — accumulated from all loaded files, each point has {lat, lon, ele, time, type}
- `smoothedPoints[]` — after processing (smoothing + averaging)
- `removedPoints[]` — points removed by speed filter
- `loadedFiles[]` — metadata for each imported file {name, pointCount, points[]}
- `units` — 'imperial' or 'metric', affects all display formatting
- `corsObsData` — parsed RINEX data from a CORS station
- `baseFileName` — first loaded file's name, used for export filenames

## Processing pipeline

1. Parse GPX/NMEA → rawPoints
2. Speed filter → removes spikes above threshold
3. Stop detection + averaging → clusters of nearby points become single averaged point
4. Kalman smooth → reduces jitter on moving segments
5. (Optional) CORS correction → applies differential corrections from NOAA reference station
6. Export → KML/GPX/GeoJSON with original filename + `_trailfix` suffix

## CORS stations

Hardcoded list of ~18 NOAA CORS stations in WA/OR/ID/BC in the `CORS_STATIONS` array. The direct download URL pattern is:
```
https://noaa-cors-pds.s3.amazonaws.com/rinex/{year}/{doy}/{station_lowercase}/{station}{doy}0.{yy}o.gz
```

## Deployment

The live version is hosted at adavenport.dev/trailfix. The site appears to be a static site. To deploy, copy `src/index.html` to the appropriate static directory for the site framework.

## Testing

No test framework yet. To manually test:
1. Open `src/index.html` in a browser
2. Load a GPX file (any GPX with `<trkpt>` elements works)
3. Process with default settings
4. Verify: stats appear, canvas map renders, table populates
5. Test exports: KML, GPX, GeoJSON should all download with correct filename
6. Test satellite map: "Use free tiles" should load ESRI satellite imagery
7. Test CORS: "Find nearby CORS stations" should show stations sorted by distance
8. Test multi-file: load 2+ GPX files, verify combine toggle appears

## What to work on next

Priority order:
1. Multi-pass track averaging (ride same trail N times, average for better accuracy)
2. Elevation profile visualization
3. GPX timestamp-based analytics (speed, distance, time)
4. PWA support (service worker for offline use)
5. Shareable map links
6. Better RINEX 3.x parser support
