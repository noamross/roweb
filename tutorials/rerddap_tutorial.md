---
title: rerddap tutorial
layout: tutorial
packge_version: 0.3.0
---



`rerddap` is a general purpose R client for working with ERDDAP servers. ERDDAP is a server built on top of OPenDAP, which serves some NOAA data. You can get gridded data ([griddap](http://upwell.pfeg.noaa.gov/erddap/griddap/documentation.html)), which lets you query from gridded datasets, or table data ([tabledap](http://upwell.pfeg.noaa.gov/erddap/tabledap/documentation.html)) which lets you query from tabular datasets. In terms of how we interface with them, there are similarties, but some differences too. We try to make a similar interface to both data types in `rerddap`.

## netCDF

`rerddap` supports netCDF format, and is the default when using the `griddap()` function. netCDF is a binary file format, and will have a much smaller footprint on your disk than csv. The binary file format means it's harder to inspect, but the `ncdf` and `ncdf4` packages make it easy to pull data out and write data back into a netCDF file. Note the the file extension for netCDF files is `.nc`. Whether you choose netCDF or csv for small files won't make much of a difference, but will with large files.

## Caching

Data files downloaded are cached in a single hidden directory `~/.rerddap` on your machine. It's hidden so that you don't accidentally delete the data, but you can still easily delete the data if you like.

When you use `griddap()` or `tabledap()` functions, we construct a MD5 hash from the base URL, and any query parameters - this way each query is separately cached. Once we have the hash, we look in `~/.rerddap` for a matching hash. If there's a match we use that file on disk - if no match, we make a http request for the data to the ERDDAP server you specify.

## ERDDAP servers

You can get a data.frame of ERDDAP servers using the function `servers()`. Most I think serve some kind of NOAA data, but there are a few that aren't NOAA data.  If you know of more ERDDAP servers, send a pull request, or let us know.

<section id="installation">

## Installation

Stable version from CRAN


```r
install.packages("rerddap")
```

Or, the development version from GitHub


```r
devtools::install_github("ropensci/rerddap")
```


```r
library("rerddap")
```

<section id="usage">

## Usage

### Search

First, you likely want to search for data, specify either `griddadp` or `tabledap`


```r
ed_search(query = 'size', which = "table")
#> 11 results, showing first 20 
#>                                                                                          title
#> 1                                                                           CalCOFI Fish Sizes
#> 2                                                                         CalCOFI Larvae Sizes
#> 3                 Channel Islands, Kelp Forest Monitoring, Size and Frequency, Natural Habitat
#> 9                                                          CalCOFI Larvae Counts Positive Tows
#> 10                                                                                CalCOFI Tows
#> 316                                                 OBIS - ARGOS Satellite Tracking of Animals
#> 317                                                GLOBEC NEP Vertical Plankton Tow (VPT) Data
#> 318                                                    GLOBEC NEP MOCNESS Plankton (MOC1) Data
#> 319 AN EXPERIMENTAL DATASET: Underway Sea Surface Temperature and Salinity Aboard the Oleander
#> 321                            NWFSC Observer Fixed Gear Data, off West Coast of US, 2002-2006
#> 322                                 NWFSC Observer Trawl Data, off West Coast of US, 2002-2006
#>              dataset_id
#> 1      erdCalCOFIfshsiz
#> 2      erdCalCOFIlrvsiz
#> 3        erdCinpKfmSFNH
#> 9   erdCalCOFIlrvcntpos
#> 10       erdCalCOFItows
#> 316           aadcArgos
#> 317        erdGlobecVpt
#> 318       erdGlobecMoc1
#> 319            nodcPJJU
#> 321  nwioosObsFixed2002
#> 322  nwioosObsTrawl2002
```


```r
ed_search(query = 'size', which = "grid")
#> 311 results, showing first 20 
#>                                                                                                 title
#> 4                                                            AVHRR PATHFINDER L3 BSST 5DAY DAYTIME V5
#> 5                                                           AVHRR PATHFINDER L3 BSST DAILY DAYTIME V5
#> 6                                                          AVHRR PATHFINDER L3 SST DAILY NIGHTTIME V5
#> 7                                                          AVHRR PATHFINDER L3 BSST DAILY DAYTIME V51
#> 8                                                        AVHRR PATHFINDER L3 BSST DAILY NIGHTTIME V51
#> 11        ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0008) [time][eta_rho][xi_rho]
#> 12            ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0008) [time][eta_u][xi_u]
#> 13            ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0008) [time][eta_v][xi_v]
#> 14 ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0008) [time][s_rho][eta_rho][xi_rho]
#> 15  ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0008) [time][Nbed][eta_rho][xi_rho]
#> 16        ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0009) [time][eta_rho][xi_rho]
#> 17            ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0009) [time][eta_u][xi_u]
#> 18            ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0009) [time][eta_v][xi_v]
#> 19 ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0009) [time][s_rho][eta_rho][xi_rho]
#> 20  ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0009) [time][Nbed][eta_rho][xi_rho]
#> 21        ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0010) [time][eta_rho][xi_rho]
#> 22            ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0010) [time][eta_u][xi_u]
#> 23            ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0010) [time][eta_v][xi_v]
#> 24 ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0010) [time][s_rho][eta_rho][xi_rho]
#> 25  ROMS3.0 CBLAST2007 Ripples with SWAN-40m res (his case7 ar0fd 0010) [time][Nbed][eta_rho][xi_rho]
#>                 dataset_id
#> 4  nasa_jpl_ccfc_1654_21c4
#> 5  nasa_jpl_bef9_68a1_1fd5
#> 6  nasa_jpl_77ba_ec94_a5d6
#> 7  nasa_jpl_5533_5dee_222e
#> 8  nasa_jpl_c2a2_7085_780e
#> 11     whoi_cbae_ef31_0877
#> 12     whoi_61f3_6dac_36c1
#> 13     whoi_4eff_1b8e_513a
#> 14     whoi_4d06_1f87_db0b
#> 15     whoi_4849_5c78_58dc
#> 16     whoi_28c3_4a74_191a
#> 17     whoi_e627_874c_2b1b
#> 18     whoi_9183_ea0f_9417
#> 19     whoi_322d_c428_66b4
#> 20     whoi_5f75_6229_f722
#> 21     whoi_02c9_858d_bc77
#> 22     whoi_689d_0109_9280
#> 23     whoi_4f6e_439a_8e4b
#> 24     whoi_5451_efbd_60ac
#> 25     whoi_1ace_a9ee_f343
```

### Information

Then you can get information on a single dataset


```r
info('whoi_62d0_9d64_c8ff')
#> <ERDDAP info> whoi_62d0_9d64_c8ff 
#>  Dimensions (range):  
#>      time: (2012-06-25T01:00:00Z, 2015-10-14T00:00:00Z) 
#>      eta_v: (0, 334) 
#>      xi_v: (0, 895) 
#>  Variables:  
#>      bedload_Vsand_01: 
#>          Units: kilogram meter-1 s-1 
#>      bedload_Vsand_02: 
#>          Units: kilogram meter-1 s-1 
...
```

### griddap (gridded) data

First, get information on a dataset to see time range, lat/long range, and variables.


```r
(out <- info('noaa_esrl_027d_0fb5_5d38'))
#> <ERDDAP info> noaa_esrl_027d_0fb5_5d38 
#>  Dimensions (range):  
#>      time: (1850-01-01T00:00:00Z, 2014-05-01T00:00:00Z) 
#>      latitude: (87.5, -87.5) 
#>      longitude: (-177.5, 177.5) 
#>  Variables:  
#>      air: 
#>          Range: -20.9, 19.5 
#>          Units: degC
```

Then query for gridded data using the `griddap()` function


```r
(res <- griddap(out,
  time = c('2012-01-01', '2012-01-30'),
  latitude = c(21, 10),
  longitude = c(-80, -70)
))
#> <ERDDAP griddap> noaa_esrl_027d_0fb5_5d38
#>    Path: [~/.rerddap/648ed11e8b911b65e39eb63c8df339df.nc]
#>    Last updated: [2015-10-09 09:20:59]
#>    File size:    [0 mb]
#>    Dimensions (dims/vars):   [3 X 1]
#>    Dim names: time, latitude, longitude
#>    Variable names: CRUTEM3: Surface Air Temperature Monthly Anomaly
#>    data.frame (rows/columns):   [18 X 4]
#>                    time  lat   lon  air
#> 1  2012-01-01T00:00:00Z 22.5 -77.5   NA
#> 2  2012-01-01T00:00:00Z 22.5 -72.5   NA
#> 3  2012-01-01T00:00:00Z 22.5 -67.5   NA
#> 4  2012-01-01T00:00:00Z 17.5 -77.5 -0.1
#> 5  2012-01-01T00:00:00Z 17.5 -72.5   NA
#> 6  2012-01-01T00:00:00Z 17.5 -67.5 -0.2
#> 7  2012-01-01T00:00:00Z 12.5 -77.5  0.2
#> 8  2012-01-01T00:00:00Z 12.5 -72.5   NA
#> 9  2012-01-01T00:00:00Z 12.5 -67.5  0.3
#> 10 2012-02-01T00:00:00Z 22.5 -77.5   NA
#> ..                  ...  ...   ...  ...
```

The output of `griddap()` is a list that you can explore further. Get the summary


```r
res$summary
#> $filename
#> [1] "~/.rerddap/648ed11e8b911b65e39eb63c8df339df.nc"
#> 
#> $writable
#> [1] FALSE
#> 
#> $id
#> [1] 65536
#> ...
```

Get the dimension variables


```r
names(res$summary$dim)
#> [1] "time"      "latitude"  "longitude"
```

Get the data.frame (beware: you may want to just look at the `head` of the data.frame if large)


```r
res$data
#>                    time  lat   lon   air
#> 1  2012-01-01T00:00:00Z 22.5 -77.5    NA
#> 2  2012-01-01T00:00:00Z 22.5 -72.5    NA
#> 3  2012-01-01T00:00:00Z 22.5 -67.5    NA
#> 4  2012-01-01T00:00:00Z 17.5 -77.5 -0.10
#> 5  2012-01-01T00:00:00Z 17.5 -72.5    NA
#> 6  2012-01-01T00:00:00Z 17.5 -67.5 -0.20
#> 7  2012-01-01T00:00:00Z 12.5 -77.5  0.20
#> 8  2012-01-01T00:00:00Z 12.5 -72.5    NA
#> 9  2012-01-01T00:00:00Z 12.5 -67.5  0.30
#> 10 2012-02-01T00:00:00Z 22.5 -77.5    NA
#> 11 2012-02-01T00:00:00Z 22.5 -72.5    NA
#> 12 2012-02-01T00:00:00Z 22.5 -67.5    NA
#> 13 2012-02-01T00:00:00Z 17.5 -77.5  0.40
#> 14 2012-02-01T00:00:00Z 17.5 -72.5    NA
#> 15 2012-02-01T00:00:00Z 17.5 -67.5  0.20
#> 16 2012-02-01T00:00:00Z 12.5 -77.5  0.00
#> 17 2012-02-01T00:00:00Z 12.5 -72.5    NA
#> 18 2012-02-01T00:00:00Z 12.5 -67.5  0.32
```

### tabledap (tabular) data


```r
(out <- info('erdCalCOFIfshsiz'))
#> <ERDDAP info> erdCalCOFIfshsiz 
#>  Variables:  
#>      calcofi_species_code: 
#>          Range: 19, 1550 
#>      common_name: 
#>      cruise: 
#>      fish_1000m3: 
#>          Units: Fish per 1,000 cubic meters of water sampled 
#>      fish_count: 
#>      fish_size: 
...
```


```r
(dat <- tabledap(out, 'time>=2001-07-07', 'time<=2001-07-10', fields = c('longitude', 'latitude', 'fish_size', 'itis_tsn', 'scientific_name')))
#> <ERDDAP tabledap> erdCalCOFIfshsiz
#>    Path: [~/.rerddap/f013f9ee09bdb4184928d533e575e948.csv]
#>    Last updated: [2015-10-09 09:21:00]
#>    File size:    [0.03 mb]
#>    Dimensions:   [558 X 5]
#> 
#>     longitude  latitude fish_size itis_tsn       scientific_name
#> 2  -118.10667 32.738335      31.5   623625  Lipolagus ochotensis
#> 3  -118.10667 32.738335      48.3   623625  Lipolagus ochotensis
#> 4  -118.10667 32.738335      15.5   162221 Argyropelecus sladeni
#> 5  -118.10667 32.738335      16.3   162221 Argyropelecus sladeni
#> 6  -118.10667 32.738335      17.8   162221 Argyropelecus sladeni
#> 7  -118.10667 32.738335      18.2   162221 Argyropelecus sladeni
#> 8  -118.10667 32.738335      19.2   162221 Argyropelecus sladeni
#> 9  -118.10667 32.738335      20.0   162221 Argyropelecus sladeni
#> 10 -118.10667 32.738335      21.0   162221 Argyropelecus sladeni
#> 11 -118.10667 32.738335      21.5   162221 Argyropelecus sladeni
#> ..        ...       ...       ...      ...                   ...
```

Since both `griddap()` and `tabledap()` give back data.frame's, it's easy to do downstream manipulation. For example, we can use `dplyr` to filter, summarize, group, and sort:


```r
library("dplyr")
dat$fish_size <- as.numeric(dat$fish_size)
tbl_df(dat) %>%
  filter(fish_size > 30) %>%
  group_by(scientific_name) %>%
  summarise(mean_size = mean(fish_size)) %>%
  arrange(desc(mean_size))
#> Source: local data frame [20 x 2]
#> 
#>                 scientific_name mean_size
#>                           (chr)     (dbl)
#> 1       Idiacanthus antrostomus 253.00000
#> 2            Stomias atriventer 189.25000
#> 3            Lestidiops ringens  98.70000
#> 4     Tarletonbeania crenularis  56.50000
#> 5      Ceratoscopelus townsendi  53.70000
#> 6     Stenobrachius leucopsarus  47.74538
#> 7               Sardinops sagax  47.00000
#> 8         Nannobrachium ritteri  43.30250
#> 9         Bathylagoides wesethi  43.09167
#> 10         Vinciguerria lucetia  42.00000
#> 11       Cyclothone acclinidens  40.80000
#> 12         Lipolagus ochotensis  39.72500
#> 13        Leuroglossus stilbius  38.35385
#> 14        Triphoturus mexicanus  38.21342
#> 15                Diaphus theta  37.88571
#> 16       Trachipterus altivelis  37.70000
#> 17 Symbolophorus californiensis  37.66000
#> 18         Nannobrachium regale  37.50000
#> 19         Merluccius productus  36.61333
#> 20        Argyropelecus sladeni  32.43333
```


<section id="citing">

## Citing

To cite `rerddap` in publications use:

<br>

>  Scott Chamberlain (2015). rerddap: General Purpose Client for 'ERDDAP' Servers. R
  package version 0.3.0. https://github.com/ropensci/rerddap

<section id="license_bugs">

## License and bugs

* License: [MIT](http://opensource.org/licenses/MIT)
* Report bugs at [our Github repo for rerddap](https://github.com/ropensci/rerddap/issues?state=open)

[Back to top](#top)
