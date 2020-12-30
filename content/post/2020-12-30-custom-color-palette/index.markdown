---
title: Custom Color Palette
author: rjfranssen
date: '2020-12-30'
slug: custom-color-palette
categories: []
tags: []
header:
  caption: ''
  image: ''
  preview: yes
---




Last updated 2020-12-30.

I'm describing customizing my own color palette for ggplot2. This is based off of [Creating corporate colour palettes for ggplot2](https://drsimonj.svbtle.com/creating-corporate-colour-palettes-for-ggplot2) by [drsimonj](https://twitter.com/drsimonj). 

First, find a bunch of colors that you like. [Canva](https://www.canva.com/) is a good place to go for inspiration; or, us this [tool from Adobe Color](https://color.adobe.com/create/image) that will extract colors from a photo you like. Then, there's always [colorbrewer2](https://colorbrewer2.org/#type=sequential&scheme=BuGn&n=3).  

_Note: In many cases, you can get away with using the `scale_color_manual()` and `scale_fill_manual()` functions directly within ggplot2. See [the docs](https://ggplot2.tidyverse.org/reference/scale_manual.html) for examples._


Define colors


```r
rjfranssen_colors <- c(
  `ebony` = "#1E2225",
  `blue_grotto` = "#05263B",
  `charcoal` = "#43758A",
  `light_sea_green` = "#36B4AF",
  `pewter` = "#D3DADA",
  `carafe` = "#55423A",
  `goldenrod` = "#F1B416",
  `burnt_sienna` = "#E56A19",
  `crimson` = "#900000")
```


Function to extract colors as hex codes (source: [drsimonj](https://drsimonj.svbtle.com/creating-corporate-colour-palettes-for-ggplot2))


```r
rjfranssen_cols <- function(...) {
  cols <- c(...)

  if (is.null(cols))
    return (rjfranssen_colors)

  rjfranssen_colors[cols]
}
```

Examples of returning specific color codes. The `scales` package has a convenient `show_col()` function for this:


```r
library(scales)
library(dplyr)

par(mfrow=c(1,2))
rjfranssen_cols() %>% show_col()
rjfranssen_cols("light_sea_green", "burnt_sienna") %>% show_col()
```

<img src="/post/2020-12-30-custom-color-palette/index_files/figure-html/examples-1.png" width="672" />

Simply plot example.


```r
ggplot(mtcars, aes(hp, mpg)) +
    geom_point(color = rjfranssen_cols("goldenrod"),
               size = 4, alpha = .8)
```

<img src="/post/2020-12-30-custom-color-palette/index_files/figure-html/simpltplot-1.png" width="672" />


Now, combine sets of colors into individual palettes.


```r
rjfranssen_palettes <- list(
  
  `primary`  = rjfranssen_cols("ebony", "blue_grotto", "charcoal", "light_sea_green", "pewter"),

  `secondary`  = rjfranssen_cols("carafe", "goldenrod", "burnt_sienna", "crimson"),

  `all`   = rjfranssen_cols("ebony", "blue_grotto", "charcoal", "light_sea_green", "pewter", "carafe", "goldenrod", "burnt_sienna", "crimson")
)
```


Function to interpolate the palettes (source: [drsimonj](https://drsimonj.svbtle.com/creating-corporate-colour-palettes-for-ggplot2)). Option to padd additional arguments into `colorRampPallete()` like `alpha`.


```r
rjfranssen_pal <- function(palette = "main", reverse = FALSE, ...) {
  pal <- rjfranssen_palettes[[palette]]

  if (reverse) pal <- rev(pal)

  colorRampPalette(pal, ...)
}
```


Preview the palettes, e.g. applying 10 levels to the `primary` color palette. Again using `scales::show_col()` to display the colors with hex codes.  



```r
par(mfrow=c(1,2))

rjfranssen_pal("primary")(12) %>% show_col()
rjfranssen_pal("secondary")(4) %>% show_col()
```

<img src="/post/2020-12-30-custom-color-palette/index_files/figure-html/showinterp-1.png" width="672" />


Everything that's been up to this point is creating a storage places for favorite colors. Now, prepare some functions specifically for using with `ggplot`.


```r
scale_color_rjfranssen <- function(palette = "main",
           discrete = TRUE,
           reverse = FALSE,
           ...) {
    pal <- rjfranssen_pal(palette = palette, reverse = reverse)
    
    if (discrete) {
      discrete_scale("colour", paste0("rjfranssen_", palette), palette = pal, ...)
    } else {
      scale_color_gradientn(colours = pal(256), ...)
    }
}

scale_fill_rjfranssen <- function(palette = "main", discrete = TRUE, reverse = FALSE, ...) {
  pal <- rjfranssen_pal(palette = palette, reverse = reverse)

  if (discrete) {
    discrete_scale("fill", paste0("rjfranssen_", palette), palette = pal, ...)
  } else {
    scale_fill_gradientn(colours = pal(256), ...)
  }
}
```


Done - now plotting a few examples.



```r
ggplot(iris, aes(Sepal.Width, Sepal.Length, color = Sepal.Length)) +
    geom_point(size = 4, alpha = .6) +
    scale_color_rjfranssen(discrete = FALSE, palette = "primary")
```

<img src="/post/2020-12-30-custom-color-palette/index_files/figure-html/exampletwo-1.png" width="672" />


```r
ggplot(iris, aes(Sepal.Width, Sepal.Length, color = Species)) +
    geom_point(size = 4) +
    scale_color_rjfranssen("secondary")
```

<img src="/post/2020-12-30-custom-color-palette/index_files/figure-html/exampleone-1.png" width="672" />



```r
ggplot(mpg, aes(manufacturer, fill = manufacturer)) +
    geom_bar() +
    theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
    scale_fill_rjfranssen(palette = "all", guide = "none")
```

<img src="/post/2020-12-30-custom-color-palette/index_files/figure-html/examplethree-1.png" width="672" />

Finally, I want to be able to pull down my palette whereever I am, so I copied the above R code into `rjfranssen_col_pal.r` and pushed it to my [GitHub repo](https://github.com/rjfranssen/rjfranssen). I can read it down using `devtools` as an [r file](https://stackoverflow.com/questions/35720660/how-to-use-an-r-script-from-github) or as a [gist](https://gist.github.com/jtoll/4041987). [(devtools cheatsheet)](https://rstudio.com/wp-content/uploads/2015/03/devtools-cheatsheet.pdf)  


```r
library(devtools)
source_url("https://raw.githubusercontent.com/rjfranssen/rjfranssen/main/utils/rjfranssen_col_pal.r")
```





### Acknowledgements


This blog post was made possible thanks to:

* *[BiocStyle](https://bioconductor.org/packages/3.12/BiocStyle)* <a id='cite-Oles_2020'></a>(<a href='https://github.com/Bioconductor/BiocStyle'>Oles, Morgan, and Huber, 2020</a>)
* *[blogdown](https://CRAN.R-project.org/package=blogdown)* <a id='cite-Xie_2017'></a>(<a href='https://github.com/rstudio/blogdown'>Xie, Hill, and Thomas, 2017</a>)
* *[devtools](https://CRAN.R-project.org/package=devtools)* <a id='cite-Wickham_2020'></a>(<a href='https://CRAN.R-project.org/package=devtools'>Wickham, Hester, and Chang, 2020</a>)
* *[knitcitations](https://CRAN.R-project.org/package=knitcitations)* <a id='cite-Boettiger_2020'></a>(<a href='https://github.com/cboettig/knitcitations'>Boettiger, 2020</a>)

### References

<p><a id='bib-Boettiger_2020'></a><a href="#cite-Boettiger_2020">[1]</a><cite>
C. Boettiger.
<em>knitcitations: Citations for 'Knitr' Markdown Files</em>.
R package version 1.0.11.
2020.
URL: <a href="https://github.com/cboettig/knitcitations">https://github.com/cboettig/knitcitations</a>.</cite></p>

<p><a id='bib-Oles_2020'></a><a href="#cite-Oles_2020">[2]</a><cite>
A. Oles, M. Morgan, and W. Huber.
<em>BiocStyle: Standard styles for vignettes and other Bioconductor documents</em>.
R package version 2.18.1.
2020.
URL: <a href="https://github.com/Bioconductor/BiocStyle">https://github.com/Bioconductor/BiocStyle</a>.</cite></p>

<p><a id='bib-Wickham_2020'></a><a href="#cite-Wickham_2020">[3]</a><cite>
H. Wickham, J. Hester, and W. Chang.
<em>devtools: Tools to Make Developing R Packages Easier</em>.
R package version 2.3.1.
2020.
URL: <a href="https://CRAN.R-project.org/package=devtools">https://CRAN.R-project.org/package=devtools</a>.</cite></p>

<p><a id='bib-Xie_2017'></a><a href="#cite-Xie_2017">[4]</a><cite>
Y. Xie, A. P. Hill, and A. Thomas.
<em>blogdown: Creating Websites with R Markdown</em>.
ISBN 978-0815363729.
Boca Raton, Florida: Chapman and Hall/CRC, 2017.
URL: <a href="https://github.com/rstudio/blogdown">https://github.com/rstudio/blogdown</a>.</cite></p>

### Reproducibility


```
## - Session info -------------------------------------------------------------------------------------------------------
##  setting  value                       
##  version  R version 4.0.2 (2020-06-22)
##  os       Windows 10 x64              
##  system   x86_64, mingw32             
##  ui       RTerm                       
##  language (EN)                        
##  collate  English_United States.1252  
##  ctype    English_United States.1252  
##  tz       America/New_York            
##  date     2020-12-30                  
## 
## - Packages -----------------------------------------------------------------------------------------------------------
##  package       * version date       lib source                                 
##  assertthat      0.2.1   2019-03-21 [1] CRAN (R 4.0.2)                         
##  BiocManager     1.30.10 2019-11-16 [1] CRAN (R 4.0.3)                         
##  BiocStyle     * 2.18.1  2020-11-24 [1] Bioconductor                           
##  blogdown        0.20    2020-06-23 [1] CRAN (R 4.0.2)                         
##  bookdown        0.21    2020-10-13 [1] CRAN (R 4.0.3)                         
##  callr           3.4.3   2020-03-28 [1] CRAN (R 4.0.2)                         
##  cli             2.2.0   2020-11-20 [1] CRAN (R 4.0.3)                         
##  colorspace      2.0-0   2020-11-11 [1] CRAN (R 4.0.3)                         
##  crayon          1.3.4   2017-09-16 [1] CRAN (R 4.0.2)                         
##  desc            1.2.0   2018-05-01 [1] CRAN (R 4.0.2)                         
##  devtools      * 2.3.1   2020-07-21 [1] CRAN (R 4.0.2)                         
##  digest          0.6.27  2020-10-24 [1] CRAN (R 4.0.3)                         
##  dplyr         * 1.0.2   2020-08-18 [1] CRAN (R 4.0.3)                         
##  ellipsis        0.3.1   2020-05-15 [1] CRAN (R 4.0.2)                         
##  evaluate        0.14    2019-05-28 [1] CRAN (R 4.0.2)                         
##  fansi           0.4.1   2020-01-08 [1] CRAN (R 4.0.2)                         
##  farver          2.0.3   2020-01-16 [1] CRAN (R 4.0.2)                         
##  fs              1.5.0   2020-07-31 [1] CRAN (R 4.0.3)                         
##  generics        0.1.0   2020-10-31 [1] CRAN (R 4.0.3)                         
##  ggplot2       * 3.3.2   2020-06-19 [1] CRAN (R 4.0.2)                         
##  glue            1.4.2   2020-08-27 [1] CRAN (R 4.0.2)                         
##  gtable          0.3.0   2019-03-25 [1] CRAN (R 4.0.2)                         
##  htmltools       0.5.0   2020-06-16 [1] CRAN (R 4.0.2)                         
##  httr            1.4.2   2020-07-20 [1] CRAN (R 4.0.3)                         
##  jsonlite        1.7.2   2020-12-09 [1] CRAN (R 4.0.3)                         
##  knitcitations * 1.0.11  2020-12-30 [1] Github (cboettig/knitcitations@23750d5)
##  knitr           1.30    2020-09-22 [1] CRAN (R 4.0.3)                         
##  labeling        0.4.2   2020-10-20 [1] CRAN (R 4.0.3)                         
##  lifecycle       0.2.0   2020-03-06 [1] CRAN (R 4.0.2)                         
##  lubridate     * 1.7.9.2 2020-11-13 [1] CRAN (R 4.0.3)                         
##  magrittr        2.0.1   2020-11-17 [1] CRAN (R 4.0.3)                         
##  memoise         1.1.0   2017-04-21 [1] CRAN (R 4.0.2)                         
##  munsell         0.5.0   2018-06-12 [1] CRAN (R 4.0.2)                         
##  pillar          1.4.6   2020-07-10 [1] CRAN (R 4.0.3)                         
##  pkgbuild        1.0.8   2020-05-07 [1] CRAN (R 4.0.2)                         
##  pkgconfig       2.0.3   2019-09-22 [1] CRAN (R 4.0.2)                         
##  pkgload         1.1.0   2020-05-29 [1] CRAN (R 4.0.2)                         
##  plyr            1.8.6   2020-03-03 [1] CRAN (R 4.0.2)                         
##  prettyunits     1.1.1   2020-01-24 [1] CRAN (R 4.0.2)                         
##  processx        3.4.4   2020-09-03 [1] CRAN (R 4.0.3)                         
##  ps              1.4.0   2020-10-07 [1] CRAN (R 4.0.3)                         
##  purrr           0.3.4   2020-04-17 [1] CRAN (R 4.0.2)                         
##  R6              2.5.0   2020-10-28 [1] CRAN (R 4.0.3)                         
##  Rcpp            1.0.5   2020-07-06 [1] CRAN (R 4.0.2)                         
##  RefManageR      1.3.0   2020-11-13 [1] CRAN (R 4.0.3)                         
##  remotes         2.2.0   2020-07-21 [1] CRAN (R 4.0.3)                         
##  rlang           0.4.9   2020-11-26 [1] CRAN (R 4.0.3)                         
##  rmarkdown       2.5     2020-10-21 [1] CRAN (R 4.0.3)                         
##  rprojroot       2.0.2   2020-11-15 [1] CRAN (R 4.0.2)                         
##  scales        * 1.1.1   2020-05-11 [1] CRAN (R 4.0.2)                         
##  sessioninfo     1.1.1   2018-11-05 [1] CRAN (R 4.0.2)                         
##  stringi         1.5.3   2020-09-09 [1] CRAN (R 4.0.3)                         
##  stringr         1.4.0   2019-02-10 [1] CRAN (R 4.0.2)                         
##  testthat        2.3.2   2020-03-02 [1] CRAN (R 4.0.2)                         
##  tibble          3.0.4   2020-10-12 [1] CRAN (R 4.0.3)                         
##  tidyselect      1.1.0   2020-05-11 [1] CRAN (R 4.0.2)                         
##  usethis       * 2.0.0   2020-12-10 [1] CRAN (R 4.0.3)                         
##  vctrs           0.3.4   2020-08-29 [1] CRAN (R 4.0.3)                         
##  withr           2.3.0   2020-09-22 [1] CRAN (R 4.0.3)                         
##  xfun            0.19    2020-10-30 [1] CRAN (R 4.0.3)                         
##  xml2            1.3.2   2020-04-23 [1] CRAN (R 4.0.2)                         
##  yaml            2.2.1   2020-02-01 [1] CRAN (R 4.0.0)                         
## 
## [1] C:/Users/rober/Documents/R/win-library/4.0
## [2] C:/Program Files/R/R-4.0.2/library
```
