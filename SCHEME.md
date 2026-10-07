# Open data scheme (2026-10-07)

Where every published file lives and how the apps address it. Written for the viewer,
City Timeline and release script; all three follow this exactly.

## Hosts

| Name | URL | Holds |
|---|---|---|
| **D** (data base) | `https://data.hyderabad.urbanobservatory.in/` | catalogue, vectors, small files (GitHub Pages; CORS `*`, range requests, 100 MB/file) |
| **W** (tile worker) | `https://hyd-tiles.hulf-observatory.workers.dev` | raster tiles read from GitHub Releases of `hulf-observatory/hyderabad-data` |
| Release | `https://github.com/hulf-observatory/hyderabad-data/releases/tag/<release>` | raster PMTiles (≤2 GB each; bigger ones split into a mosaic) |

`<release>` is a date tag, first one `2026-10-07`. A published release never changes.

## Files under D

```
layers.json                    the catalogue (schema below); fetched with cache: no-cache
stac/catalog.json              STAC root; stac/<id>.json one item per layer
vector/<id>.<hash>.pmtiles     vector tiles (read by the pmtiles:// protocol, range requests)
vector/<id>.parquet            the same layer as GeoParquet (download / analysis)
legend/<id>.legend.json        baked legends (built-up, heights)
legends/<series>.webp          legend pictures cut from the land-use sheets
nav/areas.json                 breadcrumb + place-search areas
heritage.json                  City Timeline's heritage list
```

## Raster tiles under W

```
W/r/<release>/<id>.json                TileJSON
W/r/<release>/<id>/{z}/{x}/{y}.<ext>   tile; ext = webp | png | pbf
W/r/<release>/<id>.pmtiles             download (302 to GitHub)
```

Release assets are named `<id>.pmtiles` (no hash; the release tag is the version).
A layer over 2 GB is uploaded as `<id>.mosaic.json` + `<id>-part0000.pmtiles`, …; the
Worker resolves the mosaic itself, so the URLs above stay the same.

## layers.json

Same schema as before (`id, title, group, subgroup, order, kind, geom, source_layer,
version, minzoom, maxzoom, bounds, center, feature_count, fields, description,
source_name, source_url, attribution, date, licence, default_visible, style, encoding,
display, legend, legend_image`) with these changes:

- top level: `"release": "2026-10-07"`, `"data_base": D`, `"tiles_worker": W`. `tiles_base` is gone.
- **vector layers**: `"pmtiles_url": "<D>vector/<id>.<hash>.pmtiles"` (absolute). No `tile_url`.
  The viewer makes the MapLibre source `{ type: 'vector', url: 'pmtiles://' + pmtiles_url }`.
- **raster / terrain / encoded layers**: `"tile_url": "<W>/r/<release>/<id>/{z}/{x}/{y}.<ext>"`
  (absolute, `ext` per file: webp for pictures, png for encoded + terrain).
  MapLibre source `{ type: 'raster' | 'raster-dem', tiles: [tile_url], tileSize, minzoom, maxzoom, bounds }`
  as before. `version` is no longer appended as `?v=` (the release is the version).
- `legend` and `legend_image` are absolute URLs under D.
- new `"download"`: `{ "parquet": "<D>vector/<id>.parquet" }` for vectors,
  `{ "pmtiles": "<W>/r/<release>/<id>.pmtiles" }` for rasters.
- `fields`, `style`, `display` etc. unchanged.

## Apps

- All fetches of `layers.json`, `nav/areas.json`, legends and `heritage.json` use absolute URLs
  under D (a `DATA_BASE` constant), never relative paths, so the apps work from any host.
- No nginx, no `pmtiles serve`, no tile proxy, no CSP header. Apps are static sites on
  GitHub Pages (`maps.hyderabad.urbanobservatory.in/`, `/timeline/`), relative asset paths.
- Vector sources via `pmtiles.js` (vendored) + `maplibregl.addProtocol('pmtiles', …)`.
- Each layer's Details shows **Download** (from `download`) and the licence/source lines.
- Rasters show a small "Loading…" indicator while their source is loading
  (`map.on('sourcedata')` / `map.isSourceLoaded`), hidden once tiles are in.
- Allowed external hosts in the headless check: D's host, W's host, `*.arcgisonline.com`,
  `photon.komoot.io`, plus `github.com` / `objects.githubusercontent.com` redirects for downloads.
