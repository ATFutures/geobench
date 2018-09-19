
<!-- README.md is generated from README.Rmd. Please edit that file -->

geobench is a place to put data (in the releases) and code (in any
language) to test performance.

It relies on the `bench` package for testing and datasets uploaded into
the releases.

## Pre-requisites

This section lists the software that needs to be installed to run these
benchmarks. It will likely evolve over time.

### R packages

  - `sf`

## Set-up

The code in this section only needs to run once, or at least each time a
new dataset is added to the benchmarks.

### Large point dataset covering UK

``` r
stplanr::dl_stats19()
ac = stplanr::read_stats19_ac() # 214 mb compressed data
ac = ac[!is.na(ac$Latitude), ]
ac_sf = sf::st_as_sf(ac[ c("Accident_Severity", "Longitude", "Latitude")], coords = c("Longitude", "Latitude"))
sf::write_sf(ac_sf, "ac.geojson") # 228 MB .geojson file
zip("ac.geojson.zip", "ac.geojson") # 14 MB
piggyback::pb_upload("ac.geojson.zip")
```
