---
title: "site notes"
output: html_document
---

[![Netlify Status](https://api.netlify.com/api/v1/badges/a87061ba-7943-4aa3-a4ca-d4f3271456c2/deploy-status)](https://app.netlify.com/sites/rjfranssen/deploys)   

## Hello

Welcome to the repo for my site. Excuse the mess. I'm working this out a little bit at a time. I created this site to organize my notes and personal GIS & data science projects. This site includes code snippets and various apps.

This site uses the ["Compose"](https://github.com/onweru/compose) theme for Hugo written by Dan Weru.


```r
knitr::opts_chunk$set(echo = TRUE)
```

```r
library(blogdown)

new_site(
  dir = "~/io",
  install_hugo = TRUE,
  format = "toml",
  sample = TRUE,
  theme = "onweru/compose",
  hostname = "github.com",
  theme_example = TRUE,
  empty_dirs = FALSE,
  to_yaml = TRUE,
  serve = interactive()
)

blogdown::build_site()
blogdown::serve_site()
```

Checkout the [blogdown documentation](https://bookdown.org/yihui/blogdown/static-files.html).  

### Configuring the site  
Used [Toha "Getting started" pages](https://toha-guides.netlify.app/posts/getting-started/github-pages/) for configuring this site with GitHub pages and deploy workflow.  

### Directory notes  

📦 root  
 ┣ 📂 **assets** - override default assets here   
 ┃  ┗ 📂 sass  
 ┃  ┃  ┣ 📜 _bass.sass - set `padding-top: 35px` to prevent head overlap race condition (29nov2020 - dont need this anymore)  
 ┃  ┃  ┣ 📜 _variables.sass - updated color schema here (rgb vals)    
 ┣ 📂 **content** - This is where I go to add content and new mds!!  
 ┃  ┣ 📂 about  
 ┃  ┃  ┗ 📜 _index.md  
 ┃  ┣ 📂 docs  
 ┃  ┃  ┣ 📂 getting started  
 ┃  ┃  ┃  ┣ 📂 shortcodes  
 ┃  ┃  ┃  ┃  ┗ 📜 index.md  
 ┃  ┃  ┃  ┣ 📜 _index.md  
 ┃  ┃  ┃  ┣ 📜 overview.md  
 ┃  ┃  ┃  ┗ 📜 shorcodes-example.md  
 ┃  ┃  ┗ 📜 _index.md  
 ┃  ┣ 📂 links  
 ┃  ┃  ┣ 📂 list  
 ┃  ┃  ┃  ┣ 📂 shortcodes  
 ┃  ┃  ┃  ┃  ┗ 📜 index.md  
 ┃  ┃  ┃  ┣ 📜 _index.md  
 ┃  ┃  ┃  ┣ 📜 overview.md  
 ┃  ┃  ┃  ┗ 📜 shortcodes-example.md  
 ┃  ┃  ┗ 📜 _index.md  
 ┃  ┣ 📂 post - where blogdown articles will be posted  
 ┃  ┃  ┃  ┣ 📜 2015-07-23-r-rmarkdown.Rmd  
 ┃  ┃  ┃  ┗ 📜 2015-07-23-r-rmarkdown.html  
 ┃  ┗ 📜 _index.md  
 ┣ 📂 **layouts** - This overrides the other files and is what I should be editing for theme changes. Note: edited baseof.html for pages with anchor links.  
 ┃  ┣ 📂 _default   
 ┃  ┃  ┣ 📜 baseof.html - need to update if adding nav items  
 ┃  ┣ 📂 partials  
 ┃  ┃  ┣ 📜 head.html - added crossorigin="anonymous" due to CORS error when pushing through netlify    
 ┃  ┣ 📂 shortcodes  
 ┃  ┣ 📜 404.html  
 ┃  ┗ 📜 index.html  
 ┣ 📂 public - don't need to put anything here manually; this is the site that gets launched in github or netlify - it is automatically updated when I serve the site.  
 ┣ 📂 resources  
 ┃  ┗ 📂 _gen  
 ┣ 📂 **static** - can put images and stuff in here.  
 ┃  ┣ 📂 favicons - updated the favicons in here using [favicon.io](https://favicon.io/favicon-converter/)  
 ┃  ┣ 📂 links  
 ┃  ┣ 📂 images - logos here!  
 ┃  ┣ 📂 css - custom css!  
 ┃  ┣ 📂 js - custom js!  
 ┃  ┗ 📂 post  
 ┗ 📂 themes - don't touch!  
 ┃  ┗ 📂 compose (and then update config.toml)  
 ┃    ┣ 📂 assets  
 ┃    ┣ 📂 exampleSite  
 ┃    ┣ 📂 images  
 ┃    ┣ 📂 layouts  
 ┃    ┣ 📂 static  
 ┃    ┣ 📜 .gitignore  
 ┃    ┣ 📜 LICENSE  
 ┃    ┣ 📜 README.md  
 ┃    ┗ 📜 theme.toml  
 ┣ 📜 .gitignore  
 ┣ 📜 .Rhistory  
 ┣ 📜 **config.toml**  
 ┣ 📜 index.Rmd  
 ┣ 📜 LICENSE  
 ┣ 📜 **netlify.toml**  
 ┣ 📜 **README.md**  
 ┗ 📜 rjfranssen.github.io.Rproj  

TODO:
- [x] - add github link to tiger-tracks dash  
- [ ] - add shiny and flex icons  
- [x] - fix flex-wrap on dash container  
- [x] - centralize css files  
- [x] - add flexdashboard snippets to gh 
- [ ] - add flexdashboard snippets to site 
- [ ] - update and add cv to about    
- [x] - troubleshoot /js/search.min
- [x] - convert .rmd to .rmarkdown to enable hugo tocs  
- [x] - set eval=FALSE for htmlwidgets and instead save them to dir and bring them in via iframe (rmd supports htmlwidgets, rmarkdown does not)gohugo.io/content-management/organization#index-pages-index-md)  
[page bundles (hugo-sandbox)](https://hugo-sandbox.netlify.app/hugodocs/content-management/page-bundles/)  
