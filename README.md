
<!-- README.md is generated from README.Rmd. Please edit that file -->

# gdalio

<!-- badges: start -->
<!-- badges: end -->

The goal of gdalio is to read data direct with GDAL warp, with an
assumed grid specification.

## Installation

You can install the released version of gdalio from
[CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("gdalio")
```

## Example

These equivalent functions format the data into objects used by various
packages.

``` r
library(gdalio)


## spatstat
gdalio_im <- function(dsn, ...) {
   v <- gdalio_data(dsn, ...)
g <- gdalio_get_default_grid()
## can we have a list of im?
   if (length(v) > 1) message("only returning one image layer im, for now")
   m <- matrix(v[[1]], g$dimension[1])
  spatstat.geom::im(t(m[,ncol(m):1]), xrange = g$extent[1:2], yrange = g$extent[3:4])
 
}

## raster
gdalio_raster <- function(dsn, ...) {
   v <- gdalio_data(dsn, ...)
g <- gdalio_get_default_grid()
  r <- raster::raster(raster::extent(g$extent), nrows = g$dimension[2], ncols = g$dimension[1], crs = g$projection)
  if (length(v) > 1) {
    r <- raster::brick(replicate(length(v), r, simplify = FALSE))
  }
  raster::setValues(r, matrix(unlist(v), prod(g$dimension)))
 }

## terra

gdalio_terra <- function(dsn, ...) {
   v <- gdalio_data(dsn, ...)
g <- gdalio_get_default_grid()
   r <- terra::rast(terra::ext(g$extent), nrows = g$dimension[2], ncols = g$dimension[1], crs = g$projection)
   if (length(v) > 1) terra::nlyr(r) <- length(v)
   terra::setValues(r, matrix(unlist(v), prod(g$dimension)))
 }

## stars

gdalio_stars <- function(dsn, ...) {
   v <- gdalio_data(dsn, ...)
   g <- gdalio_get_default_grid()

   r <- stars::st_as_stars(array(unlist(v), c(g$dimension[1], g$dimension[2], length(v)))[,g$dimension[2]:1,],
                           xlim = g$extent[1:2], ylim = g$extent[3:4])
   r <- sf::st_set_crs(r, g$projection)
   
   r
 }

gdalio_set_default_grid(list(extent = c(-1e6, 1e6, -5e5, 5e5 ), dimension = c(1018, 512), projection = "+proj=laea +lon_0=147 +lat_0=-42"))
f <- "NETCDF:\"/vsicurl/https://www.ncei.noaa.gov/data/sea-surface-temperature-optimum-interpolation/v2.1/access/avhrr/198403/oisst-avhrr-v02r01.19840308.nc\":sst"

## this source doesn't know its projection so we augment by passing that in
plot(gdalio_stars(f, source_wkt = "+proj=longlat"))
#> downsample set to c(1,1)
```

<img src="man/figures/README-example-1.png" width="100%" />

``` r
raster::plot(gdalio_raster(f, source_wkt = "+proj=longlat"), col = hcl.colors(26))
```

<img src="man/figures/README-example-2.png" width="100%" />

``` r
terra::plot(gdalio_raster(f, source_wkt = "+proj=longlat"))
```

<img src="man/figures/README-example-3.png" width="100%" />

``` r
plot(gdalio_im(f, source_wkt = "+proj=longlat"))
```

<img src="man/figures/README-example-4.png" width="100%" />

Say we don’t set a grid at all, just go with default. We can’t help that
OISST is 0,360 but that’s irrelevant.

``` r
gdalio_set_default_grid()
f <- "NETCDF:\"/vsicurl/https://www.ncei.noaa.gov/data/sea-surface-temperature-optimum-interpolation/v2.1/access/avhrr/198403/oisst-avhrr-v02r01.19840308.nc\":sst"
plot(gdalio_stars(f, source_wkt = "+proj=longlat"))
```

<img src="man/figures/README-default-1.png" width="100%" />

``` r
raster::plot(gdalio_raster(f, source_wkt = "+proj=longlat"), col = hcl.colors(26))
```

<img src="man/figures/README-default-2.png" width="100%" />

``` r
terra::plot(gdalio_raster(f, source_wkt = "+proj=longlat"))
```

<img src="man/figures/README-default-3.png" width="100%" />

``` r
plot(gdalio_im(f, source_wkt = "+proj=longlat"))
```

<img src="man/figures/README-default-4.png" width="100%" />

Some sources, files in spData, image servers, etc.

``` r
"NETCDF:\"/vsicurl/https://www.ncei.noaa.gov/data/sea-surface-temperature-optimum-interpolation/v2.1/access/avhrr/198403/oisst-avhrr-v02r01.19840308.nc\":anom"
```

## Code of Conduct

Please note that the gdalio project is released with a [Contributor Code
of
Conduct](https://contributor-covenant.org/version/2/0/CODE_OF_CONDUCT.html).
By contributing to this project, you agree to abide by its terms.
