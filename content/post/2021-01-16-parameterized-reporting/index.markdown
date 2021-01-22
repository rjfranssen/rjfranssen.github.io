---
title: Parameterized Reporting in R
author: rjfranssen
date: '2021-01-16'
slug: parameteried-reporting-in-r
categories:
  - helps
tags:
  - r
header:
  caption: ''
  image: ''
  preview: yes
params:
  dataloc: https://raw.githubusercontent.com/tidyverse/ggplot2/master/data-raw/mpg.csv
  dataname: mpg.csv
  manufacturer: pontiac
  date: !r lubridate::today()
---






2021-01-21 | _9 min_


### Just give me the code

Skip all this and jump to the [parameterized-reporting-example.Rmd](https://github.com/rjfranssen/parameterized-reporting/blob/main/parameterized-reporting-example.Rmd) file in my [parameterized-reporting repo](https://github.com/rjfranssen/parameterized-reporting).


### About parametrized reports

This is a parameterized report written in in R Markdown. It consists of a combination of text and code chunks and uses `R`, `pandoc`, and `LaTeX` to connect to a data source, create visualizations, and export a single file to `html`, `pdf`, or `word` _without changing the code_. The resulting pages stand on their own, have no dependencies, and can be shared or stored on a central location, like git pages, a site on netlify, or a sharepoint documents library. The data and code are readily accessible for transparency and reproducibility.

Each export format has its own advantages:


|                         | html | word |  pdf  |
|-------------------------|:----:|:----:|:-----:|
|Standalone               | X    | X    | X     |
|Easy print               |      | X    | X     |
|Editable                 |      | X    |       |
|Static charts/maps       | X    | X    | X     |
|Interactive charts/maps  | X    |      |       |
|Embed data for download  | X    |      |       |


For good documentation, check out:  
* [Parameterized Reports (rmarkdown)](https://garrettgman.github.io/rmarkdown/developer_parameterized_reports.html)  
* [Knitting with parameters (bookdown)](https://bookdown.org/yihui/rmarkdown/params-knit.html#knit-with-custom-parameters)  
* [Parameterized R Markdown - incudes `marmap` example (rstudio)](https://docs.rstudio.com/connect/1.7.4/user/param-rmarkdown.html)  
* [Iterate multiple RMarkdown reports (rstudio community)](https://community.rstudio.com/t/iterate-multiple-rmarkdown-reports/43208)  



### Example Report elements for **pontiac** 

Start by pulling down a csv from GitHub to use in this example:



```r
csv_url <- "https://raw.githubusercontent.com/tidyverse/ggplot2/master/data-raw/mpg.csv"

download.file(csv_url, destfile = "mpg.csv", method = "curl")
df <- data.table::fread("mpg.csv")
```


### Parameter Specfic Charts

Next, I'm going to insert the params that I defined in the `yml` header of this document into the R code to come up with some **inline statistics**.

For example, the `yml` header of this post looks like this - look at the params:

```
title: Parameterized Reporting in R
author: rjfranssen
date: '2021-01-16'
slug: parameteried-reporting-in-r
categories:
  - helps
tags:
  - r
header:
  caption: ''
  image: ''
  preview: yes
params:
  manufacturer: pontiac
  date: !r lubridate::today()
```

The manufacturer **pontiac** has **5** vehicles in this dataset.  
They average **17** mpg city and **26.4** mpg highway. 

**Vehicle Counts**


```r
df2 <- df[df$manufacturer == params$manufacturer, ]
  
plt <- ggplot(df2) +
  geom_bar( aes(x = as.factor(cyl), fill = as.factor(cyl)), stat = 'count') +
  scale_fill_viridis_d(option = "viridis") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  theme(legend.position = "none")

#ggplotly(plt)
plt
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />

**MPGs**


```r
df2 <- df[df$manufacturer == params$manufacturer, ]
  
plt <- ggplot(df2) +
  geom_point(aes(x = displ, y = hwy, color = as.factor(year), label = model), size = 5) +
  scale_color_viridis_d(option = "viridis") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  theme(legend.position = "none")

#ggplotly(plt)
plt
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />



### All Car Manufacturers

**Vehicle Counts**


```r
plt <- ggplot(df) +
  geom_bar( aes(manufacturer, fill = manufacturer), stat = 'count') +
  scale_fill_viridis_d(option = "viridis") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  theme(legend.position = "none")

#ggplotly(plt)
plt
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />


**MPGs**


```r
plt <- ggplot(df) +
  geom_point(aes(x = cty, y = hwy, color = manufacturer), alpha = 0.6) +
  scale_fill_viridis_d(option = "viridis") +
  facet_wrap(~manufacturer) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  theme(legend.position = "none") 

#ggplotly(plt)
plt
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />


Lastly, I'm going to use the below function to read in the `.Rmd` file and ouput an `html` file using the parameters as inputs. Then, I'm going to run this function for each manufacturer in the sample dataset to generate a custom report for each manufactuer.

Credit: [bookdown docs](https://bookdown.org/yihui/rmarkdown/params-knit.html#knit-with-custom-parameters) and this [stackoverflow post](https://stackoverflow.com/questions/54969070/rendering-multiple-parametrized-rmarkdown-files-by-render-function-fails)  


```r
# Function to render this rmd file as an html doc; run this in R console
render_report = function(manufacturer = params$manufacturer, date = params$date) {
  rmarkdown::render(
    input = "parameterized-reporting-example.Rmd",
    output_format = "html_document",
    params = list(
      manufacturer = manufacturer,
      date = date
    ),
    output_file = paste0(manufacturer, "_mpg_report_", date, ".html")
  )
}

# Test one-off report with default params
render_report()

# Now loop through and create one report for each manufacturer
# (note: need to pull down the file and create `df` before running)
for (i in unique(df$manufacturer)){
  render_report(manufacturer = i)
  print(paste0('finished knitting ', i))
}
```

### Final report!

The final report will look like this [(see full screen)](https://htmlpreview.github.io/?https://github.com/rjfranssen/parameterized-reporting/blob/main/reports/pontiac_mpg_report_2021-01-20.html) 

<iframe src= "https://htmlpreview.github.io/?https://github.com/rjfranssen/parameterized-reporting/blob/main/reports/pontiac_mpg_report_2021-01-20.html" width="100%" height="400" style="border: none;"></iframe>

### Acknowledgements




This blog post was made possible thanks to:

* *[lubridate](https://bioconductor.org/packages/3.12/lubridate)* <a id='cite-Grolemund_2011'></a>(<a href='https://www.jstatsoft.org/v40/i03/'>Grolemund and Wickham, 2011</a>)
* *[rmarkdown](https://bioconductor.org/packages/3.12/rmarkdown)* 
* *[BiocStyle](https://bioconductor.org/packages/3.12/BiocStyle)* <a id='cite-Oles_2020'></a>(<a href='https://github.com/Bioconductor/BiocStyle'>Oles, Morgan, and Huber, 2020</a>)
* *[blogdown](https://CRAN.R-project.org/package=blogdown)* <a id='cite-Xie_2017'></a>(<a href='https://github.com/rstudio/blogdown'>Xie, Hill, and Thomas, 2017</a>)
* *[devtools](https://CRAN.R-project.org/package=devtools)* <a id='cite-Wickham_2020'></a>(<a href='https://CRAN.R-project.org/package=devtools'>Wickham, Hester, and Chang, 2020</a>)
* *[knitcitations](https://CRAN.R-project.org/package=knitcitations)* <a id='cite-Boettiger_2020'></a>(<a href='https://github.com/cboettig/knitcitations'>Boettiger, 2020</a>)
* [_htmlpreview_ (Glowacki 2019)](https://github.com/htmlpreview/htmlpreview.github.com)

