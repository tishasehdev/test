---
title: "Processing Satellite Images with R"
author: "Tisha Sehdev"
date: `r Sys.time()`
output: html_document
---

This is a compilation of processing satellite imagery in an open source environment 'R'. The idea is to diminish difficulty in gaining acess to propriety based softwares like ERDAS, ENVI etc. With the advancements in satellites and remote sensing capibilities, satellite imagery offer great utility in terms of acquiring information about geographies unknown and at times inaccessible that too all from your desktop/laptop. If embeded in the infrastructure of organisations into monitoring and assessment, offers as a great tool/technology in closely monitoring the land use changes. 

Another plausible attribute is resolution and frequency at which satellite data is collected these days. Open source imagery from LandSat Series of USGS-NASA [link](http://landsat.usgs.gov/LandsatLook_Viewer.php) offers free downloadable imagery with resolution of 30m X 30m and as recent as last month with at times bi-monthly updation. There are other plenty sources even private vendors like Quickbird, DigitalGlobe capturing images with 2.5m,1m and even 0.61 cm of resolution.

In this tutorial, we will constantly focus to acquiant the user with the satellite data terminology, attributes, basic processing and converting it into usable piece of information. Later touch up on few applications as well.

### Understanding & Reading Satellite Imagery

Any satellite essentialy captures radiance/reflectance in different wavelengths of the spectra ranging from visible to microwaves.[link](http://imagine.gsfc.nasa.gov/science/toolbox/emspectrum1.html). What we obtain as data in the image much like a photograph is reflectance of sunlight/optical rays in these different spectral bands reflected by land features like buildings, water body, vegetation , barren surface etc. Except that in photographs, reflectance is captured only in the visible (RGB) spectra.

Landsat 8 imagery consists of nine spectral bands with a spatial resolution of 30 meters for Bands 1 to 7 and 9. New band 1 (ultra-blue) is useful for coastal and aerosol studies. Band 9 is useful for cirrus cloud detection. The resolution for Band 8 (panchromatic) is 15 meters. Thermal bands 10 and 11 are useful in providing more accurate surface temperatures and are collected at 100 meters

For someone who has worked with flat files or non-spatial data, imagine an image as a 2D matrix with values representing radiance and each cell representing an area on the earth having dimensions equivalent to the resolution of the satellite in this case 30m X 30m. An Image is a continuous grid of equal dimenions representing some area on the earth's surface and its feature.

Without much adieu, lets begin by reading 7 bands of Landsat Imagery downloaded and unzipped. I find two packages very handy - raster & readGDAL for raster/image related processing. Raster is essentially the data structure in which an image is read. 
Following are the Band designations:
**Band 1**: Ultra Blue, **Band 2**: Blue, **Band 3**: Green, **Band 4**: Red, **Band 5**: Near-Infrared, **Band 6**: Short-wave Infrared 1, **Band 7**: Short-wave Infrared 2


```{r,warning=FALSE, message=FALSE}
library(rgdal)
library(raster)
path <- "C:/Users/sehdti01/Documents/Projects/GIS Data/Raster/LandSat2015/"
setwd(path)
ls.B1 <- readGDAL("LC81440512015013LGN00_B1.tif")
ls.B2 <- readGDAL("LC81440512015013LGN00_B2.tif")
ls.B3 <- readGDAL("LC81440512015013LGN00_B3.tif")
ls.B4 <- readGDAL("LC81440512015013LGN00_B4.tif")
ls.B5 <- readGDAL("LC81440512015013LGN00_B5.tif")
ls.B6 <- readGDAL("LC81440512015013LGN00_B6.tif")
ls.B7 <- readGDAL("LC81440512015013LGN00_B7.tif")
```

Note that there are **7721 rows** and **7551 columns** in all the 7 bands (B1, B2, B3, B4, B5, B6 & B7) of the Landsat imagery and have been read as ls.B1,ls.B2,ls.B3,ls.B4,ls.B5,ls.B6 and ls.B7 objects/variables respectively. 

Lets inspect the the class of any one of the bands of the imagery and plot to see it in the map format.
```{r,warning=FALSE, message=FALSE}
class(ls.B5)
slotNames(ls.B5)
```
It is a 'SpatialGridDataFrame' with "data", "grid", "bbox" and "proj4string" as slots consisting metadata of the image. Let us inspect each one of them by using '@'.

```{r,warning=FALSE, message=FALSE}
summary(ls.B5@data)
```
Gives the summary statistics of the  digital numbers store in the slot 'DATA' in Band 5 of the image.

```{r,warning=FALSE, message=FALSE}
str(ls.B5@grid)
ls.B5@grid@cellcentre.offset
ls.B5@grid@cellsize
ls.B5@grid@cells.dim
```
Gives the string description of the slot 'GRID' and further consists of three slots : cellcentre.offset ("lower left cell centre coordinates"), cellsize (resolution of each pixel in the image) and cells.dim (the rows correspond
to the spatial dimensions).

```{r,warning=FALSE, message=FALSE}
ls.B5@bbox
```
Would consist information on the bounding box of the image- minimum and maximum of latitude (y) and longitude(x). X and Y are in metres as the landsat imagery is obtained in projected coordinate system (Universal Transverse Mercator).[link](https://en.wikipedia.org/wiki/Universal_Transverse_Mercator_coordinate_system).

```{r,warning=FALSE, message=FALSE}
ls.B5@proj4string
```

Consists string description (CRS Arguements - Coordinate Reference System) of the  Coordinate System: UTM with zone 43.

### Visualizing Satellite Imagery

```{r,warning=FALSE, message=FALSE}
ls.B5.1 <- raster(ls.B5)
plot(ls.B5.1,
     main="Near Infrared Band of the Landsat Image")
```

This is a general plot of the image/scene covering approx. 185 km (115 miles) wide swath on the earth's surface with 30 m resolution. The legend represents the DN(Digital number) value of the reflectance captured in the near-infrared band.

#### Plotting data using breaks

We can visualise the image color coded according to the range of values rather than the continuous colour gradient as in the previous attempt. This will be comparable to classified maps. To create breaks, we would need to closely examine the distribution of th data using a histogram. 
  
```{r,warning=FALSE, message=FALSE}
ls.B5hist <- hist(ls.B5@data$band1,
                 breaks =4,
                 main = "Frequecy Distribution of pixels in\n Near-Infrared Band (NIR)",
                 col = "wheat3",
                 xlab = "Reflectance in NIR (DN values)")
```

Examining the breaks and number of pixels in each category.

```{r,warning=FALSE, message=FALSE}
ls.B5hist$breaks
ls.B5hist$counts
```

So R has binned the data in 4 categories (fromm 0 to 50000) and as we can see the maximum number of pixels exist in 10000 - 20000 bracket. We can use this understanding and bins to plot our raster data. We will use the function terrain.colors() function to create pallete of 6 colors to plot the raster.

```{r,warning=FALSE, message=FALSE}
plot(ls.B5.1, 
     breaks = c(0,10000, 15000,30000,50000), 
     col = terrain.colors(4),
     main="Reflectance in NIR (DN values)")
     
```














