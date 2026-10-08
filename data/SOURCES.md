# Data sources (`data/`)

All boundary and place files come from **Natural Earth 1:10m, v5.x** (public domain,
naturalearthdata.com; downloaded from github.com/nvkelso/natural-earth-vector). Coordinates are
rounded to 1e-5 deg (about 1 m).

| file | from | processing | used for |
|---|---|---|---|
| `india_boundary_official.geojson` | `ne_10m_admin_0_countries_ind` (India point-of-view edition) | India feature, full resolution | default "over India" test |
| `india_boundary_defacto.geojson` | `ne_10m_admin_0_countries` (default edition, line of control) | India feature, full resolution | boundary-sensitivity check |
| `india_states.geojson` | `ne_10m_admin_1_states_provinces` | 36 Indian states/UTs, simplified to 0.002 deg (~200 m); drawn on the de facto line | state-by-state path |
| `kashmir_defacto_admin.geojson` | `ne_10m_admin_0_countries` | Pakistan, China and Siachen Glacier clipped to 72–81.5 E, 31–38 N | labelling ground India claims but does not administer |
| `region_countries.geojson` | `ne_10m_admin_0_countries_ind` | neighbours clipped to 40–120 E, 15 S–50 N, simplified to 0.01 deg | map background |
| `india_cities.csv` | `ne_10m_populated_places_simple` | the 214 Indian places; state taken from the admin-1 polygons, because the source's own state field is wrong for Gujarat and uses "Orissa" | towns near the track |

Element sets are not stored here: notebook `01_fetch_tle.ipynb` writes them to `tle/`
(from CelesTrak, or from a Space-Track file you downloaded). Space-Track data may be
republished with attribution.
