# Songkhla Rainfall Threshold Project

## Project goal
Develop rainfall intensity-duration (I-D) thresholds for flood warning in Songkhla province, following Georganta 2022 methodology.

## Coaching style
- The user is learning to code — guide with conceptual hints and method names, not ready-to-run code
- Only write code when explicitly asked
- Let the user make mistakes and reason through them; correct direction not syntax
- Keep explanations concise — one concept at a time

## Data
- Primary dataset: `df` — multi-resolution rainfall from `20260410_Rainfall-สงขลา.csv` (42 stations after cleaning)
- Secondary dataset: `df_r1h` — hourly rainfall from `20260410_Rainfall1h-สงขลา.csv` (114 stations)
- Metadata: two files — `df_meta1` (older, 2022) and `df_meta2` (2026-05-07, primary for lat/lon qualification; 102/134 Songkhla stations covered)
- Flood extents: daily shapefiles in `D:\HII\OneDrive\HydroDataSci\Data\flood\` — filter by `PV_IDN == 90` for Songkhla
- Data directory: `D:\HII\OneDrive\HydroDataSci\Project\เกณฑ์ฝน จ.สงขลา\Data`
- Output directory: `D:\HII\rainfall_threshold_songkla\output`
- Dataset cutoff date: 2026-04-10 (matches CSV filenames; used as reference for recency criterion)

## Key cleaning decisions
- Drop phantom rows: rows where ALL rainfall columns are NaN (not a signal, not usable data)
- Replace sentinel value `999999.0` with NaN (found in `rainfall10m` and `rainfall1h`, HII agency stations)
- Stations with `dominant_pct < 70` excluded from completeness heatmap (cadence too mixed for expected-count formula) and from the df qualification funnel (`df_resolution_filtered`)

## Notebooks
1. `01_eda_rainfall.ipynb` — station QC, completeness heatmap, station selection, source assignment
2. `02_flood_events.ipynb` — enumerate Songkhla flood event dates from shapefiles (planned)
3. Later: spatial join, I-D extraction, threshold fitting

## Station qualification criteria (applied in this order)
Both `df` and `df_r1h` use the same 4 gates in sequence:
1. **Recency** — avg monthly completeness ≥ 70% over past 12 complete months (excludes partial current month; computed as `pivot.iloc[:,-13:-1].mean(axis=1)`)
2. **Lifetime completeness** — avg monthly completeness ≥ 70% over full record
3. **Span** — record spans ≥ 3 years
4. **Lat/lon** — station present in `df_meta2`

For `df` only, an extra step precedes recency: filter to stations with `dominant_pct ≥ 70%` (consistent reporting interval) using `df_resolution_filtered`.

Qualified sets: `df_summary_qual` and `r1h_summary_qual`

## Station source assignment (`stn_source`)
Final lookup table mapping each qualified station to its primary dataset (`df` or `r1h`):
- All `df_summary_qual` stations → `df`, **except** ONE098
- All `r1h_summary_qual` stations not in `df_summary_qual` → `r1h`
- ONE098 → `r1h` (same span in both datasets; r1h is cleaner)
- KAOP, NPCT, TSDO → `df` (longer usable span despite 60-min interval)

Stored in `stn_source` DataFrame (index = station code, column = `source`).

## Conventions
- Use `df` (not `df_cleaned`) — overwrite after cleaning steps are confirmed
- Station identity column: `tele_station_oldcode`
- Completeness metric: lifetime % (average over months from station's first record, not full calendar window)
- Funnel variable naming: `after_resolution` → `after_recent` → `after_completeness` → `after_span` → `df_summary_qual` (for df); `r1h_after_recent` → `r1h_after_completeness` → `r1h_after_span` → `r1h_summary_qual` (for df_r1h)
