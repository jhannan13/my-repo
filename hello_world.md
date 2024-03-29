Interactive Census Data
================
Jake Hannan
September 21, 2019

### Overview

The US Census Bureau publishes a wealth of aggregated metrics for several geographic boundaries (e.g., zip codes, census block groups, census tracts) - this information can be used by anyone.

Additionally, in order to visualize this data, the US Census Bureau offers spatial files at many levels of granularity (see <https://www.census.gov/programs-surveys/geography.html> for more). The `tigris` package is what we'll start with to find some county level data, and also use `spdplyr` to do some basic filtering on the data frame.

``` r
require(tigris)
require(spdplyr)
require(ggplot2)
#get US Counties using `tigris`
us_counties <- counties(state = NULL, year = 2017)
#filter areas not interested via `spdplyr`
us_counties <- us_counties %>%
  filter(!STATEFP %in% c("02", "15", "66", "72", "69", "78", "60"))
#check our work!
plot(us_counties)
```

![](hello_world_files/figure-markdown_github/tigris_map-1.png)

Nice! It looks like we've created the desired outcome so far: a map of US counties we wish to learn more about. Now that we have an SPDF, or SpatialPolygonsDataFrame, it's time to get some data to visualize.

### The Census API

In order to aptly query the Census Bureau's robust dataset, we'll need to leverage `censusapi`. This package is rather intuitive, and the only additional step needed is to get an API key from the bureau (you can do that [here](https://api.census.gov/data/key_signup.html)). Once you have your key, you can store it as a value like so --

`my_census_key <- ("insert_key_here")`

Once your key is generated and stored in your R session, you'll call it in each operation to get data. From there, it will be important to famililarize yourself with the function structure. The call below is very basic, and the comments on the code should walk you through which each does.

``` r
require(censusapi)
census_sample <- getCensus(name = "acs/acs5", #type of survey data
                         vintage = 2017, #year we seek to get data on
                         key = my_census_key, #as we mentioned earlier
                         vars = c("B19001_001E"), #number of homes for this sample
                         region = "county:*") #all counties
```

``` r
require(tidyverse)
require(dplyr)
census_sample <- as_tibble(census_sample)
#rename var to something more easily readable
census_sample <- census_sample %>%
  rename(households = B19001_001E)
#check total number of homes
census_sample %>%
  summarise(total = sum(households))
```

    ## # A tibble: 1 x 1
    ##       total
    ##       <dbl>
    ## 1 120048527

120MM is the right number of homes we're looking for within the year's survey, so this looks good! Now we'll need to join this data back to our SPDF, in order to visualize and interact with it.

``` r
#leverage GEOID as key to join
census_sample$GEOID <- paste0(census_sample$state,census_sample$county, sep="")
#join df
county_census_spdf <- left_join(x = us_counties, y = census_sample, by = "GEOID")
```
