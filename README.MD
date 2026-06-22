# Polar Hackathon

This repository is a guided set of notebooks for discovering Antarctic datasets, converting them into cloud-native formats, combining them into analysis-ready collections, and using those collections in scientific workflows.

## Repository Map

| Folder | Purpose | Best starting point |
| --- | --- | --- |
| `1_Datasets/` | Dataset catalogue and one access notebook per source product. | `1_Datasets/datasets_sumarry.ipynb` |
| `2_Datacube/` | Remote combined-cube access and examples for building a small cube. | `2_Datacube/1_remote_cube_access.ipynb` |
| `3_File_formats_and_metadata/` | Format conversion examples for Zarr, COG, GeoParquet, and the creation of STAC metadata. | `3_File_formats_and_metadata/0_intro.ipynb` |
| `4_Visualisation/` | Early visualisation experiments for interactive and static views. | `4_Visualisation/intro.ipynb` |
| `Hackathon Code/` | Your hackathon solution space. Put new work here unless a challenge asks otherwise. | Create your own notebook or package here. |

Generated local data is expected in `downloaded_data/`, which is ignored by Git. The canonical copies of the prepared assets live in cloud object storage.


# EarthCODE Context



# 1. Datasets prepared for the hackathon.
Each of the datasets has an associated notebook, referenced in the access column that shows how the data can be accessed directly from object storage. More information about the datasets is available in the respective notebooks and the OSC pages. All notebooks reside in the `1_Datasets` folder.



