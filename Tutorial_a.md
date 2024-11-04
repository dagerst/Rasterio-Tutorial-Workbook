Rasterio Tutorial 

David Gerstenfeld, Adrian Terech<br> Month Day, 2024

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

2272 = NAD83 State Plain Pennsylvania South


5070 = Albers Conical

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

*After masking to Philadelphia County we are then going to remove all data with zero values*


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

current script

    import pysal
    import os
    import geopandas as gpd
    import numpy as np
    import rasterio
    import fiona
    import subprocess
    import rasterio.mask
    import matplotlib.pyplot as plt
    from rasterio import Affine as A
    from rasterio.warp import calculate_default_transform, reproject, Resampling
    from rasterio.transform import from_origin
    from rasterio.enums import Resampling

#ABOVE IS ALL IMPORTING


    workspace = os.getcwd()
    overwriteOutput = True

    planning_dist = "Planning_Districts.shp"
    census_tracts = "PHL_Census_Tracts_2021.shp"
    land_surf_temp = "Land_Surface_Temperature_Landsat_2021.tif"
    land_cover = "NLCD_LandCover_PhiladelphiaRegion_2021.tif"
    tree_cover = "NLCD_TreeCoverCanopy_PhiladelphiaRegion_2021.tif"
    lst_prj = 'LST_2021.tif'
    lc_prj = 'LC_2021.tif'
    tcc_prj = 'TCC_2021.tif'
    census_prj = 'census_nad_83.shp'
    planning_prj = 'planning_nad_83.shp'


#VARIABLE NAMING
    
    src_crs = (5070)
    dst_crs = (2272)
#CODES FOR PROJECTIONS

#BELOW IS REPROJECTION OF LAND_COVER RASTER FILE

    with rasterio.open(land_cover) as src:
      source = src.read(1)
      src_transform = src.transform
      src_shape = source.shape

    dst_transform, dst_width, dst_height = calculate_default_transform(
        src_crs, dst_crs, src_shape[1], src_shape[0], *src.bounds)

    destination = np.zeros((dst_height, dst_width), dtype=source.dtype)

    
    reproject(
        source,
        destination,
        src_transform=src_transform,
        src_crs=src_crs,
        dst_transform=dst_transform,
        dst_crs=dst_crs,
        resampling=Resampling.nearest
    )

    with rasterio.open(
        lc_prj,
        'w',
        driver='GTiff',
        height=dst_height,
        width=dst_width,
        count=1,
        dtype=destination.dtype,
        crs=dst_crs,
        transform=dst_transform
    ) as dst:
        dst.write(destination, 1)




#BELOW IS REPROJECTION OF LAND_SURF_TEMP LANDSAT RASTER DATA FILE

    src_crs = (32618) 

    with rasterio.open(land_surf_temp) as src:
      source = src.read(1)
      src_transform = src.transform
      src_shape = source.shape

    dst_transform, dst_width, dst_height = calculate_default_transform(
        src_crs, dst_crs, src_shape[1], src_shape[0], *src.bounds)

    destination = np.zeros((dst_height, dst_width), dtype=source.dtype)
    
    reproject(
        source,
        destination,
        src_transform=src_transform,
        src_crs=src_crs,
        dst_transform=dst_transform,
        dst_crs=dst_crs,
        resampling=Resampling.nearest
    )

    with rasterio.open(
        lst_prj,
        'w',
        driver='GTiff',
        height=dst_height,
        width=dst_width,
        count=1,
        dtype=destination.dtype,
        crs=dst_crs,
        transform=dst_transform
    ) as dst:
        dst.write(destination, 1)



#BELOW IS REPROJECTION OF TREE_COVER RASTER DATA FILE

    src_crs = (5070)

    with rasterio.open(tree_cover) as src:
      source = src.read(1)
      src_transform = src.transform
      src_shape = source.shape

    dst_transform, dst_width, dst_height = calculate_default_transform(
        src_crs, dst_crs, src_shape[1], src_shape[0], *src.bounds)

    destination = np.zeros((dst_height, dst_width), dtype=source.dtype)

    reproject(
        source,
        destination,
        src_transform=src_transform,
        src_crs=src_crs,
        dst_transform=dst_transform,
        dst_crs=dst_crs,
        resampling=Resampling.nearest
    )

    with rasterio.open(
        tcc_prj,
        'w',
        driver='GTiff',
        height=dst_height,
        width=dst_width,
        count=1,
        dtype=destination.dtype,
        crs=dst_crs,
        transform=dst_transform
    ) as dst:
        dst.write(destination, 1)

#BELOW IS REPROJECTION OF CENSUS_TRACTS SHAPEFILE

    gdf = gpd.read_file(census_tracts)

    print("Original CRS:", gdf.crs)

    target_crs = "EPSG:2272"

    gdf_reprojected = gdf.to_crs(target_crs)

    print("New CRS:", gdf_reprojected.crs)

    output_shapefile = census_prj
    gdf_reprojected.to_file(census_prj, driver='ESRI Shapefile')


#BELOW IS REPROJECTION OF PLANNING_DISTRICT SHAPEFILE

    gdf = gpd.read_file(planning_dist)

    print("Original CRS:", gdf.crs)

    target_crs = "EPSG:2272"

    gdf_reprojected = gdf.to_crs(target_crs)

    print("New CRS:", gdf_reprojected.crs)

    output_shapefile = planning_prj
    gdf_reprojected.to_file(planning_prj, driver='ESRI Shapefile')


##################

    with fiona.open(census_prj, "r") as shapefile:
      shapes = [feature["geometry"] for feature in shapefile]

    with rasterio.open(lc_prj) as src:
      out_image, out_transform = rasterio.mask.mask(src, shapes, crop=True)
      out_meta = src.meta

    out_meta.update({"driver": "GTiff",
                 "height": out_image.shape[1],
                 "width": out_image.shape[2],
                 "transform": out_transform})

    with rasterio.open("land_cover_mask.tif", "w", **out_meta) as dest:
      dest.write(out_image)

    print(f'{lc_prj} has been masked.')



    with rasterio.open(lst_prj) as src:
      out_image, out_transform = rasterio.mask.mask(src, shapes, crop=True)
      out_meta = src.meta

    out_meta.update({"driver": "GTiff",
                 "height": out_image.shape[1],
                 "width": out_image.shape[2],
                 "transform": out_transform})

    with rasterio.open("land_surf_temp_mask.tif", "w", **out_meta) as dest:
      dest.write(out_image)

    print(f'{lst_prj} has been masked.')



    with rasterio.open(tcc_prj) as src:
      out_image, out_transform = rasterio.mask.mask(src, shapes, crop=True)
      out_meta = src.meta

    out_meta.update({"driver": "GTiff",
                 "height": out_image.shape[1],
                 "width": out_image.shape[2],
                 "transform": out_transform})

    with rasterio.open("tree_cover_mask.tif", "w", **out_meta) as dest:
      dest.write(out_image)

    print(f'{tcc_prj} has been masked.')



    output_path = 'lst_mask_color.tif'

    with rasterio.open('land_cover_mask.tif') as src:
      float_data = src.read(1)  # Read the first band
      meta = src.meta  # Get metadata for later use
  
    max_value = 90.0
    scaled_data = np.clip(float_data, 0, max_value
    scaled_data = (scaled_data / max_value * 255).astype(np.uint8) 

    meta.update(dtype=rasterio.uint8)

    with rasterio.open(output_path, 'w', **meta) as dst:
      dst.write(scaled_data, indexes=1)

    cmap = plt.get_cmap('coolwarm')  # Choose a suitable continuous colormap

    plt.show()

