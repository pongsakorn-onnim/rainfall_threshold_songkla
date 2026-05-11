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
- Metadata: `df_meta` — station coordinates and attributes
- Flood extents: daily shapefiles in `D:\HII\OneDrive\HydroDataSci\Data\flood\` — filter by `PV_IDN == 90` for Songkhla
- Data directory: `D:\HII\OneDrive\HydroDataSci\Project\เกณฑ์ฝน จ.สงขลา\Data`
- Output directory: `D:\HII\rainfall_threshold_songkla\output`

## Key cleaning decisions
- Drop phantom rows: rows where ALL rainfall columns are NaN (not a signal, not usable data)
- Replace sentinel value `999999.0` with NaN (found in `rainfall10m` and `rainfall1h`, HII agency stations)
- Stations with `dominant_pct < 70` excluded from completeness heatmap (cadence too mixed for expected-count formula)

## Notebooks
1. `01_exploratory_data_analysis.ipynb` — station QC, completeness heatmap, station selection
2. `02_flood_events.ipynb` — enumerate Songkhla flood event dates from shapefiles (planned)
3. Later: spatial join, I-D extraction, threshold fitting

## Conventions
- Use `df` (not `df_cleaned`) — overwrite after cleaning steps are confirmed
- Station identity column: `tele_station_oldcode`
- Completeness metric: lifetime % (average over months from station's first record, not full calendar window)
- Resolution groups: A ≤15 min, B 16–90 min, C ≥91 min
