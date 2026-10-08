# Satellite overflight of India: two notebooks

Given satellite **names**, this project downloads their latest orbital elements (TLEs) and then
answers, for any list of **time windows**:

- Did the satellite pass over India? (yes / no, with the closest approach if no)
- If yes: **when** it entered and left, **which states** and **which towns** it crossed, and the
  **ground track** as files you can plot or animate later.

No account or credentials are needed.

```
01_fetch_tle.ipynb   names  ->  tle/satellites.tle
02_overflight_india.ipynb   tle/satellites.tle + time_windows.txt  ->  outputs/
```

## Quick start

```bash
pip install skyfield shapely pandas matplotlib jupyter
jupyter notebook
```

Python 3.9 or newer.

1. Open **`01_fetch_tle.ipynb`**. Edit `SATELLITE_NAMES` in the first code cell (the default is
   IONOSFERA-M 1 to 4). Run all cells. It writes `tle/satellites.tle` and prints a status table;
   every satellite should say `ok`.
2. Edit **`time_windows.txt`** (format below).
3. Open **`02_overflight_india.ipynb`** and run all cells. Results appear in `outputs/`.

Re-run step 1 whenever you want fresh elements, then step 3.

## Notebook 01: names to TLE file

| setting | meaning |
|---|---|
| `SATELLITE_NAMES` | Names as text (`"IONOSFERA-M 1"`) or NORAD numbers (`61735`). |
| `SOURCE` | `"celestrak"` (default, live download) or `"local"` (read a catalogue file you downloaded). |
| `LOCAL_FILE` | Catalogue path when `SOURCE = "local"`. |
| `OUT_FILE` | Output file, default `tle/satellites.tle`. |

Notes:

- The status table says `ok`, `not found`, `ambiguous` or `fetch failed`. Bad-checksum element sets are
  skipped. Network failures are retried four times.
- If a name is `ambiguous` or `not found`, use the NORAD number instead. It is unique.
- CelesTrak returns **only the latest** element set. For a past date, use `SOURCE = "local"` with a
  catalogue downloaded from Space-Track for that period.

## `time_windows.txt`

Two columns, start and end, separated by comma, tab or semicolon:

```
# start, end  (ISO 8601; no time zone = UTC)
start,end
2026-10-07T00:00:00Z,2026-10-08T00:00:00Z
2026-10-07T05:30:00+05:30,2026-10-07T07:30:00+05:30
```

- No time zone means **UTC**. An offset such as `+05:30` is converted to UTC.
- Lines starting with `#` and a `start,end` header row are ignored.
- Each row is analysed separately; one output folder per row.

## Notebook 02: the analysis

For each window and each satellite, the notebook:

1. Picks the element set closest to the middle of the window. It warns if that is more than
   `MAX_TLE_GAP_DAYS` (3) days away, because accuracy falls as the elements age.
2. Propagates the orbit with SGP4 (Skyfield) and finds the sub-satellite point every 10 s, then every
   1 s near India.
3. Intersects the track with India's boundary to get exact entry and exit times, the states crossed
   and towns within `CITY_RADIUS_KM` (50 km) of the track.
4. For passes that miss India, records the closest approach.
5. Runs three self-checks (`RUN_CHECKS = True`): an independent propagation, an independent crossing
   detector, and the same test on the other boundary edition.

Other settings: `ONLY_NORAD` (restrict to one satellite), `BOUNDARY` (`"official"` or `"defacto"`),
`COARSE_STEP_S`, `FINE_STEP_S`, `OUT_ROOT`.

### What "over India" means

The **sub-satellite point** (the spot on Earth directly below the satellite) lies inside India's land
and island boundary. Territorial waters and the exclusive economic zone are **not** included, and
neither is the satellite being merely *visible* from India. The default boundary is India's official
one; the check repeats the test on the line-of-control edition, and the notebook reports any
difference.

## Outputs

`outputs/summary.csv` lists every window and satellite. Each window gets its own folder
`outputs/<start>_<end>/`:

| file | content |
|---|---|
| `india_overflights.csv` | each pass over India: entry/exit time (UTC and IST), duration, direction, entry and exit point, states crossed, height, local solar time, Sun elevation |
| `india_states.csv` | states and union territories crossed, with times |
| `india_towns.csv` | towns within the corridor, with time and distance to the track |
| `regional_passes.csv` | every pass near the region, including misses, with its closest approach to India |
| `ground_tracks_30s.csv` | sub-satellite latitude, longitude and altitude every 30 s |
| `india_paths.geojson`, `ground_tracks.geojson` | the in-India paths and full tracks, for GIS tools |
| `overflights.czml` | animation file for CesiumJS |
| `map_india_overflights.png`, `timeline.png` | quick-look figures |

To view the animation, load `overflights.czml` in [CesiumJS](https://cesium.com/platform/cesiumjs/)
(`Cesium.CzmlDataSource.load(...)`) or drop it onto Cesium Sandcastle. The satellite turns orange
while over India.

## Required data (`data/`)

The six boundary and place files must be present before running notebook 02:

```
india_boundary_official.geojson
india_boundary_defacto.geojson
india_states.geojson
kashmir_defacto_admin.geojson
region_countries.geojson
india_cities.csv
```

Where they came from and how they were processed is in [`data/SOURCES.md`](data/SOURCES.md).

## Accuracy

- SGP4 with a TLE is typically accurate to a few kilometres at the element-set epoch and degrades by
  roughly a kilometre or more per day. Use windows close to the TLE epoch (a few days).
- A pass that grazes the border can flip between "over" and "not over" within that error. The
  boundary-sensitivity check flags the cases that depend on which border is used.
- The sub-satellite point is on the WGS84 ellipsoid. Earth-orientation details (UT1−UTC is under a
  second) change positions by tens of metres, which is negligible next to the TLE error.
- Notebook 01's live download was tested against a mock server, not the live CelesTrak service.

## Data licences and attribution

- Boundaries and places: [Natural Earth](https://www.naturalearthdata.com/), public domain.
- Orbital elements: [CelesTrak](https://celestrak.org/) (derived from US Space Force data distributed
  by Space-Track). Cite the source if you republish the element sets.
- Boundary lines are cartographic data for analysis; they are not legal or survey-grade borders.

## Layout

```
.
├── 01_fetch_tle.ipynb
├── 02_overflight_india.ipynb
├── time_windows.txt
├── data/            six data files + SOURCES.md
├── tle/             created by notebook 01   (git-ignored)
└── outputs/         created by notebook 02   (git-ignored)
```
