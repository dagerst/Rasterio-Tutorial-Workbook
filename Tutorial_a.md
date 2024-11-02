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

    #import extensions
    import pysal
    import os
    import geopandas as gpd
    import numpy as np
    import rasterio
    import fiona
    import rasterio.mask
    from rasterio import Affine as A
    from rasterio.warp import calculate_default_transform, reproject, Resampling
    from rasterio.transform import from_origin
    from rasterio.enums import Resampling
    
    #Reprojected variables:
    lst_prj = 'LST_2021.tif'
    lc_prj = 'LC_2021.tif'
    census_prj = 'census_nad_83.shp'

    #  Specify the source and destination CRS
    src_crs = (5070)  # Albers Conical Equal Area (NAD83)
    dst_crs = (2272)  # NAD 1983 State Plane Pennsylvania South (EPSG:2272)

    # Load your land cover dataset
    with rasterio.open(land_cover) as src:
      source = src.read(1)  # Read the first band
      src_transform = src.transform
      src_shape = source.shape

    # Calculate the transform and shape for the destination
    dst_transform, dst_width, dst_height = calculate_default_transform(
        src_crs, dst_crs, src_shape[1], src_shape[0], *src.bounds)

    # Initialize the destination array
    destination = np.zeros((dst_height, dst_width), dtype=source.dtype)

    # Perform the reprojection
    reproject(
        source,
        destination,
        src_transform=src_transform,
        src_crs=src_crs,
        dst_transform=dst_transform,
        dst_crs=dst_crs,
        resampling=Resampling.nearest
    )

    # Optionally, save the reprojected dataset
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

    # Load the shapefile
    gdf = gpd.read_file(census_tracts)

    # Print the original CRS
    print("Original CRS:", gdf.crs)

    # Specify the target CRS (NAD 1983 State Plane Pennsylvania South)
    target_crs = "EPSG:2272"

    # Reproject the GeoDataFrame
    gdf_reprojected = gdf.to_crs(target_crs)

    # Print the new CRS
    print("New CRS:", gdf_reprojected.crs)

    # Save the reprojected shapefile
    output_shapefile = census_prj
    gdf_reprojected.to_file(output_shapefile, driver='ESRI Shapefile')

my script

    #import extensions
    import pysal
    import os
    import geopandas as gpd
    import numpy as np
    import rasterio
    import fiona
    import rasterio.mask
    from rasterio import Affine as A
    from rasterio.warp import calculate_default_transform, reproject, Resampling
    from rasterio.transform import from_origin
    from rasterio.enums import Resampling


    #I set the current directory folder to the workspace below
    workspace = os.getcwd()
    overwriteOutput = True

    planning_dist = "Planning_Districts.shp"
    census_tracts = "PHL_Census_Tracts_2021.shp"
    land_surf_temp = "Land_Surface_Temperature_Landsat_2021.tif"
    land_cover = "NLCD_LandCover_PhiladelphiaRegion_2021.tif"
    tree_cover = "NLCD_TreeCoverCanopy_PhiladelphiaRegion_2021.tif"
    lst_prj = 'LST_2021.tif'
    lc_prj = 'LC_2021.tif'
    census_prj = 'census_nad_83.shp'

    # Specify the source and destination CRS
    src_crs = (5070)  # Albers Conical Equal Area (NAD83)
    dst_crs = (2272)  # NAD 1983 State Plane Pennsylvania South (EPSG:2272)

    # Load your land cover dataset
    with rasterio.open(land_cover) as src:
      source = src.read(1)  # Read the first band
      src_transform = src.transform
      src_shape = source.shape

    # Calculate the transform and shape for the destination
    dst_transform, dst_width, dst_height = calculate_default_transform(
        src_crs, dst_crs, src_shape[1], src_shape[0], *src.bounds)

    # Initialize the destination array
    destination = np.zeros((dst_height, dst_width), dtype=source.dtype)

    # Perform the reprojection
    reproject(
        source,
        destination,
        src_transform=src_transform,
        src_crs=src_crs,
        dst_transform=dst_transform,
        dst_crs=dst_crs,
        resampling=Resampling.nearest
    )

    # Optionally, save the reprojected dataset
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

    # Load the shapefile
    gdf = gpd.read_file(census_tracts)

    # Print the original CRS
    print("Original CRS:", gdf.crs)

    # Specify the target CRS (NAD 1983 State Plane Pennsylvania South)
    target_crs = "EPSG:2272"

    # Reproject the GeoDataFrame
    gdf_reprojected = gdf.to_crs(target_crs)

    # Print the new CRS
    print("New CRS:", gdf_reprojected.crs)

    # Save the reprojected shapefile
    output_shapefile = census_prj
    gdf_reprojected.to_file(output_shapefile, driver='ESRI Shapefile')


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

    print('Raster has been masked.')


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

