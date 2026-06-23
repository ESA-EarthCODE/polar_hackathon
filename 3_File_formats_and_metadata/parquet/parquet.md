# GeoParquet

GeoParquet is a cloud-native format for geospatial vector data. It is useful when a dataset is made of features such as points, lines, or polygons rather than a dense raster grid or data cube.

The Antarctic grounding-line product is a good example. The data describes grounding-line positions as vector geometries with attributes such as acquisition date and satellite sensor. That is a natural fit for GeoParquet: the geometry is kept with the tabular metadata, and users can read only the columns they need from object storage.

GeoParquet builds on [Apache Parquet](https://parquet.apache.org/), a widely used columnar storage format for tabular data. The "Geo" part defines how geometry columns and geospatial metadata, such as the coordinate reference system (CRS), are stored. Because GeoParquet is still standard Parquet, non-geospatial tools can read the table, while geospatial tools such as GeoPandas can also understand the geometry.

This guide adapts material from the [Cloud-Native Geospatial Guide](https://guide.cloudnativegeo.org/geoparquet/).

## Why GeoParquet?

Cloud-hosted datasets should be usable without downloading full archives first. GeoParquet helps because it supports:

- **Column access:** read only the attributes needed for an analysis, such as `SENSOR`, `DATE1`, and `geometry`.
- **Efficient filtering:** skip row groups whose statistics show they cannot match a query.
- **Compact storage:** store large vector tables more efficiently than CSV or many shapefile sidecars.
- **Interoperability:** work with Python, GeoPandas, PyArrow, DuckDB, GDAL, QGIS, and other tools.
- **Cloud access:** read directly from object storage using HTTP or S3-compatible endpoints.

GeoParquet is especially appropriate for vector products such as grounding lines, calving fronts, polygons, transects, point observations, and catalog-like feature tables.

## Example: Antarctic Grounding Lines

This example reads an Antarctic grounding-line GeoParquet file directly from object storage:

```python
import geopandas as gpd

bucket = "s3://EarthCODE/"
endpoint_url = "https://s3.waw4-1.cloudferro.com"
region_name = "eu-west-2"
file = "OSCAssets/polar_cube_datasets/groundlines/InSAR_GL_Antarctica.parquet"

gdf = gpd.read_parquet(
    f"{bucket}{file}",
    storage_options={
        "anon": True,
        "client_kwargs": {
            "endpoint_url": endpoint_url,
            "region_name": region_name,
        },
    },
)

gdf.head()
```

Once loaded, the dataset behaves like a normal GeoDataFrame:

```python
gdf["SENSOR"].unique()
```

You can also reproject and plot the grounding lines in Antarctic Polar Stereographic coordinates:

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(12, 12))
gdf.to_crs("EPSG:3031").plot(column="SENSOR", legend=True, ax=ax)
ax.set_title("Grounding-line observations by sensor")
```

This is the basic cloud-native pattern: open the remote vector asset directly, then filter, plot, or join it with other datasets.

## File Layout

Parquet files are laid out differently from row-oriented formats like CSV, so it helps to think of them as column-oriented tables split into chunks.

![Schematic of Parquet file layout](../../static/geoparquet_layout.png)

A Parquet file consists of row groups. Each row group contains column chunks, where the values for each column are stored together. This layout matters in the cloud because a reader can fetch only the byte ranges needed for selected columns and selected row groups.

The Parquet footer stores metadata describing the schema, column locations, and row-group statistics. A reader usually fetches the footer first, then requests only the relevant column chunks.

## What GeoParquet Adds

GeoParquet defines two geospatial conventions on top of standard Parquet:

- how geometries are encoded, usually as [Well-Known Binary](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry#Well-known_binary) (WKB);
- how geospatial metadata such as the geometry column name, geometry type, bounds, and CRS are recorded.

GeoParquet supports the usual OGC Simple Features geometry types: `Point`, `LineString`, `Polygon`, `MultiPoint`, `MultiLineString`, `MultiPolygon`, and `GeometryCollection`.

It is best to keep a single geometry type per file where possible. For example, grounding lines should be stored as line geometries, while calving-front outlines or drainage basins may be stored as line or polygon geometries depending on the source product.

## Filtering and Access Patterns

GeoParquet is strongest when users access a subset of attributes or features:

```python
cols = ["SENSOR", "DATE1", "geometry"]

gdf = gpd.read_parquet(
    f"{bucket}{file}",
    columns=cols,
    storage_options={
        "anon": True,
        "client_kwargs": {
            "endpoint_url": endpoint_url,
            "region_name": region_name,
        },
    },
)
```

If a dataset is sorted or partitioned by a useful field, readers can skip more data. For polar time-series vector products, useful fields may include observation year, satellite sensor, basin ID, or product version.

::: {.callout-note}

GeoParquet is a file format, not a database. It can support efficient column reads and row-group skipping, but it does not replace a spatial database with multiple indexes. For very complex spatial queries, use a database or a spatial indexing workflow. For portable cloud-hosted vector assets, GeoParquet is a strong default.

:::

## Multi-File Datasets

Small and medium vector products are often easiest to publish as a single `.parquet` file. Larger products can be split into multiple Parquet files, often partitioned by a field such as year or region.

For multi-file datasets, publish shared metadata files such as `_metadata` and `_common_metadata` where possible. Without a top-level metadata file, readers may need to inspect the footer of every Parquet file before reading data.

This pattern is useful for products that grow over time. For example, new yearly vector observations can be written as additional files, as long as the schema remains consistent.

## Describing GeoParquet with STAC

GeoParquet stores the data. STAC describes how users discover it.

A GeoParquet asset should be described in STAC with clear metadata about:

- the product title and scientific purpose;
- spatial and temporal extent;
- license and provenance;
- geometry type and CRS;
- key attributes such as sensor, acquisition date, or processing version;
- a direct asset link to the `.parquet` file or partitioned dataset.

In the grounding-lines case, the STAC record should make it clear that the product contains Antarctic grounding-line positions derived from multiple SAR missions between 1992 and 2014, and that the cloud asset is the GeoParquet file used by the example above.

## When to Use GeoParquet

Use GeoParquet when your product is vector or tabular geospatial data:

- grounding lines;
- calving fronts;
- glacier outlines;
- drainage basins;
- point measurements;
- ship, buoy, or campaign tracks;
- feature tables with geometries and attributes.

Use COG for scene-like raster imagery. Use Zarr for dense, aligned multidimensional data cubes. GeoParquet is the vector counterpart to those formats: compact, cloud-friendly, and easy to use from Python and GIS tools.
