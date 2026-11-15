# Trails

Trails

![flood](https://img.shields.io/badge/flood-blue) ![flood hazard](https://img.shields.io/badge/flood_hazard-blue) ![flood risk](https://img.shields.io/badge/flood_risk-blue) ![floodplain](https://img.shields.io/badge/floodplain-blue) ![depth](https://img.shields.io/badge/depth-blue) ![velocity](https://img.shields.io/badge/velocity-blue) ![annual exceedance probability](https://img.shields.io/badge/annual_exceedance_probability-blue) ![stormwater](https://img.shields.io/badge/stormwater-blue) ![hydrology](https://img.shields.io/badge/hydrology-blue) ![Second Creek](https://img.shields.io/badge/Second_Creek-blue) ![Third Creek](https://img.shields.io/badge/Third_Creek-blue) ![Denver](https://img.shields.io/badge/Denver-blue)

## Spatial Coverage

- **Bounding Box**: [-104.857509950441, 39.7877143869053, -104.74348636682407, 39.98693728742624]

## Temporal Coverage

- **Start**: open
- **End**: ongoing

## Schema

| Column | Type | Description |
|--------|------|-------------|
| length | double |  |
| trail_coords | string |  |
| impact_factor | double |  |
| receptor_id | string |  |
| segment_ID | int64 |  |
| depth_1000_max | double |  |
| depth_750_max | double |  |
| depth_500_max | double |  |
| depth_400_max | double |  |
| depth_300_max | double |  |
| depth_200_max | double |  |
| depth_100_max | double |  |
| depth_50_max | double |  |
| depth_25_max | double |  |
| depth_10_max | double |  |
| depth_5_max | double |  |
| depth_2_max | double |  |
| depth_1_max | double |  |
| velocity_1000_max | double |  |
| velocity_750_max | double |  |
| velocity_500_max | double |  |
| velocity_400_max | double |  |
| velocity_300_max | double |  |
| velocity_200_max | double |  |
| velocity_100_max | double |  |
| velocity_50_max | double |  |
| velocity_25_max | double |  |
| velocity_10_max | double |  |
| velocity_5_max | double |  |
| velocity_2_max | double |  |
| velocity_1_max | double |  |
| depth_velocity_1000_max | double |  |
| depth_velocity_750_max | double |  |
| depth_velocity_500_max | double |  |
| depth_velocity_400_max | double |  |
| depth_velocity_300_max | double |  |
| depth_velocity_200_max | double |  |
| depth_velocity_100_max | double |  |
| depth_velocity_50_max | double |  |
| depth_velocity_25_max | double |  |
| depth_velocity_10_max | double |  |
| depth_velocity_5_max | double |  |
| depth_velocity_2_max | double |  |
| depth_velocity_1_max | double |  |
| hazard_score_depth_1000_max | double |  |
| hazard_score_depth_750_max | double |  |
| hazard_score_depth_500_max | double |  |
| hazard_score_depth_400_max | double |  |
| hazard_score_depth_300_max | double |  |
| hazard_score_depth_200_max | double |  |
| hazard_score_depth_100_max | double |  |
| hazard_score_depth_50_max | double |  |
| hazard_score_depth_25_max | double |  |
| hazard_score_depth_10_max | double |  |
| hazard_score_depth_5_max | double |  |
| hazard_score_depth_2_max | double |  |
| hazard_score_depth_1_max | double |  |
| hazard_score_depth_velocity_1000_max | double |  |
| hazard_score_depth_velocity_750_max | double |  |
| hazard_score_depth_velocity_500_max | double |  |
| hazard_score_depth_velocity_400_max | double |  |
| hazard_score_depth_velocity_300_max | double |  |
| hazard_score_depth_velocity_200_max | double |  |
| hazard_score_depth_velocity_100_max | double |  |
| hazard_score_depth_velocity_50_max | double |  |
| hazard_score_depth_velocity_25_max | double |  |
| hazard_score_depth_velocity_10_max | double |  |
| hazard_score_depth_velocity_5_max | double |  |
| hazard_score_depth_velocity_2_max | double |  |
| hazard_score_depth_velocity_1_max | double |  |
| hazard_score_velocity_1000_max | double |  |
| hazard_score_velocity_750_max | double |  |
| hazard_score_velocity_500_max | double |  |
| hazard_score_velocity_400_max | double |  |
| hazard_score_velocity_300_max | double |  |
| hazard_score_velocity_200_max | double |  |
| hazard_score_velocity_100_max | double |  |
| hazard_score_velocity_50_max | double |  |
| hazard_score_velocity_25_max | double |  |
| hazard_score_velocity_10_max | double |  |
| hazard_score_velocity_5_max | double |  |
| hazard_score_velocity_2_max | double |  |
| hazard_score_velocity_1_max | double |  |
| flood_risk_depth_max | double |  |
| flood_risk_velocity_max | double |  |
| flood_risk_depth_velocity_max | double |  |
| flood_risk_max | double |  |
| FR_score_P | double |  |
| FR_score_F | double |  |
| receptors_version | string |  |
| geometry | binary |  |
| bbox | struct<xmin: double, ymin: double, xmax: double, ymax: double> |  |

## Files

| File | Size | Checksum |
|------|------|----------|
| ./FR_trails.parquet | 1.1 MB | 1220be98ac87... |

## Quick Start

```python
import geopandas as gpd

gdf = gpd.read_parquet("./FR_trails.parquet")
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
