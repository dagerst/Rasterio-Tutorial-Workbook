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








***********David to change raster tiff landsat to temperature and check out meta data for dates for that data and tree cover and land cover*************









This how we are going to tell them to install rasterio using conda on anaconda powershell prompt

gus5031 = environment name

conda create -n gus5031 -c conda-forge pysal geopandas
