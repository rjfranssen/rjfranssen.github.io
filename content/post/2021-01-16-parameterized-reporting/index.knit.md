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
















