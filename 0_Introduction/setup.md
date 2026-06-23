# Setup

## Working Locally

The notebooks assume a Python geospatial environment with common Pangeo tools: `xarray`, `dask`, `geopandas`, `rasterio`, `rioxarray`, `pyproj`, `shapely`, `pandas`, `numpy`, `matplotlib`, and `pystac` for metadata examples. You can install the pangeo-notebook environment from:

`conda create -n pangeo-notebook --file https://raw.githubusercontent.com/pangeo-data/pangeo-docker-images/2021.12.02/pangeo-notebook/conda-linux-64.lock`

Network access is only needed for remote object-store reads or optional source downloads. The format tutorials are written so downloaded examples land in `downloaded_data/` and can be regenerated instead of committed.

## Working on Polar TEP and EarthCODE Workspaces

If you are using the provided cloud platform environment these notebooks and environment will already have all needed packages installed. 

**You can directly start working in the environment. You can access this cloud environment during the week of the online pre-hackathon and the week after!**

To request an account (only available during the workshop!) click on the following and use your github account to sign in: https://workspace.polar-hackathon.hub-otc-sc.eox.at

NOTE: UPDATE THIS WITH NBGITPULL TO THIS REPO!