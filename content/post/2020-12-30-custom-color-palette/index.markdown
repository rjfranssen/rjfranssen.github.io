---
title: "Custom Color Palette in R"
author: "rjfranssen"
date: '2020-12-30'
header:
  caption: ''
  image: ''
  preview: yes
slug: custom-color-palette
tags:
- r
- ggplot2
- color
categories:
- utilities
- helpers
---







2021-01-21 | _8 min_

I'm describing customizing my own color palette for ggplot2. This is based off of [Creating corporate colour palettes for ggplot2](https://drsimonj.svbtle.com/creating-corporate-colour-palettes-for-ggplot2) by [drsimonj](https://twitter.com/drsimonj).

First, find a bunch of colors that you like. [Canva](https://www.canva.com/) is a good place to go for inspiration; or, us this [tool from Adobe Color](https://color.adobe.com/create/image) that will extract colors from a photo you like. Then, there's always [colorbrewer2](https://colorbrewer2.org/#type=sequential&scheme=BuGn&n=3).  

_Note: In some cases, it's enough to use the `scale_color_manual()` and `scale_fill_manual()` functions directly within ggplot2. See [the docs](https://ggplot2.tidyverse.org/reference/scale_manual.html) for examples._


### Define colors

This is the color scheme used by the non-profit, **[splashdown.org](https://www.splashdown.org/)**.  


```r
splashdown_colors <- c(
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
splashdown_cols <- function(...) {
  cols <- c(...)
  if (is.null(cols))
    return (splashdown_colors)
  splashdown_colors[cols]
}
```

Examples of returning specific color codes. The `scales` package has a convenient `show_col()` function for this:


```r
library(scales)
library(dplyr)

par(mfrow=c(1,2))
splashdown_cols() %>% show_col()
splashdown_cols("light_sea_green", "burnt_sienna") %>% show_col()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/examples-1.png" width="672" />

Very simple plot with color selection.


```r
ggplot(mtcars, aes(hp, mpg)) +
    geom_point(color = splashdown_cols("goldenrod"),
               size = 4, alpha = .8)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/simpltplot-1.png" width="672" />

### Create palettes

Combine sets of colors to create individual palettes.


```r
splashdown_palettes <- list(
  `primary`  = splashdown_cols("ebony", "blue_grotto", "charcoal", "light_sea_green", "pewter"),
  `secondary`  = splashdown_cols("carafe", "goldenrod", "burnt_sienna", "crimson"),
  `all`   = splashdown_cols("ebony", "blue_grotto", "charcoal", "light_sea_green", "pewter", "carafe", "goldenrod", "burnt_sienna", "crimson")
)
```


Function to interpolate the palettes (source: [drsimonj](https://drsimonj.svbtle.com/creating-corporate-colour-palettes-for-ggplot2)). Option to add additional arguments into `colorRampPallete()` like `alpha`.



```r
splashdown_pal <- function(palette = "main", reverse = FALSE, ...) {
  pal <- splashdown_palettes[[palette]]
  if (reverse) pal <- rev(pal)
  colorRampPalette(pal, ...)
}
```


Preview the palettes, e.g. applying 10 levels to the `primary` color palette. Again using `scales::show_col()` to display the colors with hex codes.  



```r
par(mfrow=c(1,2))

splashdown_pal("primary")(12) %>% show_col()
splashdown_pal("secondary")(4) %>% show_col()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/showinterp-1.png" width="672" />


### Create Scale functions for ggplot2

Everything that's been up to this point is creating a storage places for favorite colors. Now, prepare some functions specifically for using with `ggplot`.


```r
scale_color_splashdown <- function(palette = "main",
           discrete = TRUE,
           reverse = FALSE,
           ...) {
    pal <- splashdown_pal(palette = palette, reverse = reverse)
    
    if (discrete) {
      discrete_scale("colour", paste0("splashdown_", palette), palette = pal, ...)
    } else {
      scale_color_gradientn(colours = pal(256), ...)
    }
}

scale_fill_splashdown <- function(palette = "main", discrete = TRUE, reverse = FALSE, ...) {
  pal <- splashdown_pal(palette = palette, reverse = reverse)

  if (discrete) {
    discrete_scale("fill", paste0("splashdown_", palette), palette = pal, ...)
  } else {
    scale_fill_gradientn(colours = pal(256), ...)
  }
}
```



### Example plots



```r
ggplot(iris, aes(Sepal.Width, Sepal.Length, color = Sepal.Length)) +
    geom_point(size = 4, alpha = .6) +
    scale_color_splashdown(discrete = FALSE, palette = "primary")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/exampletwo-1.png" width="672" />


```r
ggplot(iris, aes(Sepal.Width, Sepal.Length, color = Species)) +
    geom_point(size = 4) +
    scale_color_splashdown("secondary")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/exampleone-1.png" width="672" />



```r
ggplot(mpg, aes(manufacturer, fill = manufacturer)) +
    geom_bar() +
    theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
    scale_fill_splashdown(palette = "all", guide = "none")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/examplethree-1.png" width="672" />


Finally, I want to be able to pull down my palette whereever I am, so I copied the above R code into `splashdown_col_pal.r` and pushed it to my [GitHub repo](https://github.com/rjfranssen/rjfranssen). I can read it down using `devtools` as an [r file](https://stackoverflow.com/questions/35720660/how-to-use-an-r-script-from-github) or as a [gist](https://gist.github.com/jtoll/4041987). [(devtools cheatsheet)](https://rstudio.com/wp-content/uploads/2015/03/devtools-cheatsheet.pdf)  


```r
library(devtools)
source_url("https://raw.githubusercontent.com/rjfranssen/rjfranssen/main/utils/splashdown_col_pal.r")
```





### Acknowledgements

This post was made possible thanks to

* *[lubridate](https://bioconductor.org/packages/3.12/lubridate)* <a id='cite-Grolemund_2011'></a>(<a href='https://www.jstatsoft.org/v40/i03/'>Grolemund and Wickham, 2011</a>)
* *[dplyr](https://bioconductor.org/packages/3.12/dplyr)* <a id='cite-Wickham_2020'></a>(<a href='https://CRAN.R-project.org/package=dplyr'>Wickham, François, 
             Henry, and Müller, 2020</a>)
* *[ggplot2](https://CRAN.R-project.org/package=ggplot2)* <a id='cite-Wickham_2016'></a>(<a href='https://ggplot2.tidyverse.org'>Wickham, 2016</a>)
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
