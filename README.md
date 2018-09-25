
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

Python was set-up to run `python3`. This can be done at the system level
or, if you’re running these benchmarks from RStudio by adding the
following line to `.Renviron` (which can be edited with
`usethis::edit_r_environ()`):

    RETICULATE_PYTHON=/usr/bin/python3

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
piggyback::pb_upload(file = "ac-100K.geojson")
piggyback::pb_upload(file = "ac-10K.geojson")
```

## Download data

``` r
if(!file.exists("ac.geojson")) {
  download.file("https://github.com/ATFutures/geobench/releases/download/0.0.1/ac.geojson.zip", "ac.geojson.zip")
  unzip("ac.geojson.zip")
  download.file("https://github.com/ATFutures/geobench/releases/download/0.0.1/ac-100k.geojson", "ac-100k.geojson")
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

### 100K sample

This runs happily and quickly so the results are run each time:

``` r
bench::mark(iterations = 1, check = FALSE,
            sf = {ac_sf10k = sf::read_sf("ac-100K.geojson")},
            sp = {ac_sp10k = rgdal::readOGR("ac-100K.geojson", verbose = F)}
)
#> Warning: Some expressions had a GC in every iteration; so filtering is
#> disabled.
#> # A tibble: 2 x 10
#>   expression    min   mean median    max `itr/sec` mem_alloc  n_gc n_itr
#>   <chr>      <bch:> <bch:> <bch:> <bch:>     <dbl> <bch:byt> <dbl> <int>
#> 1 sf          4.02s  4.02s  4.02s  4.02s    0.249     46.5MB    15     1
#> 2 sp         13.86s 13.86s 13.86s 13.86s    0.0721      90MB     2     1
#> # ... with 1 more variable: total_time <bch:tm>
```

``` bash
pip3 install geopandas
pip3 install pytictoc
pip3 install rasterio
```

``` python
import sys
sys.version
import geopandas as gpd
from pytictoc import TicToc
t = TicToc()
t.tic()
s = gpd.read_file("ac-100K.geojson")
t.toc()
#> Elapsed time is 6.033249 seconds.
```

## Benchmark 2: spatial subsetting

Work in progress…

## System info

``` r
system("lscpu", intern = TRUE)
#>  [1] "Architecture:        x86_64"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        
#>  [2] "CPU op-mode(s):      32-bit, 64-bit"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                
#>  [3] "Byte Order:          Little Endian"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 
#>  [4] "CPU(s):              4"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
#>  [5] "On-line CPU(s) list: 0-3"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
#>  [6] "Thread(s) per core:  2"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
#>  [7] "Core(s) per socket:  2"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
#>  [8] "Socket(s):           1"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
#>  [9] "NUMA node(s):        1"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
#> [10] "Vendor ID:           GenuineIntel"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  
#> [11] "CPU family:          6"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
#> [12] "Model:               142"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
#> [13] "Model name:          Intel(R) Core(TM) i7-7500U CPU @ 2.70GHz"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      
#> [14] "Stepping:            9"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
#> [15] "CPU MHz:             2296.253"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      
#> [16] "CPU max MHz:         3500.0000"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     
#> [17] "CPU min MHz:         400.0000"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      
#> [18] "BogoMIPS:            5808.00"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       
#> [19] "Virtualisation:      VT-x"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          
#> [20] "L1d cache:           32K"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
#> [21] "L1i cache:           32K"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
#> [22] "L2 cache:            256K"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          
#> [23] "L3 cache:            4096K"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         
#> [24] "NUMA node0 CPU(s):   0-3"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
#> [25] "Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp"
```
