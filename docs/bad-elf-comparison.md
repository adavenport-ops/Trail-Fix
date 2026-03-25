# Bad Elf Flex Mini: Standard vs Extreme

## TL;DR

Same hardware. The $1,000 difference is a firmware unlock.

## Hardware (identical)

| Spec | Standard | Extreme |
|------|----------|---------|
| GNSS chip | Same | Same |
| Frequencies | L1/L5 | L1/L5 |
| Constellations | GPS, GLONASS, Galileo, BeiDou, QZSS | GPS, GLONASS, Galileo, BeiDou, QZSS |
| SBAS | WAAS, EGNOS, MSAS | WAAS, EGNOS, MSAS |
| Bluetooth | 4.0 (HS), Class 1.5, 3 connections | 4.0 (HS), Class 1.5, 3 connections |
| USB | USB-C | USB-C |
| Display | 128x128 transflective LCD | 128x128 transflective LCD |
| Accelerometer | 3-axis | 3-axis |
| Battery | 24 hours | 24 hours |
| Waterproof | IPX7 | IPX7 |
| Weight | 143g | 143g |
| Update rate | 1-10 Hz | 1-10 Hz |

## Software differences (firmware unlock)

| Feature | Standard | Extreme |
|---------|----------|---------|
| **SBAS accuracy** | 1.5-2.0m | 1.0m |
| **RTK engine** | Locked | Unlocked (1-2cm) |
| **NTRIP client** | Locked | Unlocked |
| **Point One Navigation** | Not supported | Supported |
| **Multi-frequency processing** | Limited | Full |
| **Price** | ~$499 | ~$1,499 |

## What the Extreme unlock actually enables

1. **Better SBAS processing** — The hardware already receives L1+L5 dual-frequency signals. The Extreme firmware uses both frequencies more aggressively to compute ionospheric corrections, improving standalone SBAS from 1.5m to 1.0m.

2. **RTK positioning engine** — The GNSS chip is physically capable of carrier-phase measurements. The Standard firmware doesn't expose this capability. The Extreme firmware enables it, allowing 1-2cm accuracy when connected to an NTRIP correction source.

3. **Built-in NTRIP client** — The Bad Elf Flex app has an NTRIP client, but it's grayed out on Standard units. The Extreme unlock activates it.

## Alternatives to the $1,000 upgrade

- **Bad Elf Flex Tokens** — $25/day for full Extreme features. Good for occasional survey work.
- **Post-processing with TrailFix** — Free. Uses CORS data to apply DGPS corrections, improving accuracy from ~1.5m toward ~0.5m.
- **Separate RTK receiver** — A u-blox ZED-F9P board (~$200-300) gives full RTK capability independently.
- **Multi-pass averaging** — Ride/walk the same route multiple times and average the tracks.
