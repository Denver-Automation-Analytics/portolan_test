# Portolan Test, Phase 2: Reorganizing the Catalog for Hundreds of Watersheds

The Phase 1 catalog ([README.md](README.md)) publishes **one watershed** (Second and Third Creek). It
passes `portolan check`, but its hierarchy does not scale. This document describes the target
structure, the per-watershed workflow, and the migration plan.

> **Status:** Proposal. Nothing in `catalog/` or S3 has been changed. Items marked **Verify** depend on
> behavior of `portolan-cli` 0.9.0 or the spec that has not been tested yet.

Spec references: [core.md](https://github.com/portolan-sdi/portolan-spec/blob/main/specs/portolan/core.md),
[formats.md](https://github.com/portolan-sdi/portolan-spec/blob/main/specs/portolan/formats.md),
[philosophy.md](https://github.com/portolan-sdi/portolan-spec/blob/main/specs/best-practices/philosophy.md).

---

## 1. Why the Phase 1 structure doesn't scale

Phase 1 has 27 collections and 4 sub-catalogs for one watershed:

| Group | Collections | Notes |
|---|---|---|
| Raster (depth, velocity, DV, AEP, risk drivers, threat AEP) | 6 | Items are return periods. This part is sound. |
| `hazard-floodplains` | 13 | One collection per return period, each with a single file. |
| `risk-scores-ard` / `-grid` / `-presentation` | 4 / 2 / 2 | Four distinct schemas in ARD. Grid has one collection per resolution. |

Repeated per watershed, that is about **8,100 collections** and 1,200 sub-catalogs, with no way to
ask "what is the 100-year depth at this location" without crawling. The problems:

1. **The watershed is the wrong boundary.** The spec scopes a catalog by *steward*, not by place
   ("Scope by steward, not by theme"). The steward here is MHFD. Watersheds are a dimension *inside*
   the MHFD catalog.
2. **A dataset is split across collections.** Floodplains are one dataset. Thirteen collections make
   every consumer reassemble it. The spec's guidance is one collection per published dataset.
3. **Inconsistent depth.** Raster collections sit at the root. Vector collections sit one level down
   under a sub-catalog. The spec prefers a shallow, uniform layout so an agent that has walked one
   branch has learned the whole catalog.
4. **Single-file collections don't repeat.** A "single file is a collection-level asset" rule fits one
   watershed. Across many watersheds, the same data becomes multiple scenes or partitions.

## 2. Design principles

1. **One MHFD catalog.** One root, one license posture, one contact.
2. **One collection per dataset (schema).** Never pack several datasets into one collection as sibling
   assets. Depth, velocity, and DV stay separate collections.
3. **Watershed is data, not structure.** It is an item property (rasters) or a column and partition key
   (vectors), not a catalog level.
4. **Uniform depth.** Every collection lives at `<theme>/<collection>`.
5. **Collection count is fixed.** Adding a watershed adds items and files, never collections.
6. **Themes stay in prose.** Collections are grouped under two sub-catalogs (`hazard`, `risk`) plus
   `reference`. Anything finer goes in README tables.

## 3. Target hierarchy

```
catalog.json                      MHFD Flood Risk Program (root, absolute self link)
README.md  AGENTS.md
reference/
  catalog.json
  watersheds/                     tabular + polygon index: one row per watershed (see 3.4)
    collection.json  watersheds.parquet  watersheds.pmtiles
hazard/
  catalog.json
  depth/                          raster collection
    collection.json  items.parquet  README.md  AGENTS.md
    <watershed_id>/catalog.json   item catalog (groups the 13 items of one watershed)
      <watershed_id>_100yr_D/<watershed_id>_100yr_D.json
      <watershed_id>_100yr_D/<watershed_id>_100yr_D.tif
  velocity/                       raster, same pattern
  depth_velocity/                 raster, same pattern
  aep/                            raster, one item per watershed
  floodplains/                    partitioned GeoParquet (see 3.2)
    collection.json  README.md  AGENTS.md  floodplains.pmtiles  styles/
    watershed=<watershed_id>/floodplains.parquet
risk/
  catalog.json
  drivers/                        raster, items = watershed x receptor
  threat_aep/                     raster, items = watershed x receptor
  ard_critical_facilities/        partitioned GeoParquet
  ard_roads/                      partitioned GeoParquet
  ard_structures/                 partitioned GeoParquet
  ard_trails/                     partitioned GeoParquet
  grid/                           partitioned GeoParquet, `resolution_m` column (500, 1000)
  presentation_structures/        partitioned GeoParquet
  presentation_transit/           partitioned GeoParquet
```

That is **15 collections** and 3 sub-catalogs, regardless of watershed count. Collection ids are
short and snake_case (`hazard/depth`), with human-readable `title` and `description`, as the
spec requires.

### 3.1 Rasters

| Collection | Item id | Item properties (beyond defaults) |
|---|---|---|
| `hazard/depth`, `velocity`, `depth_velocity` | `<watershed_id>_<N>yr_<D\|V\|DV>` | `watershed`, `return_period_yr`, `aep` (1/N), `units` |
| `hazard/aep` | `<watershed_id>_aep` | `watershed` |
| `risk/drivers`, `risk/threat_aep` | `<watershed_id>_<receptor>` | `watershed`, `receptor` (`structures` or `transit`) |

- Each item carries its COG as an item-level asset (collection-level COG assets are not allowed for
  multi-scene collections).
- Each collection publishes an `items.parquet` item mirror, regenerated on every publish. At about
  300 watersheds, each hazard collection has roughly 3,900 items, and the mirror makes spatial
  search possible with one range request.
- The 13 items of a watershed sit under a per-watershed item catalog (`<watershed_id>/catalog.json`).
  The spec allows a catalog below a collection to organize items, and asks for sub-catalogs once a
  node has 20 or more children. **Verify** that `portolan add` and `portolan check` accept this.
- Use real `datetime` values (the model or study date). Do not use the `1900-01-01` / `9999-12-31`
  sentinels from Phase 1.

### 3.2 Vectors: partitioned collections

Vector collections use the spec's **partitioned collection** model (partition extension v1.0.0),
not one collection per file and not hundreds of items:

- One file per watershed at `<collection>/watershed=<watershed_id>/<name>.parquet`.
- Every file has an identical Parquet schema. `watershed` (and `return_period_yr` for floodplains)
  are columns in the data, so clients can filter after reading one glob.
- The collection sets `partition:scheme`, `partition:keys`, and `partition:glob`. The glob must be an
  absolute `s3://portolan-test/.../*.parquet` URL, and it should also appear in the `description`.
  **Verify** the exact `partition:scheme` value for key=value paths in the extension docs.
- The spec says partition files should **not** be items when there are hundreds of them.
- Floodplains: merge the 13 return-period layers of a watershed into one file with a
  `return_period_yr` column. This makes the 13-collection design unnecessary.
- ARD and presentation layers are small (a few MB per watershed). Hundreds of tiny files hurt DuckDB
  performance, so plan a periodic compaction (for example, by region) once the file count grows. The
  spec's guide is 200 MB to 1 GB per file.
- GeoParquet must still meet the existing rules: `bbox` covering column with statistics, spatially
  ordered rows, row groups of at most 150,000 rows.

**PMTiles:** the spec registers PMTiles as a collection-level link. Build one PMTiles per
collection, merged from per-watershed tilesets (for example with `tile-join`) so adding a watershed
does not rebuild everything. **Verify** this works with `portolan add --pmtiles` and with the
`pmtiles` link and `visual` asset rules.

### 3.3 Provenance and metadata

- Metadata is defined **once** in `.portolan/metadata.yaml` at the root and inherited (title,
  contact, providers, license, keywords), not re-entered per watershed.
- A **data dictionary** (columns, units, code lists) is written once per collection and fills
  `table:columns` descriptions and the README schema table. Phase 1's `known_issues` list is the
  starting point. Do not publish placeholder descriptions.
- `README.md` and `AGENTS.md` are **generated** from metadata on every publish. Each `AGENTS.md`
  needs real content: access patterns (DuckDB glob query, `items.parquet` search), schema notes,
  CRS and units, and related collections. The Phase 1 files are still empty templates.
- Decide provider roles with MHFD. If MBI is the producer of the analysis, the catalog is
  "official". If MBI is only a processor and host, it is a mirror and needs `canonical` links and
  sync records (the 27 `PTL-PRO-002` suggestions).

### 3.4 Watershed reference collection

Add `reference/watersheds`: one row per watershed (id, name, boundary polygon, study date, status,
CRS, link to the model report). It gives users and agents a place to discover coverage, find a
`watershed_id`, and see which watersheds are published, without crawling items. Each new watershed
adds one row.

### 3.5 Naming

| Thing | Convention | Example |
|---|---|---|
| `watershed_id` | short lowercase snake_case, never reused | `second_third_creek` |
| Collection id | snake_case, no pipeline names | `hazard/depth` |
| Item id | `<watershed_id>_<product>` | `second_third_creek_100yr_D` |
| File name | same as item id, so it is unique across watersheds | `second_third_creek_100yr_D.tif` |
| Partition path | `watershed=<watershed_id>/` | `watershed=second_third_creek/` |

Names become URLs and script dependencies. Settle them before publishing watershed number two.

## 4. Raster conformance to fix in the pipeline

- **Internal overviews.** The 44 Phase 1 COGs have none. The spec requires them
  (Optimized GeoTIFF: 512 x 512 tiles, overviews down to one tile). `portolan check` doesn't flag
  this, but `rashid` will. Add overviews when each watershed's rasters are created
  (`rio cogeo create` or `gdaladdo`). They add about a third to file size.
- Embedded statistics (min, max, mean, stddev) are already present, so keep them.
- Confirm raster units (feet or metres, velocity, AEP, DV) with MHFD and store them as item
  properties and in the data dictionary.
- Decide whether to also change compression (for example ZSTD with a float predictor) while the
  files are rewritten for overviews. That is optional.

## 5. Scale estimate

| Measure | One watershed | About 300 watersheds |
|---|---|---|
| Raster COGs | 44 files, about 21 GB (about 28 GB with overviews) | about 13,200 files, 6 to 8 TB |
| Raster items | 44 | about 13,200 (about 3,900 per hazard collection) |
| Vector files | about 22 | about 6,600, or fewer after compaction |
| Collections | 15 | **15** |
| Sub-catalogs | 3 (+ 1 item catalog per watershed per raster collection) | 3 + about 1,800 item catalogs |

Phase 1's layout would reach about 8,100 collections. The target stays at 15.

## 6. S3 and access changes

- **Listing.** `partition:glob` needs clients to list the bucket. The Phase 1 policy only grants
  `s3:GetObject`. Allow anonymous `s3:ListBucket` limited to the catalog prefixes, or decide how
  partitioned layers will be read. Check this with MHFD before enabling.
- **Public access** is already on for the pilot bucket, but the license is still `other`. Get MHFD
  approval and an SPDX license (and license text URL) before any broader rollout.
- Keep SSE-S3 and the existing CORS rules (GET and HEAD, `Range` header allowed).
- Add the absolute root `self` link, and run `portolan check --live` after every publish.
- Remove the stray `catalog/test.pmtiles` from the root.

## 7. Per-watershed onboarding workflow

Make this repeatable and manifest-driven, so each watershed is a data drop and not a design exercise:

1. **Register** the watershed in a manifest (`watershed_id`, name, source folder, study date,
   boundary, CRS) and add its row to `reference/watersheds`.
2. **Pre-flight** the source folder with `portolan scan` and `portolan check --geo-assets` (as in
   Step 2 of the README).
3. **Prepare data:** add COG overviews, convert `.gpkg` to GeoParquet, merge the floodplain return
   periods into one file with `return_period_yr`, add the `watershed` column, and rename files to
   the naming convention.
4. **Add** to the catalog with `portolan add`, which creates items, thumbnails, the item mirror, and
   PMTiles updates.
5. **Regenerate** README and AGENTS files, then run `portolan check` (and `rashid` once available).
6. **Preview** with `portolan push --dry-run`, then push.
7. **Verify** with `portolan check --live`, and open the catalog in the Portolan browser.

## 8. Migration plan

| Phase | Work | Output |
|---|---|---|
| A. Decide | Confirm watershed id scheme, provider roles, license, and the open **Verify** items in this document. | Short decision log |
| B. Rebuild | Rebuild Second and Third Creek in the new layout in a **new prefix** (or a new bucket), leaving the live Phase 1 catalog untouched. | Catalog that passes `portolan check` |
| C. Validate | Run `portolan check --live`, then query with DuckDB (`partition:glob`) and `items.parquet`, and review in the browser. | Test log |
| D. Cut over | Point the public base URL to the new catalog and remove the Phase 1 keys. | One catalog |
| E. Scale test | Onboard a **second** watershed using only the Section 7 workflow. Fix anything that needed hand edits. | Proven pipeline |
| F. Roll out | Onboard the remaining watersheds in batches. Plan file compaction and PMTiles merges. | Full catalog |

Rebuilding in a new prefix means re-uploading about 21 GB (more with overviews). That is far cheaper
now than after hundreds of watersheds are published under the old layout.

## 9. What stays the same

- Source data folders under `data/03-final/` are not reorganized.
- COG and GeoParquet formats, STAC 1.1.0, and the Portolan profile (v0.2.0 schema).
- Git-backed catalog metadata with CI, and data assets in S3.
- Thumbnails, default styles, and the provider list (subject to the role decision above).

## 10. Open questions

| Topic | Owner | Status |
|---|---|---|
| `watershed_id` scheme (MHFD basin names or codes) | MHFD / MBI | Open |
| Provider roles (is MBI also a producer?) | MHFD | Open |
| License and public release | MHFD | Open |
| Do the depth, velocity, and DV grids always share one grid, for item reuse? (Phase 1: depth 1-year differs from the rest) | MBI | Open, depth 1-year has a smaller grid |
| `partition:scheme` value for key=value paths, and anonymous listing | Portolan / AWS admin | **Verify** |
| Item catalogs below a collection in `portolan-cli` 0.9.0 | Portolan | **Verify** |
| One merged PMTiles per collection | Portolan | **Verify** |
| Raster units and column descriptions | MHFD / MBI | Open |
