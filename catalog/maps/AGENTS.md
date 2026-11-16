# AGENTS.md — Maps

Guidance for AI agents and LLMs working with this collection. This file supplements the README with practical, agent-oriented notes.

## Overview

Holds GeoLibre map projects (`.geolibre`, JSON) that visualize other collections in this catalog. The collection has one asset, `second-third-test`, with role `metadata` and media type `application/json`. It contains no geospatial data of its own and has no items.

## Accessing the data

Fetch the project and read it as JSON:

```python
import json
import urllib.request

url = "https://portolan-test.s3.us-east-1.amazonaws.com/maps/second-third-test.geolibre"
project = json.load(urllib.request.urlopen(url))
for layer in project["layers"]:
    print(layer["name"], layer["type"], layer["source"]["url"])
```

## Schema & field notes

- Top-level keys: `version`, `name`, `mapView`, `basemapStyleUrl`, `layers`, `styles`, `preferences`, `plugins`, `legend`, `metadata`.
- `mapView.bbox` is `[west, south, east, north]` in EPSG:4326.
- Each entry in `layers` has `type` (`cog`, `pmtiles`, `geojson`) and `source.url`. PMTiles URLs carry a `pmtiles://` prefix.
- `basemapStyleUrl` is an external OpenFreeMap style, not part of this catalog.

## Data quality & usage notes

- The layer URLs are absolute and point at the published S3 catalog. If the bucket moves, update them.
- The project is not a STAC data asset, so `portolan add` does not track it. Its STAC metadata and `versions.json` are maintained by hand. After editing the file, refresh `file:size`, `file:checksum` and the `versions.json` entry.

## Example queries

Layers that reference a given collection:

```python
[l["name"] for l in project["layers"] if "hazard-depth" in l["source"]["url"]]
```

## Related collections

`risk-drivers`, `hazard-depth`, `hazard-floodplains`, `risk-scores-presentation`.