| Dataset | Theme | Prepared format | CRS/grid | Time coverage | Licence | Links | Access notebook |
| --- | --- | --- | --- | --- | --- | --- | --- |
| AI4AIS Larsen C cube | Ice shelf combined variables | Zarr | EPSG:3031 | Monthly from 2014 - 2021 | CC-BY-NC-4.0 | [OSC](https://opensciencedata.esa.int/products/ai4ais-larsen-c-ice-shelf-cube/collection) | `ai4ais/access.ipynb` |
| ALBATROS tidal elevation | Altimetry tide elevation prediction | GeoParquet | Points in EPSG:4326, examples regrid to EPSG:3031 | Example uses 2019 CryoSat-2 predictions | CC-BY-NC-4.0 | [OSC](https://albatross.noveltis.fr/) | `albatros/access.ipynb` |
| Ice shelf basal melt | Ice shelf melt rates | COG/GeoTIFF plus GeoParquet shelf time series | Antarctic polar stereographic, EPSG:3031 | 1997-2021 | CC-BY-4.0 | [OSC](https://opensciencedata.esa.int/products/antarctic-ice-shelf-melt-rates/collection) | `basal_melt/access.ipynb` |
| BedMachine Antarctica | Bed, surface, thickness, mask | Zarr | Antarctic polar stereographic, EPSG:3031 | Static product version | CC0-1.0 | [OSC](https://opensciencedata.esa.int/products/bedrock-topography-antarctica-bedmachine/collection) | `bedrock_topography/access.ipynb` |
| Calving fronts | Time-evolving coastlines/fronts | GeoParquet | Vector geometries, examples reproject to EPSG:3031 | Observation time series | CC-BY-4.0 | [OSC](https://opensciencedata.esa.int/products/antarctic-ice-shelf-calving-fronts/collection) | `calving_fronts/access.ipynb` |
| CS+AO sea ice thickness | Southern Ocean sea ice | Zarr | Polar grid with lon/lat coordinates | CryoSat-2/SARAL period in source product | CC-BY-NC-4.0 | [OSC](https://opensciencedata.esa.int/products/southern-ocean-sit-from-cryosat-2/collection) | `csao/access.ipynb` |
| Grounding lines | Antarctic grounding-line observations | GeoParquet | Vector geometries, examples reproject to EPSG:3031 | 1992-2014 | CC0-1.0 | [OSC](https://opensciencedata.esa.int/products/grounding-line-position-antarctica/collection) | `groundlines/access.ipynb` |
| Ice temperature profiles | Englacial temperature | Zarr, source NetCDF in format tutorials | Curvilinear source with lon/lat; examples reproject to EPSG:3031 | data is from 2013-2014 | CC-BY-4.0 | [OSC](https://opensciencedata.esa.int/products/ice-temperature-profiles-antarctica/collection) | `ice_temp/access.ipynb` |
| Ice velocity | Antarctic surface velocity | Zarr | Antarctic polar stereographic, EPSG:3031 | Monthly, 2014-2021 | CC-BY-NC-4.0 | [OSC](https://opensciencedata.esa.int/products/ice-sheet-velocity-antarctic-2021/collection) | `ice_velocity/access.ipynb` |
| Surface elevation change | Grounded ice surface elevation | Zarr | Antarctic polar stereographic, EPSG:3031 | 1991-2021 5-year means | CC-BY-4.0 | [OSC](https://opensciencedata.esa.int/products/sec-antarctic-ice-sheet/collection) | `sec/access.ipynb` |
| SOFRESH sea surface salinity | Southern Ocean salinity | Zarr | EASE-style Southern Ocean grid | 2011-2022, 9-day maps | CC-BY-4.0 | [OSC](https://opensciencedata.esa.int/products/sofresh-sea-surface-salinity/collection) | `sofresh/access.ipynb` |
| Subglacial lakes | Active lake boundaries | GeoParquet | Vector geometries, examples reproject to EPSG:3031 | 2010-2020 source analysis | CC-BY-4.0 | [OSC](https://opensciencedata.esa.int/products/subglacial-lakes-boundries/collection) | `subglacial_lakes/access.ipynb` |
| Supraglacial lakes | Surface hydrology features | GeoParquet | Vector geometries | January 2017 maximum extent | CC-BY-4.0 | [OSC](https://opensciencedata.esa.int/products/supraglacial-lakes-west-4dantarctica/collection) | `supraglacial_lakes/access.ipynb` |


# 2. Datacube

As part of the initiative, and to facilitate easier analysis of and access to scientific Antarctica data, some of the presented datasets are combined into zarr stores, reprojected to a common 100x100m grid in the EPSG:3031 Polar projection.

1. You can see how to access the data from the access notebook
2. You can see how we combined the presented from the .combine notebook. Primarily we used `nearest-neighbour` interpolation. The goal was to provide single access to all the data in one place. You can use that notebook to combine the different raw datasets.
3. You can see how to carry larger-scale analysis efficiently in the large_area_processing notebook.

You can see a visualisation of the data in eodash: https://esa-earthcode.github.io/polar-science-cluster-dashboard/




# 3. File formats and metadata

We have also prepared some notebooks that show how we transformed the data from various file formats into the three cloud-optimised formats we have used and how we generated the associated metadata.

## Dataset formats

All the original datasets have been converted to cloud-optimised formats. Primarily we used three formats

| Format | Best for | Why it matters in object storage |
| --- | --- | --- |
| Zarr | Multidimensional arrays such as time/depth/y/x cubes | Supports a flexible chunking scheme allows the indexing of data, optimised for analytics. Also, chunked reads let analysis load only the pieces it needs. |
| Cloud Optimized GeoTIFF (COG) | Single rasters or raster stacks exposed as files | Internal tiling and overviews enable efficient range reads and quicklooks. |
| GeoParquet | Vector geometries and tabular point data | Columnar storage, embedded geospatial metadata, and fast filtering. |

For more information for each, checkout the notebooks or other resources like : https://guide.cloudnativegeo.org/


## Metadata

For example datatype we also show how we genereated its associated metadata. Rich metadata ensures that the datasets are more accessible - https://esa-earthcode.github.io/documentation/Community%20and%20Best%20Practices/FAIR%20and%20Open%20Science%20Best%20Practices/Data.

All the metadata generated for the datasets and hosted on the Open Science Catalog is in STAC format. The SpatioTemporal Asset Catalog (STAC) specification was designed to establish a standard, unified language to talk about geospatial data, allowing it to be more easily searchable and queryable. For more information and a introduction to STAC, see https://esa-earthcode.github.io/tutorials/stac-and-data-access/ . 


# Suggested Hackathon Workflow

1. Pick a science question or user workflow.
2. Browse the dataset catalogue and open the relevant access notebooks.
3. Use the existing datasets and cube to answer the question
4. Use the file-format notebooks as guidence when you need to prepare a new datasets to be added to be combined with the existing datasets or cube.
5. Put your code in `Hackathon Code/` and keep generated data out of Git.
6. Submit your work as a pull request with a short description of the question, datasets, methods, and outputs.


## Environment Notes

The notebooks assume a Python geospatial environment with common Pangeo tools: `xarray`, `dask`, `geopandas`, `rasterio`, `rioxarray`, `pyproj`, `shapely`, `pandas`, `numpy`, `matplotlib`, and `pystac` for metadata examples.

Network access is only needed for remote object-store reads or optional source downloads. The format tutorials are written so downloaded examples land in `downloaded_data/` and can be regenerated instead of committed.
