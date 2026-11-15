# Res 500m

Res 500m

![flood](https://img.shields.io/badge/flood-blue) ![flood hazard](https://img.shields.io/badge/flood_hazard-blue) ![flood risk](https://img.shields.io/badge/flood_risk-blue) ![floodplain](https://img.shields.io/badge/floodplain-blue) ![depth](https://img.shields.io/badge/depth-blue) ![velocity](https://img.shields.io/badge/velocity-blue) ![annual exceedance probability](https://img.shields.io/badge/annual_exceedance_probability-blue) ![stormwater](https://img.shields.io/badge/stormwater-blue) ![hydrology](https://img.shields.io/badge/hydrology-blue) ![Second Creek](https://img.shields.io/badge/Second_Creek-blue) ![Third Creek](https://img.shields.io/badge/Third_Creek-blue) ![Denver](https://img.shields.io/badge/Denver-blue)

## Spatial Coverage

- **Bounding Box**: [-105.35554714951449, 39.38082522323337, -104.41970938939572, 40.17827110824231]

## Temporal Coverage

- **Start**: open
- **End**: ongoing

## Schema

| Column | Type | Description |
|--------|------|-------------|
| ID.500 | int64 |  |
| sar_total_flood_risk_depth_max | double |  |
| sar_total_flood_risk_velocity_max | double |  |
| sar_total_flood_risk_depth_velocity_max | double |  |
| sar_structure_count | double |  |
| sar_structure_at_risk_count | double |  |
| sar_structure_at_risk_pct | double |  |
| sar_total_flood_risk_max | double |  |
| cf_total_flood_risk_depth_max | double |  |
| cf_total_flood_risk_velocity_max | double |  |
| cf_total_flood_risk_depth_velocity_max | double |  |
| cf_structure_count | double |  |
| cf_structure_at_risk_count | double |  |
| cf_structure_at_risk_pct | double |  |
| cf_total_flood_risk_max | double |  |
| rar_total_flood_risk_depth_max | double |  |
| rar_total_flood_risk_velocity_max | double |  |
| rar_total_flood_risk_depth_velocity_max | double |  |
| rar_structure_count | double |  |
| rar_structure_at_risk_count | double |  |
| rar_structure_at_risk_pct | double |  |
| rar_total_flood_risk_max | double |  |
| tar_total_flood_risk_depth_max | double |  |
| tar_total_flood_risk_velocity_max | double |  |
| tar_total_flood_risk_depth_velocity_max | double |  |
| tar_structure_count | double |  |
| tar_structure_at_risk_count | double |  |
| tar_structure_at_risk_pct | double |  |
| tar_total_flood_risk_max | double |  |
| multireceptor_flood_risk | double |  |
| multireceptor_flood_risk_rank | double |  |
| grid_area | double |  |
| area_in_domain | double |  |
| pct_in_domain | double |  |
| effective_area | double |  |
| sar_flood_risk_score | double |  |
| sar_flood_risk_weighted | double |  |
| sar_flood_risk_rank | int64 |  |
| cf_flood_risk_score | double |  |
| cf_flood_risk_weighted | double |  |
| cf_flood_risk_rank | int64 |  |
| rar_flood_risk_score | double |  |
| rar_flood_risk_weighted | double |  |
| rar_flood_risk_rank | int64 |  |
| tar_flood_risk_score | double |  |
| tar_flood_risk_weighted | double |  |
| tar_flood_risk_rank | int64 |  |
| geometry | binary |  |
| bbox | struct<xmin: double, ymin: double, xmax: double, ymax: double> |  |

## Files

| File | Size | Checksum |
|------|------|----------|
| ./aggregated_flood_risk_500m.parquet | 1.4 MB | 1220c44959e9... |

## Quick Start

```python
import geopandas as gpd

gdf = gpd.read_parquet("./aggregated_flood_risk_500m.parquet")
print(gdf.head())
```

## STAC Metadata

- **root**: `../../catalog.json`
- **parent**: `../catalog.json`

## Processing Notes

Final outputs of the MHFD Flood Risk Program for Second and Third Creek, produced by Michael Baker International. Raster files (COG) and vector files (GeoParquet v1.1.0) were confirmed cloud-native in a format pre-flight on 2026-11-14 using portolan check --geo-assets, rio cogeo validate, and gpio inspect. Floodplain layers were originally delivered as GeoPackage (.gpkg) and converted to GeoParquet using portolan check --fix on 2026-11-14. LAT and LONG columns were dropped from risk/FR_scores_presentation/ files (structure_risk.parquet, transit_risk.parquet) because the geometry column carries the same information. PMTiles generated from GeoParquet for all vector collections. Rasters reused as delivered (all 44 TIFs are valid COGs).


## Attribution

Mile High Flood District (MHFD)

## License

[other](https://www.mhfd.org)

## Known Issues

The following column meanings have not been confirmed by MHFD/MBI documentation and are left undescribed until a data dictionary is provided: OCC_CLS and PRIM_OCC (FR_structures.parquet) — occupancy classification code list unknown; occtype (structure_risk.parquet) — code list unknown; found_type (structure_risk.parquet) — foundation type codes unknown; impact_factor (FR_structures, structure_risk, transit_risk, FR_trails) — definition and scale unknown; FR_score_P and FR_score_F (structure_risk, transit_risk) — meaning of P and F suffixes unknown; bldg_type (FR_structures) — code list unknown; priority_level (FR_critical_facilities) — values and meaning unknown; source_aggregate, source_category1, source_category2 (FR_critical_facilities) — classification taxonomy unknown. Raster pixel units (depth in feet or meters, velocity units, AEP scale, DV units) have not been confirmed from project documentation. Many of the 100+ columns in the ARD files (FR_structures 175 cols, FR_roads 111, FR_critical_facilities 121, FR_trails 91) lack descriptions pending a full data dictionary from MHFD/MBI.


---

*Generated by [Portolan](https://github.com/portolan-sdi/portolan-cli) from STAC metadata and .portolan/metadata.yaml*
