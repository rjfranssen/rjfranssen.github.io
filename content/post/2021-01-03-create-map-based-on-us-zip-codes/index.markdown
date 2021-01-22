---
title: US Zip Code Choropleth
author: rjfranssen
date: '2021-01-03'
slug: create-map-based-on-us-zip-codes
categories:
  - geography
tags:
  - r
  - leaflet
  - gis
header:
  caption: ''
  image: ''
  preview: yes
---





2021-01-21 | _10 min_

First, it's important to recognize that Zip Codes are not polygons - they are a way for the US Post Office to associate mailing addresses with a particular post office or metro area delivery station, like a lookup table. However, the US Census Bureau created the ["Zip Code Tabulation Area" (ZCTA)](https://www.census.gov/programs-surveys/geography/guidance/geo-areas/zctas.html) to generalize an areal representation of Zip Code service areas. This post demonstrates how to create a choropleth map using ZCTAs, R, and the [leaflet](https://rstudio.github.io/leaflet/) library. _I originally [posted this](https://www.reddit.com/r/rprogramming/comments/ifnfju/create_a_map_based_off_of_us_postal_zip_codes/) to reddit._



```r
library(rgdal)
library(leaflet)
library(dplyr)
library(data.table)
library(htmlwidgets)
```


### Download the ZCTA Shapefile

