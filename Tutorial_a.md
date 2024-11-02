Rasterio Tutorial 

David Gerstenfeld, Adrian Terech

Month Day, 2024

This tutorial will showcase tools from the Rasterio library for Python.
Rasterio is a Python library designed for reading and writing geospatial raster data. It provides a high-level API to interact with raster datasets, particularly those stored in formats like GeoTIFF. Raster data typically represents satellite imagery, aerial photography, or any spatially continuous variable (e.g., elevation or temperature) as a grid of pixels or cells.

Key features of Rasterio include:

Reading and Writing GeoTIFFs: It can handle a variety of raster data formats (GeoTIFF, JPEG2000, etc.) with geographic metadata.
Geospatial Metadata Handling: Rasterio integrates well with the Geospatial Data Abstraction Library (GDAL), allowing access to spatial reference systems, projections, and other geospatial metadata.
Data Access: It enables easy reading of specific windows or blocks of large raster datasets without loading the entire file into memory.
NumPy Integration: The raster data can be loaded directly into NumPy arrays for efficient numerical operations.
Coordinate Reference Systems: Rasterio allows reading and transforming coordinate systems, making it easy to project raster data into different spatial reference systems.


*Analytical business need* = *We are doing an analysis for the City of Philadelphia Planning Department and Sustinability Office on heat island issues comapred to current land cover & tree canpoy rates across the city.*


*majority be in rasterio for processing of data, then geopanda for doing final statistical analysis, and then matplotlib for output of map*

**https://rasterio.readthedocs.io/en/stable/topics/index.html**

**https://www.mrlc.gov/viewer/**

Topics:

**Color**

**Georeferencing**

**Masking raster using shapefile**

**Reclassify**

**Raster Calculator**

**Vector to Raster Conversion**



NLCD_TreeCoverCanopy_PhiladelphiaRegion_2021.tiff



NLCD_LandCover_PhiladelphiaRegion_2021.tiff



**https://www.esri.com/arcgis-blog/products/product/analytics/deriving-temperature-from-landsat-8-thermal-bands-tirs/**

Band 10 data in this case, follow formulas

July and August data recommended

*******************************
ORIGINAL PROJECTIONS:

Land_Surface_Temp = UTM Projection, WGS 84 Datum

Land_Cover = PROJCS: Albers Conical Equal Area, WGS 84 Datum

Tree_Cover = PROJCS: Albers Conical Equal Area, WGS 84 Datum

PHL_Census_Tracts = GEOGCSN: GCS_North_American_1983 

Planning_Districts = WGS 84 Datum

*****************************




***********David to change raster tiff landsat to temperature and check out meta data for dates for that data and tree cover and land cover*************


*************
Links:

Land Cover and Tree Canopy Cover: https://www.mrlc.gov/viewer/
Downloaded using custom extent

Landsat Data: https://earthexplorer.usgs.gov/
Downloaded Band 10 dataset and metadata file. Band 10 data came in as raw pixel data, which had to converted to radiance, then to Kelvin, and then to Fahrenheit

Census Tracts: https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2021&layergroup=Census+Tracts

Planning Districts: https://opendataphilly.org/datasets/planning-districts/

*************




This how we are going to tell them to install rasterio using conda on anaconda powershell prompt

gus5031 = environment name

conda create -n gus5031 -c conda-forge pysal geopandas


*************
*now code rough draft

*next week make code finalized and work on rough draft of text

* last week make text finalized and powerpoint?
  
****************





import pysal

import os

from rasterio.enums import Resampling

import rasterio

#I set the current directory folder to the workspace below

workspace = os.getcwd()

overwriteOutput = True


downscale_factor = 1/2

*******************

Download data (3 datasets)

Resample so 

Make sure all same projections (projection)

Do some sort of analysis to datasets to make only philadelphia (masking to only philly)

Landsat data (change color* to reds?)

Clip datasets

Reclassify tree cover and land cover datasets

Use zonal statistics to compute average surface temperature in and without tree cover and in and without land cover uses

Use rasterio to create an output map with legend etc…

*******************


with rasterio.open("NLCD_LandCover_PhiladelphiaRegion_2021.tiff") as dataset:

    #resample data to target shape
    data = dataset.read(
        out_shape = (
            dataset.count,
            int(dataset.height * downscale_factor),
            int(dataset.width * downscale_factor)
        ),
        resampling=Resampling.bilinear
    )
    #scale image transform
    transform = dataset.transform * dataset.transform.scale(
        (dataset.width / data.shape[-1]), 
        (dataset.height / data.shape[-2])
    )

