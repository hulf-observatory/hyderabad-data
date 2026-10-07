# Hyderabad Urban Observatory: open spatial data

Open spatial layers for Hyderabad and the HMDA region, published by the
[Hyderabad Urban Observatory](https://hyderabad.urbanobservatory.in) for teaching, research
and public use: administrative boundaries, transport, amenities, housing, water, 2031 land-use
plans, historical maps (1854–1970s), archive imagery (1967–1979), built-up growth 1985–2023,
building heights and elevation.

The data is published in **dated releases** that never change, in cloud-native formats that
open directly in QGIS, GeoLibre, DuckDB, Python and web maps without downloading everything.
Current release: **`2026-10-07`**. The full layer list with licences is in [LAYERS.md](LAYERS.md).

## Where things are

| | URL |
|---|---|
| Catalogue (one JSON, every layer) | https://data.hyderabad.urbanobservatory.in/layers.json |
| STAC catalogue | https://data.hyderabad.urbanobservatory.in/stac/catalog.json |
| Vector layers: GeoParquet + PMTiles | `https://data.hyderabad.urbanobservatory.in/vector/<id>.parquet`, `…/vector/<id>.<hash>.pmtiles` |
| Raster layers: PMTiles | [Releases](https://github.com/hulf-observatory/hyderabad-data/releases) → `<id>.pmtiles` |
| Raster tiles for web maps | `https://hyd-tiles.hulf-observatory.workers.dev/r/<release>/<id>/{z}/{x}/{y}.webp` (TileJSON at `…/<id>.json`) |
| Viewer | https://maps.hyderabad.urbanobservatory.in/ |
| City Timeline | https://timeline.hyderabad.urbanobservatory.in/ |

Vectors and small files live in this repository (served by GitHub Pages); raster files are
attached to the GitHub Release of the same date. Files over 2 GB are split into parts with a
`<id>.mosaic.json` index; the tile server above reads them as one layer.

## Use the data

**QGIS:** Layer → Add Layer → Add Vector Layer → Protocol HTTP(S) → paste a `.parquet` URL.
For rasters, download the `.pmtiles` from the release and open it (QGIS 3.32+ reads PMTiles),
or add the TileJSON URL as an XYZ connection.

**GeoLibre** (geolibre.app): Add Data → URL → paste a `.parquet` or `.pmtiles` URL.

**DuckDB:**
```sql
INSTALL spatial; LOAD spatial;
SELECT * FROM read_parquet('https://data.hyderabad.urbanobservatory.in/vector/ghmc_wards_2026.parquet') LIMIT 5;
```

**Python (GeoPandas):**
```python
import geopandas as gpd
wards = gpd.read_parquet('https://data.hyderabad.urbanobservatory.in/vector/ghmc_wards_2026.parquet')
```

**MapLibre:** vector sources via `pmtiles://<url>` with [pmtiles.js](https://github.com/protomaps/PMTiles);
raster sources with `tiles: ['https://hyd-tiles.hulf-observatory.workers.dev/r/2026-10-07/<id>/{z}/{x}/{y}.webp']`.

## Licences

Each layer keeps the licence of its source, recorded per layer in `layers.json`, the STAC items
and [LAYERS.md](LAYERS.md). Where no licence is recorded yet, the source is named and the data
is published for research and teaching; we are confirming terms with those agencies. The
Observatory's own processing, derived layers and metadata are **CC BY 4.0**. Please cite:

> Hyderabad Urban Observatory (2026). Spatial Data Repository, release 2026-10-07.
> https://github.com/hulf-observatory/hyderabad-data

## How it is made

Sources are cleaned, reprojected and clipped on the Observatory's machines
(a Python pipeline on the Observatory's machines; layout in [SCHEME.md](SCHEME.md)), then
`release.py` assembles a dated release: GeoParquet + PMTiles for vectors, PMTiles for rasters,
a STAC catalogue, and `layers.json` for the apps. Tile serving for rasters is
[hyd-tiles](https://github.com/hulf-observatory/hyd-tiles).

Questions and corrections: open an issue here.
