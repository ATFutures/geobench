
<!-- README.md is generated from README.Rmd. Please edit that file -->

geobench is a place to put data (in the releases) and code (in any
language) to test performance.

It relies on the `bench` package for testing and datasets uploaded into
the releases.

## Pre-requisites

This section lists the software that needs to be installed to run these
benchmarks. It will likely evolve over time.

### R packages

  - `bench`
  - `sf`
  - `sp`
  - `spData`

## Set-up

The code in this section only needs to run once, or at least each time a
new dataset is added to the benchmarks.

### Large point dataset covering UK

``` r
stplanr::dl_stats19()
ac = stplanr::read_stats19_ac() # 214 mb compressed data
ac = ac[!is.na(ac$Latitude), ]
ac_sf = sf::st_as_sf(ac[ c("Accident_Severity", "Longitude", "Latitude")], coords = c("Longitude", "Latitude"))
sf::write_sf(ac_sf, "ac.geojson")               # 228 MB .geojson file
sf::write_sf(ac_sf[1:1e6, ], "ac-M.geojson")    # 139 MB .geojson file
sf::write_sf(ac_sf[1:1e5, ], "ac-100K.geojson") # 14 MB .geojson file
sf::write_sf(ac_sf[1:1e4, ], "ac-10K.geojson")  # 1 MB .geojson file
sf::write_sf(ac_sf[1:1e3, ], "ac-1K.geojson")   # 0.1 MB .geojson file
zip("ac.geojson.zip", "ac.geojson") # 14 MB
piggyback::pb_upload(file = "ac.geojson.zip")
piggyback::pb_upload(file = "ac-10K.geojson")
```

## Download data

``` r
if(!file.exists("ac.geojson")) {
  download.file("https://github.com/ATFutures/geobench/releases/download/0.0.1/ac.geojson.zip", "ac.geojson.zip")
  unzip("ac.geojson.zip")
  download.file("https://github.com/ATFutures/geobench/releases/download/0.0.1/ac-10k.geojson", "ac-10k.geojson")
}
```

## Benchmark 1: reading data

### Full dataset

Took too long to run and consumed too much memory to evaluate each time.
Results from first run pasted below:

``` r
bench::mark(iterations = 1, check = FALSE,
            {ac_sf = sf::read_sf("ac.geojson")},
            {ac_sp = rgdal::readOGR("ac.geojson")}
)
#> expression     min    mean   median     max `itr/sec` mem_alloc  
#>   <chr>      <bch:t> <bch:t> <bch:tm> <bch:t>     <dbl> <bch:byt>       
#> 1 {...         36.7s   36.7s    36.7s   36.7s   0.0272    655.6MB 
#> 2 {...            2m      2m       2m      2m   0.00835     1.2GB 
```

### 10k sample

This runs happily and quickly so the results are run each time:

``` r
bench::mark(iterations = 1, check = FALSE,
            sf = {ac_sf10k = sf::read_sf("ac-10k.geojson")},
            sp = {ac_sp10k = rgdal::readOGR("ac-10k.geojson", verbose = F)}
)
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
#> # A tibble: 2 x 10
#>   expression   min  mean median   max `itr/sec` mem_alloc  n_gc n_itr
#>   <chr>      <bch> <bch> <bch:> <bch>     <dbl> <bch:byt> <dbl> <int>
#> 1 sf         237ms 237ms  237ms 237ms     4.23     12.2MB     1     1
#> 2 sp            1s    1s     1s    1s     0.995    23.4MB     0     1
#> # ... with 1 more variable: total_time <bch:tm>
```

``` bash
pip install fiona
pip install pytictoc
```

``` python
import fiona
from pytictoc import TicToc
t = TicToc()
t.tic()
s = fiona.open("ac-10k.geojson")
t.toc()
#> Elapsed time is 0.094538 seconds.
```

## Benchmark 1: spatial subsetting

Work in progress…

``` r
library(sf)
#> Linking to GEOS 3.6.2, GDAL 2.2.3, proj.4 4.9.3
library(sp)
```