Spatial polygons come in various formats - I tend to work with shapefiles and geojsons the most. For this example, I'm downloading the ZCTA shapefile from the U.S. Census Bureau website. I'm using the `cb_2018_us_zcta510_500k.zip [59 MB]` file found here: **[Cartographic Boundary Files - Shapefile](https://www.census.gov/geographies/mapping-files/time-series/geo/carto-boundary-file.html)**. There are other useful US shapefiles here, too, that include county, state, and metropolitan polygons.

Note the [tidycensus](https://walker-data.com/tidycensus/) package has great wrapper functions to pull down data from the US Census Bureau website, but I'm not doing that here because I want some code that I can use for other shapefiles.

![](zcta_capture.PNG)


### Unzip the ZCTA Shapefile

Unzip the shapefile you downloaded and put it in your working directory. This folder represents the entire shapefile.

![](zcta_unzipped.PNG)

If you open the shapefile folder, you'll see it's composed of several files that describe its projection and shape. [Go here](https://www.loc.gov/preservation/digital/formats/fdd/fdd000280.shtml) to understand what each file does. For our purposes, we'll be reading it in at the folder level.

![](zcta_insides.PNG)

### Read it into the R session

Now using the `rgdal` library to read it in as a `Large SpatialPolygonsDataFrame`.


```r
zcta_shapefile <-  rgdal::readOGR(
  dsn = "cb_2018_us_zcta510_500k", # this is the unzipped folder
  layer = "cb_2018_us_zcta510_500k", # this is the file inside of the unzipped folder
  verbose = FALSE
)
```


### Create test data

Maybe you have a spreadsheet of addressed or a table that you would import here. I'm going to create some fake numbers just for this exercise.

TODO: Use a real dataset :)


```r
# identify Alabama zip codes
al_zips <- 35004:36925

# get some random numbers
sales <- runif(length(al_zips), min=1000, max=5000) %>% round(0)

# put this in a dataframe
sales_data <- data.frame(
  state = 'AL',
  zip = al_zips,
  sales = sales
)

# ok, now i have some fake sales numbers
sales_data %>% head(5)
##   state   zip sales
## 1    AL 35004  2454
## 2    AL 35005  2972
## 3    AL 35006  2094
## 4    AL 35007  4973
## 5    AL 35008  3773
```


### Join data

Join the test data to your SpatialPolygonsDataFrame "data" slot. Again, this will look a bit different if you're using `tidycensus` and the `Simple Features (sf)` package (which is easier but not always an option with legacy data).

I tend to copy the data slot first so I can explore it easier in RStudio and then put it back in. But be careful not to duplicate or drop any rows in your join or it'll get messed up. Again, this is easier in `sf`...


```r
# copy data
zcta_data <- data.table::copy(zcta_shapefile@data)

#  join sales data - it's very important that we do NOT  duplicate or drop rows or it's going to screw up the shapefile
zcta_data$ZCTA5CE10 <- as.numeric(zcta_data$ZCTA5CE10)
zcta_data <- zcta_data %>% left_join(sales_data, by = c("ZCTA5CE10" = "zip"))

# Now reattach this data file back to the SpatialPolygonsDataFrame data slot
zcta_shapefile@data <- zcta_data

# (optional) There are 33k ZCTAs in the US; consider reducing these to a particular region of interest.
# I'm using the `state` value that i just joined for demo purposes. You can skip this if you want to do the whole country
zcta_shapefile <- zcta_shapefile[!is.na(zcta_shapefile@data$state) & zcta_shapefile@data$state=='AL',]
```

Check these - they should match; if they don't, make sure the join isn't duplicating or dropping anything.


```r
nrow(zcta_shapefile@data) == length(zcta_shapefile@polygons)
## [1] TRUE
nrow(zcta_shapefile@data)
## [1] 642
length(zcta_shapefile@polygons)
## [1] 642
```


### Mapping

Now going to do a couple of things to prepare for the interactive map, statting ith creating map labels that will appear when someone hovers over a polygon.


```r
labels <- sprintf(
  "<strong>Zip: %s</strong><br/> Sales: $%s",
  zcta_shapefile@data$ZCTA5CE10,
  prettyNum(zcta_shapefile@data$sales, big.mark=",")
) %>% lapply(htmltools::HTML)
```

Next, going to create a color palette and specify the sales value for the domain over which the choropleth gets shaded.


```r
map_pal <-  leaflet::colorNumeric(palette="viridis", domain = zcta_shapefile@data$sales, na.color="transparent")
```

Like I said, 33k polygons is A LOT. If you're going to plot them all, there's some options below to help our with processing (e.g. `preferCanvas = TRUE`, `updateWhenZooming = FALSE`, `updateWhenIdle = TRUE`). Otherwise, consider paring this down to an area of interest as shown in the optional step above.


```r
zcta_map <- zcta_shapefile %>% 
  
  leaflet::leaflet(options = leafletOptions(preferCanvas = TRUE)) %>%
  
  # Check out map providers here: https://leaflet-extras.github.io/leaflet-providers/preview/
  addProviderTiles(providers$Esri.WorldStreetMap, options = providerTileOptions(
    updateWhenZooming = FALSE,      # map won't update tiles until zoom is done
    updateWhenIdle = TRUE           # map won't load new tiles when panning
  )) %>%
  
  # Alabama zip 35148
  setView(lat = 33.7492174, lng = -87.0823462, zoom = 7) %>%
  
  # Now add those polygons!
  addPolygons(
    fillColor = ~map_pal(sales), # the map palette we made
    weight = 2,
    opacity = 1,
    color = "white",
    dashArray = "3",
    stroke = TRUE,
    fillOpacity = 0.5,
    highlight = highlightOptions(
      weight = 5,
      color = "#667",
      dashArray = "",
      fillOpacity = 0.7,
      bringToFront = TRUE
    ),
    label = labels, # cool labels
    labelOptions = labelOptions(
      style = list("font-weight" = "normal", padding = "3px 8px"),
      textsize = "15px",
      directon = "auto"
    )
  ) %>%
  
  # Dont forget a legend
  addLegend(
    pal = map_pal,
    values =  ~sales,
    opacity = 0.7,
    title = "Sales"
  )
```

And here is the finished map! [full screen](zcta_map.html)

<iframe src= "zcta_map.html" width="100%" height="400" style="border: none;"></iframe>



### Acknowledgements

This blog post was made possible thanks to:

* *[rgdal](https://bioconductor.org/packages/3.12/rgdal)* <a id='cite-Bivand_2020'></a>(<a href='https://CRAN.R-project.org/package=rgdal'>Bivand, Keitt, and Rowlingson, 2020</a>)
* *[leaflet](https://bioconductor.org/packages/3.12/leaflet)* <a id='cite-Cheng_2019'></a>(<a href='https://CRAN.R-project.org/package=leaflet'>Cheng, Karambelkar, and Xie, 2019</a>)
* *[dplyr](https://bioconductor.org/packages/3.12/dplyr)* <a id='cite-Wickham_2020'></a>(<a href='https://CRAN.R-project.org/package=dplyr'>Wickham, François, 
             Henry, and Müller, 2020</a>)
* *[data.table](https://bioconductor.org/packages/3.12/data.table)* <a id='cite-Dowle_2019'></a>(<a href='https://CRAN.R-project.org/package=data.table'>Dowle and Srinivasan, 2019</a>)
* *[htmlwidgets](https://bioconductor.org/packages/3.12/htmlwidgets)* <a id='cite-Vaidyanathan_2020'></a>(<a href='https://CRAN.R-project.org/package=htmlwidgets'>Vaidyanathan, Xie, Allaire, Cheng, et al., 2020</a>)
* *[BiocStyle](https://bioconductor.org/packages/3.12/BiocStyle)* <a id='cite-Oles_2020'></a>(<a href='https://github.com/Bioconductor/BiocStyle'>Oles, Morgan, and Huber, 2020</a>)
* *[blogdown](https://CRAN.R-project.org/package=blogdown)* <a id='cite-Xie_2017'></a>(<a href='https://github.com/rstudio/blogdown'>Xie, Hill, and Thomas, 2017</a>)
* *[devtools](https://CRAN.R-project.org/package=devtools)* <a id='cite-Wickham_2020a'></a>(<a href='https://CRAN.R-project.org/package=devtools'>Wickham, Hester, and Chang, 2020</a>)
* *[knitcitations](https://CRAN.R-project.org/package=knitcitations)* <a id='cite-Boettiger_2020'></a>(<a href='https://github.com/cboettig/knitcitations'>Boettiger, 2020</a>)

<!-- ### References -->

<!-- ```{r bibliography, results = 'asis', echo = FALSE, cache = FALSE} -->
<!-- ## Print bibliography -->
<!-- bibliography(style = 'html') -->
<!-- ``` -->

<!-- ### Reproducibility -->

<!-- ```{r reproducibility, echo = FALSE} -->
<!-- ## Reproducibility info -->
<!-- options(width = 120) -->
<!-- session_info() -->
<!-- ``` -->
