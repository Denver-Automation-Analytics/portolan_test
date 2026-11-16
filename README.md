# Portolan Test: MHFD Flood Risk Catalog (Second & Third Creek)

This test turns the final outputs of the **Mile High Flood District (MHFD) Flood Risk Program** for **Second
and Third Creek** into a [Portolan](https://www.portolan-sdi.org/) catalog. Portolan is an open
specification for cloud-native geospatial data catalogs, built on [STAC](https://stacspec.org/). The
work uses [Portolan agent skills](https://github.com/portolan-sdi/portolan-skills) in Claude Code to:

1. **Set up** the Portolan dependencies.
2. **Convert** the existing data in [data/03-final/2nd3rdCreek](data/03-final/2nd3rdCreek) into a
   spec-compliant Portolan catalog.
3. **Deploy** the catalog and data to the cloud (MBI AWS S3).

The aim is to show how the flood risk assessment outputs can be prepared for modern deployment,
made accessible to agents, and future-proofed by using STAC, AI agents, and cloud-native
geospatial formats together.

> **Status:** The catalog is built and live. Validation and review are in progress.

**Live catalog**
- Catalog root: <https://portolan-test.s3.us-east-1.amazonaws.com/catalog.json>
- Browse it in the Radiant Earth-style STAC browser: [open the catalog](https://browser.moregeo.it/external/portolan-test.s3.us-east-1.amazonaws.com/catalog.json)
- Catalog metadata repository: [Denver-Automation-Analytics/portolan_test](https://github.com/Denver-Automation-Analytics/portolan_test)

> Public access is a pilot setting, pending MHFD approval of the license and public release.

---

## Why Portolan

| Goal | How Portolan delivers it |
|---|---|
| **Modern deployment** | A catalog is a set of static files (STAC JSON plus cloud-native data) in object storage. No server, database, or GIS service needs to be maintained. Clients read only the bytes they need with HTTP range requests. |
| **Agentic access** | The spec requires an `AGENTS.md` and a `README.md` next to every catalog and collection. These files give AI agents documented schemas, provenance, and example queries. An agent that has the [`reading-portolan`](https://github.com/portolan-sdi/portolan-skills/blob/main/skills/reading-portolan/SKILL.md) skill can find and query the data without any custom integration. |
| **Future-proofing** | The catalog uses open standards: STAC for metadata, Cloud-Optimized GeoTIFF (COG) for rasters, GeoParquet for vectors, and PMTiles for map rendering. It is validated against an open, versioned [spec](https://github.com/portolan-sdi/portolan-spec). It isn't tied to any vendor or proprietary format. |
| **Trust & provenance** | Each claim in the metadata has to be attested by the source, researched and cited, or derived from the data. Producer and host roles are recorded explicitly under `providers`. |

## Scope & status

| Phase | Description | Status |
|---|---|---|
| 0. Plan | This README | ✅ Done |
| 1. Set up | Install the Claude Code plugin and `portolan-cli` | ✅ Done (confirm `/portolan:` skills in a new session) |
| 2. Format pre-flight | Confirm which files are already cloud-native | ✅ Done (see Step 2 findings) |
| 3. Convert & catalog | Bootstrap a git-backed Portolan catalog | ✅ Done |
| 4. Deploy | Push data and catalog to MBI AWS S3 | ✅ Done (live, see above) |
| 5. Validate & review | `portolan check`, `catalog-grader`, Portolan browser | 🔄 In progress |
| Next | Front end, agentic access demos | 🔜 Not yet |

**Deployment model (git-backed catalog):** catalog **metadata** (STAC JSON, `README.md`, `AGENTS.md`, styles)
is stored in a GitHub repository, and CI validates every change. **Data assets** (COG, GeoParquet,
PMTiles) are stored in the **S3 store** and referenced by their public HTTPS URLs.

---

## Source data inventory

Root: [data/03-final/2nd3rdCreek](data/03-final/2nd3rdCreek). The total is about 21 GB.

The return periods are 1, 2, 5, 10, 25, 50, 100, 200, 300, 400, 500, 750, and 1000 years.

| Path | Contents | Files | Size | Format |
|---|---|---|---|---|
| `hazard/aep/` | `aep_grid.tif`: annual exceedance probability grid | 1 | 174 MB | COG |
| `hazard/depth/` | `<N>yr_D.tif`: flood depth for each return period | 13 | 5.3 GB | COG |
| `hazard/velocity/` | `<N>yr_V.tif`: flow velocity for each return period | 13 | 6.8 GB | COG |
| `hazard/dv/` | `<N>yr_DV.tif`: depth × velocity for each return period | 13 | 7.1 GB | COG |
| `hazard/floodplains/` | `<N>yr_FP.gpkg`: floodplain polygons for each return period | 13 | 523 MB | GeoPackage |
| `risk/risk_grids/` | `risk_driver_{structures,transit}.tif`, `threat_aep_grid_{structures,transit}.tif` | 4 | 1.2 GB | COG |
| `risk/FR_scores_ard/` | Flood risk scores for critical facilities, roads, structures, and trails | 4 | 24 MB | GeoParquet |
| `risk/FR_scores_grid/` | Aggregated flood risk at 500 m and 1000 m | 2 | 2 MB | GeoParquet |
| `risk/FR_scores_presentation/` | Structure and transit risk | 2 | 22 MB | GeoParquet |

**Excluded from the catalog:**
- `risk/FR_scores_presentation/flood_risk.gdb`: the GeoParquet files in the same folder are equivalent.
- `hazard/depth/*.tif.aux.xml`: GDAL statistics sidecar files, which are not data assets.

> Descriptions of methods, units, and attributes are **TBD**. They will come from the MHFD and MBI
> project documentation during bootstrap, as the skill's provenance rules require. Don't guess
> them.

---

## Proposed catalog structure

This is a draft. The agent refines it during bootstrap and checks it with the user.

| Collection | Source | Model |
|---|---|---|
| `hazard-depth` | `hazard/depth/*.tif` | Raster collection with one item per return period (PORTO-CORE-071) and an `items.parquet` mirror |
| `hazard-velocity` | `hazard/velocity/*.tif` | Same as `hazard-depth` |
| `hazard-depth-velocity` | `hazard/dv/*.tif` | Same as `hazard-depth` |
| `hazard-aep` | `hazard/aep/aep_grid.tif` | A single COG stored as a collection-level asset (PORTO-CORE-072) |
| `hazard-floodplains` | `hazard/floodplains/*.gpkg` | Vector collection (GeoParquet + PMTiles), with either one item per return period or one combined layer. **Open question.** |
| `risk-grids` | `risk/risk_grids/*.tif` | Raster collection with one item per file (risk driver and threat AEP, each for structures and transit). **Open question:** keep as one collection, or split into `risk-drivers` and `risk-threat-aep`. |
| `risk-scores-ard` | `risk/FR_scores_ard/*.parquet` | Vector collection(s) + PMTiles |
| `risk-scores-grid` | `risk/FR_scores_grid/*.parquet` | Vector collection(s) + PMTiles |
| `risk-scores-presentation` | `risk/FR_scores_presentation/*.parquet` | Vector collection(s) + PMTiles |

**Providers (draft):** MHFD as `producer` and `licensor`, and Michael Baker International as `processor` and
`host`. The host is listed last, as PORTO-CORE-046 and PORTO-CORE-047 require. MHFD confirms the final roles.

---

## Prerequisites

- [Claude Code](https://code.claude.com/docs/en/setup)
- [uv](https://docs.astral.sh/uv/getting-started/installation/), which installs `portolan-cli`
- Git and the [GitHub CLI](https://cli.github.com/) (`gh`), with permission to create a repository in `Denver-Automation-Analytics`
- The AWS CLI, with MBI credentials that can write to the `portolan-test` bucket
- Node.js 24.12 or later and npm, which [`portolan-thumbnails`](https://github.com/portolan-sdi/portolan-skills/blob/main/skills/portolan-thumbnails/SKILL.md#requirements) needs
- Optional, for the format pre-flight: GDAL / [`rio-cogeo`](https://github.com/cogeotiff/rio-cogeo) and [`gpio`](https://github.com/geoparquet/geoparquet-io)
- [tippecanoe](https://github.com/felt/tippecanoe), which builds the PMTiles for the vector collections. Windows needs a Linux build (see Step 1)

> **Devcontainer note:** [.devcontainer](.devcontainer) supplies Git, Node, the AWS CLI, and micromamba.
> However, its `Dockerfile` copies `env.yaml`, and that file is not in this folder yet. Add `env.yaml`
> before you use the devcontainer.

---

## Step 1: Install dependencies

Run these commands from a terminal (PowerShell, Git Bash, or any shell). Each one has a check you
can run right after it.

**1. Add the Portolan skills marketplace**

```bash
claude plugin marketplace add portolan-sdi/portolan-skills
claude plugin marketplace list      # check: portolan-skills is listed
```

**2. Install the Portolan plugin** (gives Claude Code the `portolan:` skills)

```bash
claude plugin install portolan@portolan-skills
claude plugin list                  # check: portolan@portolan-skills, Status: enabled
```

**3. Install the Portolan CLI** (the agent runs this behind the scenes)

```bash
uv tool install portolan-cli
portolan --version                  # check: prints a version
```

**4. Install the optional Step 2 tools** (format pre-flight)

```bash
uv tool install rasterio --with rio-cogeo    # provides `rio` and `rio cogeo`
uv tool install geoparquet-io                # provides `gpio`
rio cogeo --help                             # check: lists validate, info, create
gpio --version                               # check: prints a version
```

**5. Install tippecanoe** (needed for PMTiles)

`portolan add --pmtiles` calls [tippecanoe](https://github.com/felt/tippecanoe) to build the vector
tiles for the floodplain and risk-score collections. Rasters don't need it. Portolan finds it on the
`PATH` and skips PMTiles with a warning if it's missing.

| System | Install |
|---|---|
| macOS | `brew install tippecanoe` |
| Linux / Ubuntu | `sudo apt install tippecanoe`, or build from source (`make -j && sudo make install`) |
| **Windows** | Needs a Linux build. See below. |

**Windows:** the PyPI package is source-only and needs `make`, so `uv tool install tippecanoe` fails.
The working route is to build tippecanoe on Linux (Docker, with Ubuntu), copy the binary into WSL,
and add a Windows wrapper script on the `PATH` that calls it, so the Windows `portolan` can run it.

```bash
tippecanoe --version                # check: prints a version
```

> Windows steps are still being verified. Fill in the exact commands once they work.

**6. Start a new Claude Code session**

Plugins only load when a session starts. Run `claude` in a new terminal, then type `/portolan:` to
see the skills. They use the `portolan:` prefix, for example `/portolan:portolan-bootstrap`.

Versions used for this test (2026-11-14, Windows):

| Component | Version |
|---|---|
| `portolan@portolan-skills` plugin | 0.5.0 (user scope) |
| `portolan-cli` | 0.9.0 |
| `rasterio` (`rio`) / `rio-cogeo` | 1.5.2 / 7.0.4 |
| `geoparquet-io` (`gpio`) | 1.7.0 |
| `tippecanoe` | TBD (record after install) |

**Troubleshooting**
- `portolan`, `rio`, or `gpio` is "not recognized": uv installs tools to `~\.local\bin`. Open a new
  terminal, or run `uv tool update-shell`.
- PowerShell blocks `claude` or `npm` (they are `.ps1` scripts) with an execution-policy error: run
  `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.
- Re-run `claude plugin marketplace update portolan-skills` to pick up newer skill versions.

## Step 2: Format pre-flight (verify cloud-native formats)

Confirm which source files are **already cloud-native** before you convert anything. Compliant COGs
and GeoParquet files are reused as they are, and only the flagged files get converted.

Nothing in this step needs a catalog. `portolan scan` and `portolan check --geo-assets` work on any
folder, so run them **before** `portolan init`. Every command here is read-only. Run them from the
repository root:

**1. Scan the folder with Portolan** (finds geo-assets, sidecars, and path problems)

```bash
portolan scan data/03-final/2nd3rdCreek --suggest-collections
```

**2. Check which assets are already cloud-native, and preview conversions** (no changes)

```bash
portolan check --geo-assets data/03-final/2nd3rdCreek
portolan check --geo-assets --fix --dry-run data/03-final/2nd3rdCreek
```

**3. Spot-check individual files with the standalone tools** (optional)

```bash
rio cogeo validate data/03-final/2nd3rdCreek/hazard/depth/100yr_D.tif
rio cogeo validate data/03-final/2nd3rdCreek/risk/risk_grids/risk_driver_structures.tif
gpio inspect data/03-final/2nd3rdCreek/risk/FR_scores_ard/FR_structures.parquet
```

> **Windows:** if `portolan` crashes with a `UnicodeEncodeError` traceback, set UTF-8 first. In
> PowerShell, run `$env:PYTHONUTF8="1"`. In Git Bash, run `export PYTHONUTF8=1`.

> The full `portolan check` (metadata, STAC structure, data) validates a *catalog*, so it runs later,
> after `portolan init` in Step 3.

> **Don't run `check --fix` for real in this step.** Step 2 is read-only, and `--fix` writes new
> GeoParquet files next to the `.gpkg` sources. Portolan's bootstrap flow runs the real conversion
> in Step 3, after `portolan init` and the dry run: `init`, `scan`, `check --fix --dry-run`,
> `check --fix`, `add`.

### Findings (confirmed with `portolan check --geo-assets`, `rio cogeo validate`, and `gpio inspect`)

`portolan scan` reports 66 geo-assets. `portolan check --geo-assets` finds 52 already cloud-native
and 13 needing conversion. The one remaining asset is `flood_risk.gdb`, which is excluded.

| Files | Finding | Action |
|---|---|---|
| All 39 depth, velocity, and DV rasters, `aep_grid.tif`, `threat_aep_grid_*.tif`, and `risk_driver_*.tif` (44 files) | Valid COG | Reuse as-is. `rio cogeo` warns that the files lack internal overviews, which is only a recommendation. |
| All 8 `risk/**/*.parquet` | GeoParquet **v1.1.0** (for example `FR_structures.parquet`: EPSG:6428, 175 columns) | Reuse as-is. Generate PMTiles for rendering. |
| 13 `hazard/floodplains/*.gpkg` | GeoPackage, which is **not cloud-native** | Convert to GeoParquet and generate PMTiles |

**Source folder layout:** a folder should hold either data files or subfolders, not both. The four
rasters that sat directly in `risk/` were moved into `risk/risk_grids/`, which fixed the scan's
"data files AND subdirectories" warning.

`portolan scan` still raises 11 "manual grouping" warnings. These are expected: each folder holds
many assets, so you decide the collection and item layout in Step 3 (see
[Proposed catalog structure](#proposed-catalog-structure)).

## Step 3: Convert & catalog (Claude Code + Portolan skills)

Start Claude Code in a working directory next to this repository, then paste this prompt. Replace
the placeholders first.

```text
/portolan:portolan-bootstrap

Build a Portolan catalog from the local folder
<path-to>/portolan_test/data/03-final/2nd3rdCreek
(MHFD Flood Risk Program outputs for Second and Third Creek).

- Use git-backed-catalog with the GitHub repository Denver-Automation-Analytics/portolan_test
  (https://github.com/Denver-Automation-Analytics/portolan_test) and the portolan-catalog-template.
  Keep the metadata in git with CI validation, and keep the data files in AWS S3 at
  s3://portolan-test, served from https://portolan-test.s3.us-east-1.amazonaws.com.
- Formats: the rasters are already COGs and the vectors are already GeoParquet v1.1.0. Reuse them
  as they are and do not re-encode them. Convert only this:
    * hazard/floodplains/*.gpkg -> GeoParquet
  Generate PMTiles and default styles for every vector collection.
- Ignore risk/FR_scores_presentation/flood_risk.gdb (it duplicates the Parquet files) and the
  *.aux.xml sidecar files.
- Model each hazard raster family (depth, velocity, DV) as one collection, with one item for each
  return period (1, 2, 5, 10, 25, 50, 100, 200, 300, 400, 500, 750, 1000 yr).
- Providers: MHFD (producer, licensor) and Michael Baker International (processor, host).
- License: TBD, pending MHFD approval. Ask me before you set one.
- Use portolan-thumbnails to generate the thumbnails and visually check them.
- Do not invent descriptions of methods, units, or attributes. Ask me, or record the gaps in
  known_issues.
- Ask me for the repository name, the license, and the maintainer contact. Show me the plan and
  the upload preview (dry run) before you publish anything.
```

Key CLI commands the agent uses, for reference:

```bash
portolan init --auto --license <SPDX> --title "MHFD Flood Risk - Second & Third Creek"
portolan check --fix --dry-run               # preview the conversions (the 13 .gpkg files)
portolan check --fix                         # convert them to GeoParquet (keeps the .gpkg unless --remove-legacy)
portolan add <collection-dir> --pmtiles      # vectors: GeoParquet + PMTiles + style
portolan add <collection-dir>                # rasters: COG items
portolan check                               # validate against the spec
```

Checkpoints to expect:
- A dry run before every expensive step
- A source tier for each metadata claim (attested, researched and cited, or derived)
- Gaps recorded in `known_issues`
- The template's guards: `tests/test_setup.py` and the sentinel check in `catalog.publish.yaml`

## Step 4: Deploy to AWS S3

| Target | Value |
|---|---|
| GitHub repository (catalog metadata) | [Denver-Automation-Analytics/portolan_test](https://github.com/Denver-Automation-Analytics/portolan_test) |
| S3 bucket (data assets) | `portolan-test` (`arn:aws:s3:::portolan-test`) |
| Public base URL | `https://portolan-test.s3.us-east-1.amazonaws.com` (Region `us-east-1`, catalog at the bucket root) |

The git-backed template keeps data uploads separate from catalog publishing:

```bash
# Data assets (COG / GeoParquet / PMTiles) -> S3. Run it dry first, then confirm.
python3 tools/upload_data.py
python3 tools/upload_data.py --confirm

# Catalog metadata (catalog/) -> S3. Run it dry first.
python3 tools/publish.py
```

If you don't use the template, `portolan push s3://portolan-test --dry-run` followed by
`portolan push` does the same job.

Requirements on the S3 side:
- **Public read** on the catalog prefix only (bucket policy `s3:GetObject` on
  `arn:aws:s3:::portolan-test/<prefix>/*`). Turn off only the two *policy* options of Block Public
  Access, and keep the two ACL options on. MHFD must approve public access before it is enabled.
- **Encryption**: use SSE-S3. SSE-KMS breaks anonymous reads.
- **CORS**: allow `GET` and `HEAD`, allow the `Range` request header, and expose `Content-Range`,
  `Accept-Ranges`, `Content-Length`, and `ETag`, so browsers and agents can make range requests.
  In the console, paste the rules as a JSON array.
- Asset hrefs use **`https://`** URLs, not `s3://` (PORTO-CORE-023). The `s3://` form can be offered
  through alternate assets.
- At publish time, add an absolute `self` link on the root catalog (PORTO-CORE-081).

Check the live deployment:

```bash
portolan check --live --url https://portolan-test.s3.us-east-1.amazonaws.com
```

## Step 5: Validate & review

1. `portolan check`, plus CI (`rashid`) on every pull request in the catalog repository.
2. Open the catalog in the [Portolan browser](https://browser.portolan-sdi.org/). Check the maps,
   thumbnails, legends, and download links.
3. Ask the agent to run the **`catalog-grader`** skill for an A+ to F quality review with evidence.
4. *(Optional, after MHFD approval)* Use **`register-catalog`** to list the catalog in the Portolan registry.

---

## Geolibre incorporation

[Geolibre](https://geolibre.app/) is a free and open source, browser based GIS system that is well suited for portolan-spec datasets. An
example geolibre project has been created and is shareable for anyone with this [link](https://share.geolibre.app/daniel-aragon/portolan-test-second-third-creek). The geolibre file has also been added to the catalog.json as its own catalog - readily accessible next to the other assets of the project as it's own first class dataset.

**Potential benefits**
- Ready-made GUI for every watershed
- Preconfigured map styles and symbology, stored in json (or even as stac items)
- Easy access for all stakeholders (no desktop based GUI configuration)
- No download of cloud items needed (when published to a public blob store, ie AWS S3)

## Next steps (not in scope yet)

- **Front end:** a publisher-branded viewer for this catalog, built either from
  [`customize-portolan-browser`](https://github.com/portolan-sdi/portolan-skills/blob/main/skills/customize-portolan-browser/SKILL.md)
  or as a custom application.
- **Agentic access demos:** a spec-compliant catalog supports agent access already, through
  `AGENTS.md` and [`reading-portolan`](https://github.com/portolan-sdi/portolan-skills/blob/main/skills/reading-portolan/SKILL.md).
  Worked examples of agents answering questions about flood risk will come later.
- **More watersheds:** reuse this workflow for other MHFD Flood Risk Program study areas.
- **CI publishing:** automate publishing when changes merge to `main`.

## Open questions & decisions log

| Topic | Status |
|---|---|
| License (open license, public release) | **Open**: needs MHFD approval |
| S3 prefix, Region, and public base URL | **Decided**: `https://portolan-test.s3.us-east-1.amazonaws.com`, catalog at the bucket root |
| GitHub org and repository name for the catalog metadata | **Decided**: `Denver-Automation-Analytics/portolan_test` |
| Floodplains: one item per return period, or one combined layer | **Open** |
| `risk-grids`: one collection, or split into drivers and threat AEP | **Open** |
| `risk_driver_*.tif`: confirm COG status, and rewrite if needed | **Decided**: already valid COGs, reuse as-is (Step 2) |
| Units and descriptions for the hazard and risk layers | **Open**: comes from MHFD/MBI documentation |
| `flood_risk.gdb` | **Decided**: excluded, because the Parquet files are equivalent |
| Where data and metadata live | **Decided**: data assets in S3, catalog metadata in GitHub |
| Agent platform | **Decided**: Claude Code with the `portolan@portolan-skills` plugin |
| Conversion scope | **Decided**: all data |

## References

- Portolan: <https://www.portolan-sdi.org/>
- Portolan spec: <https://github.com/portolan-sdi/portolan-spec>
- Portolan skills: <https://github.com/portolan-sdi/portolan-skills>
- Portolan CLI docs: <https://cli.portolan-sdi.org/>
- Catalog template (git-backed): <https://github.com/portolan-sdi/portolan-catalog-template>
- Portolan browser: <https://browser.portolan-sdi.org/>
- Example catalog: [portolan-nl](https://source.coop/cholmes/portolan-nl)
- STAC: <https://stacspec.org/>, COG: <https://cogeo.org/>, GeoParquet: <https://geoparquet.org/>, PMTiles: <https://docs.protomaps.com/pmtiles/>
