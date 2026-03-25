# How CORS Post-Processing Works in TrailFix

## The problem

Your Bad Elf Flex Mini Standard gives you ~1.5m accuracy using SBAS (WAAS) corrections from geostationary satellites. The main error sources are:

- **Ionospheric delay** — charged particles in the upper atmosphere slow GPS signals by varying amounts (~0.5-5m)
- **Tropospheric delay** — water vapor and pressure in the lower atmosphere (~0.2-0.5m)
- **Satellite orbit/clock errors** — imperfect knowledge of exactly where each satellite is (~0.1-0.5m)
- **Multipath** — signals bouncing off surfaces before reaching your antenna (~0.5-2m)

SBAS (via WAAS) corrects some of these, but only with broad regional models. The corrections are the same whether you're in Seattle or Spokane.

## How CORS makes it better

A CORS (Continuously Operating Reference Station) is a survey-grade GNSS receiver bolted to a concrete monument at a precisely known location. It records the same satellite signals you do, at the same time. Because it knows its exact position, it can calculate exactly how much error each satellite signal contains at that moment, at that location.

The key insight: **ionospheric and tropospheric errors are spatially correlated**. If the CORS station 15km away sees 2m of ionospheric delay on GPS satellite #15 at 2:34pm, your receiver probably saw ~1.8-2.1m of delay on the same satellite at the same time. The closer you are to the base station, the more similar your errors are.

## What TrailFix does with CORS data

TrailFix performs **code-phase DGPS** (Differential GPS), not full RTK:

1. **Downloads RINEX observation data** from the nearest NOAA CORS station for the same time period as your ride
2. **Parses the station's known position** from the RINEX header (ECEF coordinates, converted to lat/lon)
3. **Calculates baseline distance** between you and the station
4. **Models the spatially-correlated error** that both receivers experienced
5. **Applies a distance-scaled correction** to each of your track points

### Why not full RTK?

Full RTK (Real-Time Kinematic) uses **carrier-phase** measurements — counting the actual wave cycles of the GPS signal, which gives millimeter-level observable precision. Your Standard firmware doesn't expose carrier-phase data; it only outputs NMEA position sentences (which are already code-phase solutions). So we're limited to code-phase differential corrections.

### Accuracy vs. baseline distance

The correction quality degrades linearly with distance because the atmospheric conditions become less correlated:

| Baseline | Correction factor | Expected accuracy |
|----------|-------------------|-------------------|
| 0-20 km  | ~60-80%           | 0.3-0.7m          |
| 20-50 km | ~40-60%           | 0.5-1.0m          |
| 50-100 km| ~20-40%           | 0.7-1.2m          |
| >100 km  | <20%              | Minimal benefit   |

For Bellevue, WA — the SEAT (Seattle) CORS station is ~15km away, putting you right in the sweet spot.

## NOAA CORS data access

### RINEX file naming convention

NOAA distributes CORS data as RINEX (Receiver INdependent EXchange) files:

```
{ssss}{ddd}0.{yy}o.gz
  │     │  │   │  │
  │     │  │   │  └── gzip compressed
  │     │  │   └───── observation file (.o)
  │     │  └───────── year (2-digit)
  │     └──────────── session (0 = full day)
  └────────────────── day of year (001-366)
  station ID (4 char)

Example: cpxf0800.26o.gz
         cpxf = station ID
         080  = day 80 of year (March 21)
         0    = full day session
         26   = year 2026
         o    = observation file
         .gz  = gzip compressed
```

### Data sources

- **AWS Open Data** (fastest): `https://noaa-cors-pds.s3.amazonaws.com/rinex/{yyyy}/{ddd}/{ssss}/{filename}`
- **NOAA direct**: `https://geodesy.noaa.gov/corsdata/rinex/{yyyy}/{ddd}/{ssss}/{filename}`
- **UFCORS** (user-friendly form): `https://geodesy.noaa.gov/UFCORS/` — lets you pick station, date, duration and generates a custom file

TrailFix tries AWS first (direct fetch), falls back to manual upload.

## Limitations

- **Code-phase only** — can't achieve RTK-level (cm) accuracy without carrier-phase data
- **Post-processing only** — corrections are applied after the fact, not in real-time
- **Requires internet** — to download CORS data (but processing is all client-side)
- **Single-base DGPS** — uses one reference station, not a network solution (VRS)
- **No multipath correction** — multipath is local to your antenna and can't be corrected from a remote station
