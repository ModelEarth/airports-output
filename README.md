# airports-output

Published aviation emissions output for the [ModelEarth/airports](https://github.com/ModelEarth/airports)
project. This repository holds **only generated data** — day-partitioned CO2 estimates attributed to
departure airports — so that maps and other tools can read it straight from the raw CDN URL without
running the processing pipeline.

The producing code lives in `emissions/` of
[ModelEarth/airports](https://github.com/ModelEarth/airports/tree/main/emissions) (its `--publish`
mode writes this layout). No credentials, no backend service, and no raw OpenSky flight exports are
stored here.

## Coverage

Seven UTC departure days of OpenSky-derived flights. "Raw records" is the number of input flight
observations read; "Computed flights" is the number that produced a CO2 estimate after same-airport,
invalid-coordinate, invalid-interval and unmatched-endpoint rows were excluded.

| UTC date | Raw records | Computed flights | Total CO2 (kg) |
| --- | ---: | ---: | ---: |
| 2026-10-03 | 55,797 | 26,957 | 129,551,159 |
| 2026-10-04 | 53,598 | 26,766 | 136,503,060 |
| 2026-10-05 | 59,087 | 29,318 | 146,746,920 |
| 2026-10-06 | 63,059 | 31,007 | 148,336,357 |
| 2026-10-07 | 64,874 | 32,151 | 152,748,691 |
| 2026-10-08 | 65,281 | 32,629 | 157,716,684 |
| 2026-10-09 | 60,842 | 31,579 | 154,658,962 |
| **Total** | **422,538** | **210,407** | **1,026,261,833** |

## Layout

```text
emissions/
  manifest.json
  daily/YYYY-MM-DD/{flights.csv, routes.csv, airports.csv, report.json}
```

- `manifest.json` — `generated_at`, the model settings, `latest` (most recent date), and per date the
  flight count, total CO2 (kg), and the SHA-256 of each file.
- `daily/<date>/flights.csv` — one estimate per accepted flight, with UTC departure date, identifiers,
  distance, fuel and CO2 in kg.
- `daily/<date>/routes.csv` — daily departure/arrival route totals in kg.
- `daily/<date>/airports.csv` — daily departure-attributed totals per directory airport: `FAA`, `ICAO`,
  `Airport`, `Latitude`, `Longitude`, `Departures`, `EstimatedCO2_kg`, `Country`, `State`, `date`.
- `daily/<date>/report.json` — generation time, model settings, and per-day computed flights and CO2.

## Reading the data

Start with the manifest, then fetch the latest day's files:

```text
https://raw.githubusercontent.com/ModelEarth/airports-output/main/emissions/manifest.json
https://raw.githubusercontent.com/ModelEarth/airports-output/main/emissions/daily/<date>/airports.csv
```

For a map, read `manifest.json` to list available dates, pick one, then load that day's `airports.csv`
and use `Latitude`/`Longitude` for markers and `EstimatedCO2_kg` for marker size.

## Method

```text
distance_km = great-circle distance between airport coordinates
fuel_kg     = FIXED_FUEL_KG + FUEL_KG_PER_KM * distance_km
co2_kg      = fuel_kg * CO2_KG_PER_KG_FUEL
```

| Constant | Value |
| --- | ---: |
| `FUEL_KG_PER_KM` | 3.0 |
| `FIXED_FUEL_KG` | 500.0 |
| `CO2_KG_PER_KG_FUEL` | 3.16 |

## Limitations

This preserves the historical aviation project's generic model. It is not an aircraft/engine-specific
inventory, a measured fuel burn, or a live emission rate. It excludes non-CO2 effects and does not
reconstruct actual flown tracks. OpenSky airport assignments are inferred and receiver coverage is
incomplete. Same-airport flights, invalid coordinates, invalid time intervals, and unmatched endpoints
are excluded and reported. Each flight is attributed once to its departure airport; arrivals do not
receive a second copy. Only the departure date is used for grouping, including overnight flights.
Airports with no computed departures have no output row; absence does not mean zero actual emissions.

Follow OpenSky's applicable data-access terms when reusing or redistributing observations.
