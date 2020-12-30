---
title: Twitter Sentiment Philly
author: rjfranssen
date: '2020-12-30'
slug: twitter-sentiment-philly
categories:
  - natural language processing
tags:
  - r
  - nlp
  - twitter
header:
  caption: ''
  image: ''
  preview: yes
---



```r
# Knitr options: https://yihui.org/knitr/options/
library(lubridate)
knitr::opts_chunk$set(collapse = TRUE
                      , eval = TRUE
                      , echo = TRUE
                      , message = FALSE
                      , warning = FALSE
                      , include = TRUE)
```


<!-- ```{r, eval = FALSE, include = FALSE} -->
<!-- library(leaflet) -->
<!-- library(widgetframe) -->
<!-- l <- leaflet(height=300) %>% addTiles() %>% setView(0,0,1) -->

<!-- frameWidget(l) -->
<!-- ``` -->


Last updated 2020-12-30.  

This is a walkthrough of using the Twitter API with the `rtweet` R package. It borrows from a few difference sources, including:
* [ropensci/rtweet documentation](https://github.com/ropensci/rtweet)  
* [here](https://gist.github.com/bryangoodrich/7b5ef683ce8db592669e)  
* [Tidy Text Mining with R](https://www.tidytextmining.com/topicmodeling.html)  
* [Twitter Data in R Using Rtweet: Analyze and Download Twitter Data](https://www.earthdatascience.org/courses/earth-analytics/get-data-using-apis/use-twitter-api-r/) 
* [Comparing Twitter archives](https://www.tidytextmining.com/twitter.html)  
* [Mining NASA metadata](https://www.tidytextmining.com/nasa.html)  
* [Analyzing usenet text](https://www.tidytextmining.com/usenet.html)  

### Load libraries


```r
#library(twitteR)
library(rtweet)
library(rmarkdown)
library(NLP)
library(tm)
library(RColorBrewer)
library(wordcloud)
library(topicmodels)
library(SnowballC)
library(config)
library(dplyr)
library(tidytext)
library(tidyr)
library(ggplot2)
library(lubridate)
library(readr)
library(skimr)
library(widgetframe)
```

### Configure Twitter API access

There are a couple different ways to authenticate your session. The `rtweet` package makes this pretty easy by enabling browser-based authentication. I prefer to authenticate in code. See `vignette("auth", package = "rtweet")` for options.

What I did: Head over to the [Twitter Developer](https://developer.twitter.com/en/apply-for-access) site and requested a developer account to gain access to Twitter APIs and tools.

Then I used a `config.yml` to store my keys. The `config` package a good way to import keys and passwords without making them publc (just make sure they're covered by your `.gitignore` file!). Check out the [documentation](https://cran.r-project.org/web/packages/config/vignettes/introduction.html). _Note: If using Hugo and don't want it swept up into your `public` directory, you can also add `config.yml` to the `ignoreFiles` parameter in the `config.toml`_.



```r
config <- config::get(file = 'config.yml')

## authenticate via web browser
token <- create_token(
  app = config$app,
  consumer_key = config$apikey,
  consumer_secret = config$apikeysecret,
  access_token = config$accesstoken,
  access_secret = config$accesstokensecret)
```


### Query the Tweets!

Going to ask for the most recent 2000.  


```r
# Search tweets using rtweet search function
tweets_df <- rtweet::search_tweets(
  q = 'philadelphia philly'
  , n = 2000
  , include_rts = FALSE
  , type = 'recent'
  #, geocode = "37.78, -122.40, 1mi"
  #, retryonratelimit = TRUE
  #, parse = FALSE
  #, lang = 'en' 
  # ... see Twitter's REST API for more search parameters
)

## Create lat/lng variables using all available tweet and profile geo-location data
tweets_df <- rtweet::lat_lng(tweets_df)

## Also going to create some friendlier date variables;`created_at` is a `POSIXct` type that requires a bit of tender loving care. WIll still keep the `created_at` value, which works great with `dygraphs` and just fine with `ggplot2`, but I want to be able to group by date variables.
tweets_df$tweet_datetime <- tweets_df$created_at
tweets_df$tweet_date <- tweets_df$created_at %>% as.POSIXlt() %>% as.Date() %>% ymd()
tweets_df$tweet_year <- tweets_df$created_at %>% as.POSIXlt() %>% as.Date() %>% year()
tweets_df$tweet_month <- tweets_df$created_at %>% as.POSIXlt() %>% as.Date() %>% month()
tweets_df$tweet_day <- tweets_df$created_at %>% as.POSIXlt() %>% as.Date() %>% day()


tweets_df %>% head()
```


Optionally, save this df to a local directory so you come back to this later without having to hit the Twitter API; `Rds` and `Rdata` files load super fast. You can also add these file types to your `.gitignore` and `config.toml` if you don't want to push them to your online git repo.


```r
saveRDS(tweets_df, file = "tweets_df.Rds", compress = 'xz')
```

Reading back in the `.Rds` file..


```r
tweets_df <- readRDS("tweets_df.Rds")

tweets_df %>% head(5) %>% paged_table()
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["user_id"],"name":[1],"type":["chr"],"align":["left"]},{"label":["status_id"],"name":[2],"type":["chr"],"align":["left"]},{"label":["created_at"],"name":[3],"type":["dttm"],"align":["right"]},{"label":["screen_name"],"name":[4],"type":["chr"],"align":["left"]},{"label":["text"],"name":[5],"type":["chr"],"align":["left"]},{"label":["source"],"name":[6],"type":["chr"],"align":["left"]},{"label":["display_text_width"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["reply_to_status_id"],"name":[8],"type":["chr"],"align":["left"]},{"label":["reply_to_user_id"],"name":[9],"type":["chr"],"align":["left"]},{"label":["reply_to_screen_name"],"name":[10],"type":["chr"],"align":["left"]},{"label":["is_quote"],"name":[11],"type":["lgl"],"align":["right"]},{"label":["is_retweet"],"name":[12],"type":["lgl"],"align":["right"]},{"label":["favorite_count"],"name":[13],"type":["int"],"align":["right"]},{"label":["retweet_count"],"name":[14],"type":["int"],"align":["right"]},{"label":["quote_count"],"name":[15],"type":["int"],"align":["right"]},{"label":["reply_count"],"name":[16],"type":["int"],"align":["right"]},{"label":["hashtags"],"name":[17],"type":["list"],"align":["right"]},{"label":["symbols"],"name":[18],"type":["list"],"align":["right"]},{"label":["urls_url"],"name":[19],"type":["list"],"align":["right"]},{"label":["urls_t.co"],"name":[20],"type":["list"],"align":["right"]},{"label":["urls_expanded_url"],"name":[21],"type":["list"],"align":["right"]},{"label":["media_url"],"name":[22],"type":["list"],"align":["right"]},{"label":["media_t.co"],"name":[23],"type":["list"],"align":["right"]},{"label":["media_expanded_url"],"name":[24],"type":["list"],"align":["right"]},{"label":["media_type"],"name":[25],"type":["list"],"align":["right"]},{"label":["ext_media_url"],"name":[26],"type":["list"],"align":["right"]},{"label":["ext_media_t.co"],"name":[27],"type":["list"],"align":["right"]},{"label":["ext_media_expanded_url"],"name":[28],"type":["list"],"align":["right"]},{"label":["ext_media_type"],"name":[29],"type":["chr"],"align":["left"]},{"label":["mentions_user_id"],"name":[30],"type":["list"],"align":["right"]},{"label":["mentions_screen_name"],"name":[31],"type":["list"],"align":["right"]},{"label":["lang"],"name":[32],"type":["chr"],"align":["left"]},{"label":["quoted_status_id"],"name":[33],"type":["chr"],"align":["left"]},{"label":["quoted_text"],"name":[34],"type":["chr"],"align":["left"]},{"label":["quoted_created_at"],"name":[35],"type":["dttm"],"align":["right"]},{"label":["quoted_source"],"name":[36],"type":["chr"],"align":["left"]},{"label":["quoted_favorite_count"],"name":[37],"type":["int"],"align":["right"]},{"label":["quoted_retweet_count"],"name":[38],"type":["int"],"align":["right"]},{"label":["quoted_user_id"],"name":[39],"type":["chr"],"align":["left"]},{"label":["quoted_screen_name"],"name":[40],"type":["chr"],"align":["left"]},{"label":["quoted_name"],"name":[41],"type":["chr"],"align":["left"]},{"label":["quoted_followers_count"],"name":[42],"type":["int"],"align":["right"]},{"label":["quoted_friends_count"],"name":[43],"type":["int"],"align":["right"]},{"label":["quoted_statuses_count"],"name":[44],"type":["int"],"align":["right"]},{"label":["quoted_location"],"name":[45],"type":["chr"],"align":["left"]},{"label":["quoted_description"],"name":[46],"type":["chr"],"align":["left"]},{"label":["quoted_verified"],"name":[47],"type":["lgl"],"align":["right"]},{"label":["retweet_status_id"],"name":[48],"type":["chr"],"align":["left"]},{"label":["retweet_text"],"name":[49],"type":["chr"],"align":["left"]},{"label":["retweet_created_at"],"name":[50],"type":["dttm"],"align":["right"]},{"label":["retweet_source"],"name":[51],"type":["chr"],"align":["left"]},{"label":["retweet_favorite_count"],"name":[52],"type":["int"],"align":["right"]},{"label":["retweet_retweet_count"],"name":[53],"type":["int"],"align":["right"]},{"label":["retweet_user_id"],"name":[54],"type":["chr"],"align":["left"]},{"label":["retweet_screen_name"],"name":[55],"type":["chr"],"align":["left"]},{"label":["retweet_name"],"name":[56],"type":["chr"],"align":["left"]},{"label":["retweet_followers_count"],"name":[57],"type":["int"],"align":["right"]},{"label":["retweet_friends_count"],"name":[58],"type":["int"],"align":["right"]},{"label":["retweet_statuses_count"],"name":[59],"type":["int"],"align":["right"]},{"label":["retweet_location"],"name":[60],"type":["chr"],"align":["left"]},{"label":["retweet_description"],"name":[61],"type":["chr"],"align":["left"]},{"label":["retweet_verified"],"name":[62],"type":["lgl"],"align":["right"]},{"label":["place_url"],"name":[63],"type":["chr"],"align":["left"]},{"label":["place_name"],"name":[64],"type":["chr"],"align":["left"]},{"label":["place_full_name"],"name":[65],"type":["chr"],"align":["left"]},{"label":["place_type"],"name":[66],"type":["chr"],"align":["left"]},{"label":["country"],"name":[67],"type":["chr"],"align":["left"]},{"label":["country_code"],"name":[68],"type":["chr"],"align":["left"]},{"label":["geo_coords"],"name":[69],"type":["list"],"align":["right"]},{"label":["coords_coords"],"name":[70],"type":["list"],"align":["right"]},{"label":["bbox_coords"],"name":[71],"type":["list"],"align":["right"]},{"label":["status_url"],"name":[72],"type":["chr"],"align":["left"]},{"label":["name"],"name":[73],"type":["chr"],"align":["left"]},{"label":["location"],"name":[74],"type":["chr"],"align":["left"]},{"label":["description"],"name":[75],"type":["chr"],"align":["left"]},{"label":["url"],"name":[76],"type":["chr"],"align":["left"]},{"label":["protected"],"name":[77],"type":["lgl"],"align":["right"]},{"label":["followers_count"],"name":[78],"type":["int"],"align":["right"]},{"label":["friends_count"],"name":[79],"type":["int"],"align":["right"]},{"label":["listed_count"],"name":[80],"type":["int"],"align":["right"]},{"label":["statuses_count"],"name":[81],"type":["int"],"align":["right"]},{"label":["favourites_count"],"name":[82],"type":["int"],"align":["right"]},{"label":["account_created_at"],"name":[83],"type":["dttm"],"align":["right"]},{"label":["verified"],"name":[84],"type":["lgl"],"align":["right"]},{"label":["profile_url"],"name":[85],"type":["chr"],"align":["left"]},{"label":["profile_expanded_url"],"name":[86],"type":["chr"],"align":["left"]},{"label":["account_lang"],"name":[87],"type":["lgl"],"align":["right"]},{"label":["profile_banner_url"],"name":[88],"type":["chr"],"align":["left"]},{"label":["profile_background_url"],"name":[89],"type":["chr"],"align":["left"]},{"label":["profile_image_url"],"name":[90],"type":["chr"],"align":["left"]},{"label":["lat"],"name":[91],"type":["dbl"],"align":["right"]},{"label":["lng"],"name":[92],"type":["dbl"],"align":["right"]},{"label":["tweet_datetime"],"name":[93],"type":["dttm"],"align":["right"]},{"label":["tweet_date"],"name":[94],"type":["date"],"align":["right"]},{"label":["tweet_year"],"name":[95],"type":["dbl"],"align":["right"]},{"label":["tweet_month"],"name":[96],"type":["dbl"],"align":["right"]},{"label":["tweet_day"],"name":[97],"type":["int"],"align":["right"]}],"data":[{"1":"14221917","2":"1343970067323695105","3":"2020-12-29 17:20:08","4":"PhillyInquirer","5":"Union to sell Mark McKenzie to Belgium’s KRC Genk https://t.co/4GD0RsRv8s","6":"SocialFlow","7":"73","8":"NA","9":"NA","10":"NA","11":"FALSE","12":"FALSE","13":"0","14":"0","15":"NA","16":"NA","17":"<chr [1]>","18":"<chr [1]>","19":"<chr [1]>","20":"<chr [1]>","21":"<chr [1]>","22":"<chr [1]>","23":"<chr [1]>","24":"<chr [1]>","25":"<chr [1]>","26":"<chr [1]>","27":"<chr [1]>","28":"<chr [1]>","29":"NA","30":"<chr [1]>","31":"<chr [1]>","32":"de","33":"NA","34":"NA","35":"<NA>","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"<NA>","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA","64":"NA","65":"NA","66":"NA","67":"NA","68":"NA","69":"<dbl [2]>","70":"<dbl [2]>","71":"<dbl [8]>","72":"https://twitter.com/PhillyInquirer/status/1343970067323695105","73":"The Philadelphia Inquirer","74":"Philadelphia, PA","75":"The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs.","76":"https://t.co/QeWyyrUeoF","77":"FALSE","78":"418589","79":"1591","80":"2966","81":"199157","82":"705","83":"2008-03-26 01:57:54","84":"TRUE","85":"https://t.co/QeWyyrUeoF","86":"http://www.inquirer.com","87":"NA","88":"https://pbs.twimg.com/profile_banners/14221917/1559300463","89":"http://abs.twimg.com/images/themes/theme14/bg.gif","90":"http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png","91":"NA","92":"NA","93":"2020-12-29 17:20:08","94":"2020-12-29","95":"2020","96":"12","97":"29"},{"1":"14221917","2":"1342156765228576771","3":"2020-12-24 17:14:43","4":"PhillyInquirer","5":"It's only the first game ... right? https://t.co/mFsDogez06","6":"SocialFlow","7":"59","8":"NA","9":"NA","10":"NA","11":"FALSE","12":"FALSE","13":"5","14":"1","15":"NA","16":"NA","17":"<chr [1]>","18":"<chr [1]>","19":"<chr [1]>","20":"<chr [1]>","21":"<chr [1]>","22":"<chr [1]>","23":"<chr [1]>","24":"<chr [1]>","25":"<chr [1]>","26":"<chr [1]>","27":"<chr [1]>","28":"<chr [1]>","29":"NA","30":"<chr [1]>","31":"<chr [1]>","32":"en","33":"NA","34":"NA","35":"<NA>","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"<NA>","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA","64":"NA","65":"NA","66":"NA","67":"NA","68":"NA","69":"<dbl [2]>","70":"<dbl [2]>","71":"<dbl [8]>","72":"https://twitter.com/PhillyInquirer/status/1342156765228576771","73":"The Philadelphia Inquirer","74":"Philadelphia, PA","75":"The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs.","76":"https://t.co/QeWyyrUeoF","77":"FALSE","78":"418589","79":"1591","80":"2966","81":"199157","82":"705","83":"2008-03-26 01:57:54","84":"TRUE","85":"https://t.co/QeWyyrUeoF","86":"http://www.inquirer.com","87":"NA","88":"https://pbs.twimg.com/profile_banners/14221917/1559300463","89":"http://abs.twimg.com/images/themes/theme14/bg.gif","90":"http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png","91":"NA","92":"NA","93":"2020-12-24 17:14:43","94":"2020-12-24","95":"2020","96":"12","97":"24"},{"1":"14221917","2":"1343259117633363968","3":"2020-12-27 18:15:04","4":"PhillyInquirer","5":"Today's point of emphasis for the #Eagles against the #Cowboys: third down. @pdomo lays out what the Birds need to do to get the win. https://t.co/UwR050XZc8","6":"SocialFlow","7":"157","8":"NA","9":"NA","10":"NA","11":"FALSE","12":"FALSE","13":"4","14":"2","15":"NA","16":"NA","17":"<chr [2]>","18":"<chr [1]>","19":"<chr [1]>","20":"<chr [1]>","21":"<chr [1]>","22":"<chr [1]>","23":"<chr [1]>","24":"<chr [1]>","25":"<chr [1]>","26":"<chr [1]>","27":"<chr [1]>","28":"<chr [1]>","29":"NA","30":"<chr [1]>","31":"<chr [1]>","32":"en","33":"NA","34":"NA","35":"<NA>","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"<NA>","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA","64":"NA","65":"NA","66":"NA","67":"NA","68":"NA","69":"<dbl [2]>","70":"<dbl [2]>","71":"<dbl [8]>","72":"https://twitter.com/PhillyInquirer/status/1343259117633363968","73":"The Philadelphia Inquirer","74":"Philadelphia, PA","75":"The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs.","76":"https://t.co/QeWyyrUeoF","77":"FALSE","78":"418589","79":"1591","80":"2966","81":"199157","82":"705","83":"2008-03-26 01:57:54","84":"TRUE","85":"https://t.co/QeWyyrUeoF","86":"http://www.inquirer.com","87":"NA","88":"https://pbs.twimg.com/profile_banners/14221917/1559300463","89":"http://abs.twimg.com/images/themes/theme14/bg.gif","90":"http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png","91":"NA","92":"NA","93":"2020-12-27 18:15:04","94":"2020-12-27","95":"2020","96":"12","97":"27"},{"1":"14221917","2":"1341815666563833856","3":"2020-12-23 18:39:19","4":"PhillyInquirer","5":"COVID-19 won't stop South Philadelphians from standing outside in the cold for Christmas cannoli. Some traditions never die. https://t.co/95oXsigsko","6":"SocialFlow","7":"148","8":"NA","9":"NA","10":"NA","11":"FALSE","12":"FALSE","13":"21","14":"2","15":"NA","16":"NA","17":"<chr [1]>","18":"<chr [1]>","19":"<chr [1]>","20":"<chr [1]>","21":"<chr [1]>","22":"<chr [1]>","23":"<chr [1]>","24":"<chr [1]>","25":"<chr [1]>","26":"<chr [1]>","27":"<chr [1]>","28":"<chr [1]>","29":"NA","30":"<chr [1]>","31":"<chr [1]>","32":"en","33":"NA","34":"NA","35":"<NA>","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"<NA>","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA","64":"NA","65":"NA","66":"NA","67":"NA","68":"NA","69":"<dbl [2]>","70":"<dbl [2]>","71":"<dbl [8]>","72":"https://twitter.com/PhillyInquirer/status/1341815666563833856","73":"The Philadelphia Inquirer","74":"Philadelphia, PA","75":"The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs.","76":"https://t.co/QeWyyrUeoF","77":"FALSE","78":"418589","79":"1591","80":"2966","81":"199157","82":"705","83":"2008-03-26 01:57:54","84":"TRUE","85":"https://t.co/QeWyyrUeoF","86":"http://www.inquirer.com","87":"NA","88":"https://pbs.twimg.com/profile_banners/14221917/1559300463","89":"http://abs.twimg.com/images/themes/theme14/bg.gif","90":"http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png","91":"NA","92":"NA","93":"2020-12-23 18:39:19","94":"2020-12-23","95":"2020","96":"12","97":"23"},{"1":"14221917","2":"1342189758647324675","3":"2020-12-24 19:25:49","4":"PhillyInquirer","5":"New art installation in jeopardy after mural of queer activist Gloria Casarez in Philly’s Gayborhood whitewashed without warning https://t.co/sAOX0BSTz0","6":"SocialFlow","7":"152","8":"NA","9":"NA","10":"NA","11":"FALSE","12":"FALSE","13":"4","14":"8","15":"NA","16":"NA","17":"<chr [1]>","18":"<chr [1]>","19":"<chr [1]>","20":"<chr [1]>","21":"<chr [1]>","22":"<chr [1]>","23":"<chr [1]>","24":"<chr [1]>","25":"<chr [1]>","26":"<chr [1]>","27":"<chr [1]>","28":"<chr [1]>","29":"NA","30":"<chr [1]>","31":"<chr [1]>","32":"en","33":"NA","34":"NA","35":"<NA>","36":"NA","37":"NA","38":"NA","39":"NA","40":"NA","41":"NA","42":"NA","43":"NA","44":"NA","45":"NA","46":"NA","47":"NA","48":"NA","49":"NA","50":"<NA>","51":"NA","52":"NA","53":"NA","54":"NA","55":"NA","56":"NA","57":"NA","58":"NA","59":"NA","60":"NA","61":"NA","62":"NA","63":"NA","64":"NA","65":"NA","66":"NA","67":"NA","68":"NA","69":"<dbl [2]>","70":"<dbl [2]>","71":"<dbl [8]>","72":"https://twitter.com/PhillyInquirer/status/1342189758647324675","73":"The Philadelphia Inquirer","74":"Philadelphia, PA","75":"The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs.","76":"https://t.co/QeWyyrUeoF","77":"FALSE","78":"418589","79":"1591","80":"2966","81":"199157","82":"705","83":"2008-03-26 01:57:54","84":"TRUE","85":"https://t.co/QeWyyrUeoF","86":"http://www.inquirer.com","87":"NA","88":"https://pbs.twimg.com/profile_banners/14221917/1559300463","89":"http://abs.twimg.com/images/themes/theme14/bg.gif","90":"http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png","91":"NA","92":"NA","93":"2020-12-24 19:25:49","94":"2020-12-24","95":"2020","96":"12","97":"24"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
tweets_df %>% head(5) %>% DT::datatable()
```

<!--html_preserve--><div id="htmlwidget-954cef94d0beffd49a2e" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-954cef94d0beffd49a2e">{"x":{"filter":"none","data":[["1","2","3","4","5"],["14221917","14221917","14221917","14221917","14221917"],["1343970067323695105","1342156765228576771","1343259117633363968","1341815666563833856","1342189758647324675"],["2020-12-29T17:20:08Z","2020-12-24T17:14:43Z","2020-12-27T18:15:04Z","2020-12-23T18:39:19Z","2020-12-24T19:25:49Z"],["PhillyInquirer","PhillyInquirer","PhillyInquirer","PhillyInquirer","PhillyInquirer"],["Union to sell Mark McKenzie to Belgium’s KRC Genk https://t.co/4GD0RsRv8s","It's only the first game ... right? https://t.co/mFsDogez06","Today's point of emphasis for the #Eagles against the #Cowboys: third down. @pdomo lays out what the Birds need to do to get the win. https://t.co/UwR050XZc8","COVID-19 won't stop South Philadelphians from standing outside in the cold for Christmas cannoli. Some traditions never die. https://t.co/95oXsigsko","New art installation in jeopardy after mural of queer activist Gloria Casarez in Philly’s Gayborhood whitewashed without warning https://t.co/sAOX0BSTz0"],["SocialFlow","SocialFlow","SocialFlow","SocialFlow","SocialFlow"],[73,59,157,148,152],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[false,false,false,false,false],[false,false,false,false,false],[0,5,4,21,4],[0,1,2,2,8],[null,null,null,null,null],[null,null,null,null,null],[null,null,["Eagles","Cowboys"],null,null],[null,null,null,null,null],["trib.al/wpKTAzt","trib.al/fAjShdh","trib.al/Ukuhrda","trib.al/c1It62G","trib.al/MRKrVR2"],["https://t.co/4GD0RsRv8s","https://t.co/mFsDogez06","https://t.co/UwR050XZc8","https://t.co/95oXsigsko","https://t.co/sAOX0BSTz0"],["https://trib.al/wpKTAzt","https://trib.al/fAjShdh","https://trib.al/Ukuhrda","https://trib.al/c1It62G","https://trib.al/MRKrVR2"],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,"168750735",null,null],[null,null,"pdomo",null,null],["de","en","en","en","en"],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[null,null,null,null,null],[[null,null],[null,null],[null,null],[null,null],[null,null]],[[null,null],[null,null],[null,null],[null,null],[null,null]],[[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null]],["https://twitter.com/PhillyInquirer/status/1343970067323695105","https://twitter.com/PhillyInquirer/status/1342156765228576771","https://twitter.com/PhillyInquirer/status/1343259117633363968","https://twitter.com/PhillyInquirer/status/1341815666563833856","https://twitter.com/PhillyInquirer/status/1342189758647324675"],["The Philadelphia Inquirer","The Philadelphia Inquirer","The Philadelphia Inquirer","The Philadelphia Inquirer","The Philadelphia Inquirer"],["Philadelphia, PA","Philadelphia, PA","Philadelphia, PA","Philadelphia, PA","Philadelphia, PA"],["The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs.","The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs.","The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs.","The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs.","The Philadelphia Inquirer is your front-row seat to the Greater Philadelphia region. You can support local, impactful journalism at https://t.co/Dw4BC5NdNs."],["https://t.co/QeWyyrUeoF","https://t.co/QeWyyrUeoF","https://t.co/QeWyyrUeoF","https://t.co/QeWyyrUeoF","https://t.co/QeWyyrUeoF"],[false,false,false,false,false],[418589,418589,418589,418589,418589],[1591,1591,1591,1591,1591],[2966,2966,2966,2966,2966],[199157,199157,199157,199157,199157],[705,705,705,705,705],["2008-03-26T01:57:54Z","2008-03-26T01:57:54Z","2008-03-26T01:57:54Z","2008-03-26T01:57:54Z","2008-03-26T01:57:54Z"],[true,true,true,true,true],["https://t.co/QeWyyrUeoF","https://t.co/QeWyyrUeoF","https://t.co/QeWyyrUeoF","https://t.co/QeWyyrUeoF","https://t.co/QeWyyrUeoF"],["http://www.inquirer.com","http://www.inquirer.com","http://www.inquirer.com","http://www.inquirer.com","http://www.inquirer.com"],[null,null,null,null,null],["https://pbs.twimg.com/profile_banners/14221917/1559300463","https://pbs.twimg.com/profile_banners/14221917/1559300463","https://pbs.twimg.com/profile_banners/14221917/1559300463","https://pbs.twimg.com/profile_banners/14221917/1559300463","https://pbs.twimg.com/profile_banners/14221917/1559300463"],["http://abs.twimg.com/images/themes/theme14/bg.gif","http://abs.twimg.com/images/themes/theme14/bg.gif","http://abs.twimg.com/images/themes/theme14/bg.gif","http://abs.twimg.com/images/themes/theme14/bg.gif","http://abs.twimg.com/images/themes/theme14/bg.gif"],["http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png","http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png","http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png","http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png","http://pbs.twimg.com/profile_images/1134416080645083138/U3Kox7Ld_normal.png"],[null,null,null,null,null],[null,null,null,null,null],["2020-12-29T17:20:08Z","2020-12-24T17:14:43Z","2020-12-27T18:15:04Z","2020-12-23T18:39:19Z","2020-12-24T19:25:49Z"],["2020-12-29","2020-12-24","2020-12-27","2020-12-23","2020-12-24"],[2020,2020,2020,2020,2020],[12,12,12,12,12],[29,24,27,23,24]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>user_id<\/th>\n      <th>status_id<\/th>\n      <th>created_at<\/th>\n      <th>screen_name<\/th>\n      <th>text<\/th>\n      <th>source<\/th>\n      <th>display_text_width<\/th>\n      <th>reply_to_status_id<\/th>\n      <th>reply_to_user_id<\/th>\n      <th>reply_to_screen_name<\/th>\n      <th>is_quote<\/th>\n      <th>is_retweet<\/th>\n      <th>favorite_count<\/th>\n      <th>retweet_count<\/th>\n      <th>quote_count<\/th>\n      <th>reply_count<\/th>\n      <th>hashtags<\/th>\n      <th>symbols<\/th>\n      <th>urls_url<\/th>\n      <th>urls_t.co<\/th>\n      <th>urls_expanded_url<\/th>\n      <th>media_url<\/th>\n      <th>media_t.co<\/th>\n      <th>media_expanded_url<\/th>\n      <th>media_type<\/th>\n      <th>ext_media_url<\/th>\n      <th>ext_media_t.co<\/th>\n      <th>ext_media_expanded_url<\/th>\n      <th>ext_media_type<\/th>\n      <th>mentions_user_id<\/th>\n      <th>mentions_screen_name<\/th>\n      <th>lang<\/th>\n      <th>quoted_status_id<\/th>\n      <th>quoted_text<\/th>\n      <th>quoted_created_at<\/th>\n      <th>quoted_source<\/th>\n      <th>quoted_favorite_count<\/th>\n      <th>quoted_retweet_count<\/th>\n      <th>quoted_user_id<\/th>\n      <th>quoted_screen_name<\/th>\n      <th>quoted_name<\/th>\n      <th>quoted_followers_count<\/th>\n      <th>quoted_friends_count<\/th>\n      <th>quoted_statuses_count<\/th>\n      <th>quoted_location<\/th>\n      <th>quoted_description<\/th>\n      <th>quoted_verified<\/th>\n      <th>retweet_status_id<\/th>\n      <th>retweet_text<\/th>\n      <th>retweet_created_at<\/th>\n      <th>retweet_source<\/th>\n      <th>retweet_favorite_count<\/th>\n      <th>retweet_retweet_count<\/th>\n      <th>retweet_user_id<\/th>\n      <th>retweet_screen_name<\/th>\n      <th>retweet_name<\/th>\n      <th>retweet_followers_count<\/th>\n      <th>retweet_friends_count<\/th>\n      <th>retweet_statuses_count<\/th>\n      <th>retweet_location<\/th>\n      <th>retweet_description<\/th>\n      <th>retweet_verified<\/th>\n      <th>place_url<\/th>\n      <th>place_name<\/th>\n      <th>place_full_name<\/th>\n      <th>place_type<\/th>\n      <th>country<\/th>\n      <th>country_code<\/th>\n      <th>geo_coords<\/th>\n      <th>coords_coords<\/th>\n      <th>bbox_coords<\/th>\n      <th>status_url<\/th>\n      <th>name<\/th>\n      <th>location<\/th>\n      <th>description<\/th>\n      <th>url<\/th>\n      <th>protected<\/th>\n      <th>followers_count<\/th>\n      <th>friends_count<\/th>\n      <th>listed_count<\/th>\n      <th>statuses_count<\/th>\n      <th>favourites_count<\/th>\n      <th>account_created_at<\/th>\n      <th>verified<\/th>\n      <th>profile_url<\/th>\n      <th>profile_expanded_url<\/th>\n      <th>account_lang<\/th>\n      <th>profile_banner_url<\/th>\n      <th>profile_background_url<\/th>\n      <th>profile_image_url<\/th>\n      <th>lat<\/th>\n      <th>lng<\/th>\n      <th>tweet_datetime<\/th>\n      <th>tweet_date<\/th>\n      <th>tweet_year<\/th>\n      <th>tweet_month<\/th>\n      <th>tweet_day<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":[7,13,14,15,16,37,38,42,43,44,52,53,57,58,59,78,79,80,81,82,91,92,95,96,97]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


### Examing data

Now going to identify a few basic things, but first, this df has 92 columns. What are some of the columns that we're most likely to use? The `skimr` package is **really** good at doing this and is a good substitute for `summary()` in many cases.





```r
tweets_df %>% select(
  # About the tweet
  text
  , lang
  , hashtags
  , symbols
  , favorite_count
  , retweet_count
  , lat
  , lng
  # About the user
  , name
  , screen_name
  , followers_count
  , friends_count
  , location
  , description
  , listed_count
  , statuses_count
  , created_at
  , account_created_at
  ) %>%
  head()
## # A tibble: 6 x 18
##   text  lang  hashtags symbols favorite_count retweet_count   lat   lng name 
##   <chr> <chr> <list>   <list>           <int>         <int> <dbl> <dbl> <chr>
## 1 "Uni~ de    <chr [1~ <chr [~              0             0    NA    NA The ~
## 2 "It'~ en    <chr [1~ <chr [~              5             1    NA    NA The ~
## 3 "Tod~ en    <chr [2~ <chr [~              4             2    NA    NA The ~
## 4 "COV~ en    <chr [1~ <chr [~             21             2    NA    NA The ~
## 5 "New~ en    <chr [1~ <chr [~              4             8    NA    NA The ~
## 6 "<U+274C> S~ en    <chr [1~ <chr [~              5             2    NA    NA The ~
## # ... with 9 more variables: screen_name <chr>, followers_count <int>,
## #   friends_count <int>, location <chr>, description <chr>, listed_count <int>,
## #   statuses_count <int>, created_at <dttm>, account_created_at <dttm>
```

### Sample stats & visualizations  

What is our total tweet *count*?   


```r
nrow(tweets_df)
## [1] 1897
```

What is the most *favorited* tweet?  


```r
tweets_df %>%
  slice_max(favorite_count, n = 1) %>% # can also use arrange()
  select(screen_name
         , text
         , favorite_count
         , created_at)
## # A tibble: 1 x 4
##   screen_name text                            favorite_count created_at         
##   <chr>       <chr>                                    <int> <dttm>             
## 1 GetUpESPN   "\"Carson Wentz is soft! ... H~           1993 2020-12-21 16:16:24
```


What users are *tweeting the most*?  


```r
top_tweeters <- tweets_df %>%
  group_by(screen_name) %>%
  summarize(num_tweets = n()) %>%
  arrange(-num_tweets) %>% # can also use slice_max()
  ungroup() %>%
  head(5)

ggplot(top_tweeters) +
  geom_histogram(aes(x = screen_name, y = num_tweets, fill = screen_name), stat = 'identity') +
  theme_minimal() +
  scale_fill_viridis_d(option = "viridis") +
  labs(title = "What users are tweeting the most?") +
  theme(legend.position = "none") +
  xlab("") +
  ylab("Number of Tweets")
```

<img src="/post/2020-12-30-twitter-sentiment-philly/index_files/figure-html/statsthree-1.png" width="672" />


In what *languages* are they tweeting?  

Languages are recorded here as [alpha-3 / ISO 639.2 codes](https://www.loc.gov/standards/iso639-2/php/code_list.php).  


```r
tweets_df %>%
  group_by(lang) %>%
  summarize(num_tweets = n()) %>%
  arrange(-num_tweets) %>% # can also use slice_max()
  head(10)
## # A tibble: 10 x 2
##    lang  num_tweets
##    <chr>      <int>
##  1 en          1829
##  2 und           34
##  3 cy             8
##  4 es             6
##  5 de             3
##  6 et             3
##  7 ja             3
##  8 fr             2
##  9 pt             2
## 10 ro             2

# ggplot(tweets_df) +
#   geom_bar(aes(x = lang, fill = lang))
```


What *platforms* are people using?  


```r
users_by_source <- tweets_df %>%
  group_by(source) %>%
  summarize(num_users = n_distinct(user_id)) %>%
  arrange(-num_users) %>% # can also use slice_max()
  ungroup() %>%
  head(10)

ggplot(users_by_source) +
  geom_histogram(aes(x = reorder(source, num_users), y = num_users, fill = source), stat = 'identity') +
  scale_fill_viridis_d(option = "viridis") +
  coord_flip() +
  theme_gray() +
  theme(legend.position = "none") +
  #theme(legend.title = element_blank()) +
  labs(title = "Top 10 Twitter Platforms People are Using") +
  xlab("") +
  ylab("Number of Users") +
  scale_y_continuous(breaks = seq(0, 400, by=50), limits = c(0, 400), expand = c(0, 0)) + # use expand() to remove empty space between axis and bars
  geom_hline(yintercept = seq(0, 400, by = 25), color = 'white')
```

<img src="/post/2020-12-30-twitter-sentiment-philly/index_files/figure-html/statsfive-1.png" width="672" />


*How many* tweets per day?  

Going to create a chart with [ggplot2 and plotly](https://plotly-r.com/improving-ggplotly.html) and a second one with [dygraphs for r](https://rstudio.github.io/dygraphs/index.html).  


```r
tweets_per_day <- tweets_df %>%
  group_by(tweet_date) %>%
  summarize(num_tweets = n()) %>%
  arrange(-num_tweets) %>% # can also use slice_max()
  ungroup()

# Using ggplot + plotly
library(plotly)
plt1 <- ggplot(tweets_per_day) +
  geom_line(aes(x = tweet_date, y = num_tweets)) +
  scale_fill_viridis_d(option = "viridis") +
  theme_gray() +
  theme(legend.position = "none") +
  labs(title = "Daily Tweets") +
  xlab("") +
  ylab("Number of Tweets") 

plotly::ggplotly(plt1)
```

<!--html_preserve--><div id="htmlwidget-755369a1bc8ac2e6b05b" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-755369a1bc8ac2e6b05b">{"x":{"data":[{"x":[18617,18618,18619,18620,18621,18622,18623,18624,18625],"y":[194,287,290,240,159,138,203,240,146],"text":["tweet_date: 2020-12-21<br />num_tweets: 194","tweet_date: 2020-12-22<br />num_tweets: 287","tweet_date: 2020-12-23<br />num_tweets: 290","tweet_date: 2020-12-24<br />num_tweets: 240","tweet_date: 2020-12-25<br />num_tweets: 159","tweet_date: 2020-12-26<br />num_tweets: 138","tweet_date: 2020-12-27<br />num_tweets: 203","tweet_date: 2020-12-28<br />num_tweets: 240","tweet_date: 2020-12-29<br />num_tweets: 146"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(0,0,0,1)","dash":"solid"},"hoveron":"points","showlegend":false,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":25.5707762557078,"l":43.1050228310502},"plot_bgcolor":"rgba(235,235,235,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"title":{"text":"Daily Tweets","font":{"color":"rgba(0,0,0,1)","family":"","size":17.5342465753425},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[18616.6,18625.4],"tickmode":"array","ticktext":["Dec 22","Dec 24","Dec 26","Dec 28"],"tickvals":[18618,18620,18622,18624],"categoryorder":"array","categoryarray":["Dec 22","Dec 24","Dec 26","Dec 28"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":{"text":"","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[130.4,297.6],"tickmode":"array","ticktext":["150","200","250"],"tickvals":[150,200,250],"categoryorder":"array","categoryarray":["150","200","250"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":{"text":"Number of Tweets","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":false,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895}},"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","showSendToCloud":false},"source":"A","attrs":{"28346fdd5239":{"x":{},"y":{},"type":"scatter"}},"cur_data":"28346fdd5239","visdat":{"28346fdd5239":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


```r
# Using dygraph
# convert series to an xts object 
tweets_ts <- xts::xts(tweets_per_day$num_tweets, order.by = tweets_per_day$tweet_date)
# Create the dygraph
library(dygraphs)
dygraph(tweets_ts, main = "Daily Tweets for 'Philadelphia'") %>%
  dySeries("V1", label = "Tweet Count")
```


And *where* are people tweeting?

Of the 1897 tweets that we have, 133 are geotagged. We'll plot those using [leaflet](https://rstudio.github.io/leaflet/) and build on this later.


```r
library(leaflet)

tweet_map <- leaflet(tweets_df) %>%
  addTiles() %>%
  addCircleMarkers(lng = ~lng
                   , lat = ~lat)

htmlwidgets::saveWidget(widget = tweet_map
                        , 'tweet_map.html'
                        , selfcontained = TRUE)
```

<!-- ```{r mapshow} -->
<!-- knitr::include_url(url = 'tweet_map.html', height = "400px") -->
<!-- ``` -->


<iframe src="tweet_map.html" width="600" height="400" style="border: none;"></iframe>


### Sentiment analysis

Now switching over to [tidytext](https://www.tidytextmining.com/topicmodeling.html)  

> The function `get_sentiments()` allows us to get specific sentiment lexicons with the appropriate measures for each one.

Preview some of the sentiment lexicons. You'll be prompted to download the `textdata` package and the `afinn`, `bing`, and `nrc` lexicons if you don't have them already.

**[afinn](http://www2.imm.dtu.dk/pubdb/pubs/6010-full.html)** - assigns a numeric value to each word on a scale of -5 to +5
**[bing](https://www.cs.uic.edu/~liub/FBS/sentiment-analysis.html)** - binary assignment to each word, e.g. 'positive' or 'negative'  
**[nrc](http://saifmohammad.com/WebPages/NRC-Emotion-Lexicon.htm)** - categorizes words into categories: positive, negative, anger, anticipation, disgust, fear, joy, sadness, surprise, and trust  


```r
#install.packages("textdata")
library(tidytext)

get_sentiments("afinn") %>% head()
## # A tibble: 6 x 2
##   word       value
##   <chr>      <dbl>
## 1 abandon       -2
## 2 abandoned     -2
## 3 abandons      -2
## 4 abducted      -2
## 5 abduction     -2
## 6 abductions    -2
get_sentiments("bing") %>% head()
## # A tibble: 6 x 2
##   word       sentiment
##   <chr>      <chr>    
## 1 2-faces    negative 
## 2 abnormal   negative 
## 3 abolish    negative 
## 4 abominable negative 
## 5 abominably negative 
## 6 abominate  negative
get_sentiments("nrc") %>% head()
## # A tibble: 6 x 2
##   word      sentiment
##   <chr>     <chr>    
## 1 abacus    trust    
## 2 abandon   fear     
## 3 abandon   negative 
## 4 abandon   sadness  
## 5 abandoned anger    
## 6 abandoned fear
```


Now tidy up the data and prepare it for analysis. If running this with the `austen_books()` example from the [tidytext docs](https://www.tidytextmining.com/sentiment.html), it's relatively straightforward:


```r
library(stringr)
library(janeaustenr)

tidy_books <- austen_books() %>%
  group_by(book) %>%
  mutate(
    linenumber = row_number(),
    chapter = cumsum(str_detect(text, 
                                regex("^chapter [\\divxlc]", 
                                      ignore_case = TRUE)))) %>%
  ungroup() %>%
  unnest_tokens(word, text)

tidy_books %>% head()
## # A tibble: 6 x 4
##   book                linenumber chapter word       
##   <fct>                    <int>   <int> <chr>      
## 1 Sense & Sensibility          1       0 sense      
## 2 Sense & Sensibility          1       0 and        
## 3 Sense & Sensibility          1       0 sensibility
## 4 Sense & Sensibility          3       0 by         
## 5 Sense & Sensibility          3       0 jane       
## 6 Sense & Sensibility          3       0 austen
```


But I want to run this on tweets, so I need to reshape the data. I only executed one search for `philadelphia+philly`, so I'm going to build a dataframe from that. Though, we can run multiple searches and assign each search its own `book` value to enable comparisons and faceted plots later.  



```r
tweets_book <- cbind('Philadelphia', tweets_df$text) %>% data.frame()
colnames(tweets_book) <- c('book', 'text')

# Reformat the date - running above insists on converting the `tweet_date` value to its integer form.
tweets_book$chapter <- tweets_df$tweet_date

tidy_books <- tweets_book %>%
  group_by(book) %>%
  mutate(
    linenumber = row_number()
    ) %>%
  ungroup() %>%
  unnest_tokens(word, text)

tidy_books %>% head()
## # A tibble: 6 x 4
##   book         chapter    linenumber word    
##   <chr>        <date>          <int> <chr>   
## 1 Philadelphia 2020-12-29          1 union   
## 2 Philadelphia 2020-12-29          1 to      
## 3 Philadelphia 2020-12-29          1 sell    
## 4 Philadelphia 2020-12-29          1 mark    
## 5 Philadelphia 2020-12-29          1 mckenzie
## 6 Philadelphia 2020-12-29          1 to
```


Now the tweets are in a tidy format with one word per row.  

Looking at the `nrc` lexicon, what are some of the "joy" words?


```r
nrc_joy <- get_sentiments("nrc") %>% 
  filter(sentiment == "joy")

head(nrc_joy)
## # A tibble: 6 x 2
##   word          sentiment
##   <chr>         <chr>    
## 1 absolution    joy      
## 2 abundance     joy      
## 3 abundant      joy      
## 4 accolade      joy      
## 5 accompaniment joy      
## 6 accomplish    joy
```

How many "joy" words do we have in our tweets?


```r
tidy_books %>%
  filter(book == "Philadelphia") %>%
  inner_join(nrc_joy) %>%
  count(word, sort = TRUE) %>%
  head(10)
## # A tibble: 10 x 2
##    word        n
##    <chr>   <int>
##  1 love       58
##  2 good       56
##  3 holiday    52
##  4 art        36
##  5 food       29
##  6 music      27
##  7 deal       24
##  8 save       24
##  9 safe       23
## 10 happy      21
```

Using `tidyr`, we can continue to examine how sentiment changes throughout the book. This is more relevant to the novels examples from the docs, but I'm including it here as an exercise. This example will use the `bing` lexicon. We need some type of interval - the example creates an index from 80 line chunks, but I'm going to update this to use the `chapter` value, which now represents the date of each tweet allowing to see changes in sentiment over time.

From the docs:  

> ... First, we find a sentiment score for each word using the Bing lexicon and inner_join(). 

> Next, we count up how many positive and negative words there are in defined sections of each book. We define an index here to keep track of where we are in the narrative; this index (using integer division) counts up sections of 80 lines of text.  

> The %/% operator does integer division (x %/% y is equivalent to floor(x/y)) so the index keeps track of which 80-line section of text we are counting up negative and positive sentiment in.  

> Small sections of text may not have enough words in them to get a good estimate of sentiment while really large sections can wash out narrative structure. For these books, using 80 lines works well, but this can vary depending on individual texts, how long the lines were to start with, etc. We then use spread() so that we have negative and positive sentiment in separate columns, and lastly calculate a net sentiment (positive - negative).  


```r
philly_sentiment <- tidy_books %>%
  inner_join(get_sentiments("bing")) %>%
  #count(book, index = linenumber %/% 80, sentiment) %>% # 80-line tweet chunks
  count(book, index = chapter, sentiment) %>%
  spread(sentiment, n, fill = 0) %>%
  mutate(sentiment = positive - negative)
```


```r
ggplot(philly_sentiment) +
  geom_col(aes(index, sentiment, fill = sentiment)) +
  #facet_wrap(~book, ncol = 1, scales = "free_x") # apply if we'relooking across multiple books +
  labs(title = "Sentiment of 'Philadelphia' Tweets") +
  scale_fill_gradient2(low='red', mid='orange', high='navy') +
  #scale_fill_viridis_c(option = "viridis") +
  theme(legend.position = "none") +
  xlab("") +
  ylab("Sentiment")
```

<img src="/post/2020-12-30-twitter-sentiment-philly/index_files/figure-html/sentimentsplotone-1.png" width="672" />




### Comparing sentiments  

Going to compare the different sentiment lexicons. Still writing the code as if there are multiple books (or search results) in the dataframe.  


```r
philly_tweets <- tidy_books %>% 
  filter(book == "Philadelphia")
```


Using `dplyr::inner_join()`, estimate the different sentiment values for each lexicon.


```r
## afinn
afinn <- philly_tweets %>% 
  inner_join(get_sentiments("afinn")) %>% 
  #group_by(index = linenumber %/% 80) %>% # Can use 80-line chunks
  group_by(index = chapter) %>% # or other type of index value, like date
  summarise(sentiment = sum(value)) %>% 
  mutate(method = "AFINN")

## rbinding bing and nrc
bing_and_nrc <- bind_rows(
  philly_tweets %>% 
    inner_join(get_sentiments("bing")) %>%
    mutate(method = "Bing et al."),
  philly_tweets %>% 
    inner_join(get_sentiments("nrc") %>% 
                 filter(sentiment %in% c("positive",
                                         "negative"))
    ) %>%
    mutate(method = "NRC")) %>%
  #count(method, index = linenumber %/% 80, sentiment) %>% # Can use 80-line chunks
  count(method, index = chapter, sentiment) %>% # or other type of index value, like date
  spread(sentiment, n, fill = 0) %>%
  mutate(sentiment = positive - negative)
```


Now combine and plot them together.  


```r
bind_rows(afinn, 
          bing_and_nrc) %>%
  ggplot(aes(index, sentiment, fill = method)) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~method, ncol = 1, scales = "free_y") +
  labs(title = "Sentiment of 'Philadelphia' Tweets - Lexicon Comparison") +
  scale_fill_viridis_d(option = "viridis") +
  theme(legend.position = "none") +
  xlab("") +
  ylab("Sentiment")
```

<img src="/post/2020-12-30-twitter-sentiment-philly/index_files/figure-html/sentimentplottwo-1.png" width="672" />



### Some routine checks  

When comparing results across different lexicons, consider word counts and the ratio of positive words to negative words.  

TODO: Update include and eval settings here - ok to keep in md doc but no need to knit them  


```r
get_sentiments("nrc") %>% 
  filter(sentiment %in% c("positive", "negative")) %>% 
  count(sentiment)
## # A tibble: 2 x 2
##   sentiment     n
##   <chr>     <int>
## 1 negative   3324
## 2 positive   2312

get_sentiments("bing") %>% 
  count(sentiment)
## # A tibble: 2 x 2
##   sentiment     n
##   <chr>     <int>
## 1 negative   4781
## 2 positive   2005
```


```r
bing_word_counts <- tidy_books %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  ungroup()

bing_word_counts %>% head(25)
## # A tibble: 25 x 3
##    word       sentiment     n
##    <chr>      <chr>     <int>
##  1 corruption negative    120
##  2 like       positive     74
##  3 love       positive     58
##  4 great      positive     57
##  5 good       positive     56
##  6 best       positive     50
##  7 hurts      negative     42
##  8 well       positive     38
##  9 work       positive     29
## 10 free       positive     28
## # ... with 15 more rows
```


What are the top words driving sentiment score?  


```r
bing_word_counts %>%
  group_by(sentiment) %>%
  top_n(10) %>%
  ungroup() %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(n, word, fill = sentiment)) +
  geom_col(show.legend = FALSE) +
  scale_fill_viridis_d(option = "viridis") +
  facet_wrap(~sentiment, scales = "free_y") +
  labs(x = "Contribution to sentiment",
       y = NULL)
```

<img src="/post/2020-12-30-twitter-sentiment-philly/index_files/figure-html/sentplotfour-1.png" width="672" />



### Wordclouds  

I don't find wordclouds terrible useful, but they're not terrible for a background graphic or an attention grabber.

First, let's add some custom stop words (i.e. words to exclude from the analysis). If you want to know why, run the next chunk and see how URLs (and obviously our original search terms) throw off our term frequency matrix.   


```r
custom_stop_words <- bind_rows(tibble(word = c('philadelphia', 'philly', 't.co', 'https', 'amp'),
                                      lexicon = c("custom")),
                               stop_words)

custom_stop_words %>% head()
## # A tibble: 6 x 2
##   word         lexicon
##   <chr>        <chr>  
## 1 philadelphia custom 
## 2 philly       custom 
## 3 t.co         custom 
## 4 https        custom 
## 5 amp          custom 
## 6 a            SMART
```



```r
library(wordcloud)

tidy_books %>%
  anti_join(custom_stop_words) %>%
  count(word) %>%
  with(wordcloud(word, n, max.words = 100, col = brewer.pal(8, "Dark2")))
```

<img src="/post/2020-12-30-twitter-sentiment-philly/index_files/figure-html/wordcloudone-1.png" width="672" />

Going to compare the most popular positive and negative words using `comparison.cloud()`. Keep a look out for pronouns that get mistaken for positive or negative words (e.g. 'hurts' as in 'jalen hurts', though you might leave that in depending on how you feel about the Eagles qb situation).  


```r
library(reshape2)

tidy_books %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  acast(word ~ sentiment, value.var = "n", fill = 0) %>%
  comparison.cloud(colors = c("orange", "navy"),
                   max.words = 100)
```

<img src="/post/2020-12-30-twitter-sentiment-philly/index_files/figure-html/wordcloudtwo-1.png" width="672" />


### What about sentences?  

Rather than assign sentiment to words, some algorithms attempt to derive sentiment from complete sentences, e.g. "I am not having a good day" should imply negative sentiment, not positive. Our tweets are already pretty succinct, so we can try to look at the entire tweek to make an assessment.    

TODO: Fix how this is parsing sentences. See [docs](https://www.tidytextmining.com/sentiment.html#looking-at-units-beyond-just-words)  



Using sentences, or some other type of index, we can assign a sentiment value - not by word - but by an entire chapter.  

Which chapter (date) had the *highest proportion* of positive words?


```r
bingpositive <- get_sentiments("bing") %>% 
  filter(sentiment == "positive")

wordcounts <- tidy_books %>%
  group_by(book, chapter) %>%
  summarize(words = n())

tidy_books %>%
  semi_join(bingpositive) %>%
  group_by(book, chapter) %>%
  summarize(positivewords = n()) %>%
  left_join(wordcounts, by = c("book", "chapter")) %>%
  mutate(positive_ratio = positivewords/words) %>%
  filter(chapter != 0) %>%
  top_n(1) %>%
  ungroup()
## # A tibble: 1 x 5
##   book         chapter    positivewords words positive_ratio
##   <chr>        <date>             <int> <int>          <dbl>
## 1 Philadelphia 2020-12-21           196  5755         0.0341
```



<!-- TO BE CONTINUED -->

<!-- ### Condition the data -->

<!-- Pre-process the data (source: bryangoodrich). -->

<!-- ```{r, eval = FALSE, include = FALSE} -->
<!-- tweets <- iconv(tweets, to = "ASCII", sub = " ")  # Convert to basic ASCII text -->
<!-- tweets <- tolower(tweets)  # Make everything lower case -->
<!-- tweets <- gsub("rt", " ", tweets)  # Remove the "RT" (retweet) so duplicates are duplicates -->
<!-- tweets <- gsub("@\\w+", " ", tweets)  # Remove user names -->
<!-- tweets <- gsub("http.+ |http.+$", " ", tweets)  # Remove links -->
<!-- tweets <- gsub("[[:punct:]]", " ", tweets)  # Remove punctuation -->
<!-- tweets <- gsub("[ |\t]{2,}", " ", tweets)  # Remove tabs -->
<!-- tweets <- gsub("amp", " ", tweets)  # "&" is "&amp" in HTML, so after punctuation removed ... -->
<!-- tweets <- gsub("^ ", "", tweets)  # Leading blanks -->
<!-- tweets <- gsub(" $", "", tweets)  # Lagging blanks -->
<!-- tweets <- gsub(" +", " ", tweets) # General spaces (should just do all whitespaces no?) -->
<!-- tweets <- unique(tweets)  # Now get rid of duplicates! -->
<!-- ``` -->

<!-- Convert the `tweets` object to a corpus object to use with the `tm` package. -->

<!-- ```{r tm_corpus, eval = FALSE, include = FALSE} -->
<!-- corpus <- tm::Corpus(VectorSource(tweets))  -->
<!-- ``` -->

<!-- Remove English [stop words](https://rdrr.io/rforge/tm/man/stopwords.html). _Also see [Adding custom stopwords in R tm](https://stackoverflow.com/questions/18446408/adding-custom-stopwords-in-r-tm)_.   -->

<!-- ```{r stopwordstwo, eval = FALSE, include = FALSE} -->
<!-- corpus <- tm_map(corpus, removeWords, stopwords("en"))   -->
<!-- ``` -->

<!-- Remove the search terms from the corpus.  -->

<!-- _TODO: Make these variables and use paste() to place them in the search term earlier._ -->

<!-- ```{r rmsearchterms, eval = FALSE, include = FALSE} -->
<!-- #corpus <- tm_map(corpus, removeWords, c("energy", "electricity")) -->
<!-- corpus <- tm_map(corpus, removeWords, c('tesla', 'tsla')) -->
<!-- ``` -->

<!-- Remove numbers. -->

<!-- ```{r numbers, eval = FALSE, include = FALSE} -->
<!-- corpus <- tm_map(corpus, removeNumbers, mc.cores=1) -->
<!-- ``` -->

<!-- Stem the words  (e.g. likes, liked, likely, liking >> like). _For more on stemming and lemmatization, see [NLP Stanford](https://nlp.stanford.edu/IR-book/html/htmledition/stemming-and-lemmatization-1.html)_. -->

<!-- ```{r stemming, eval = FALSE, include = FALSE} -->
<!-- corpus <- tm_map(corpus, stemDocument) -->
<!-- ``` -->


<!-- ### Visualization -->

<!-- Display the corpus as a word cloud. -->

<!-- ```{r wordcloud, eval = FALSE, include = FALSE} -->
<!-- pal <- brewer.pal(8, "Dark2") -->
<!-- wordcloud(corpus, min.freq=2, max.words = 150, random.order = TRUE, col = pal) -->
<!-- ``` -->


<!-- ### Topic Modeling -->

<!-- Get the lengths and make sure we only create a Document Term Matrix (DTM) for tweets with actual content. -->

<!-- ```{r dtm, eval = FALSE, include = FALSE} -->
<!-- doc_lengths <- corpus %>% DocumentTermMatrix() %>% as.matrix() %>% rowSums() -->

<!-- dtm <- corpus[doc_lengths > 0] %>% DocumentTermMatrix() -->
<!-- ``` -->

<!-- Test a Latent Dirichlet Allocation model; A LDA_VEM topic model with 2 topics. -->

<!-- ```{r lda, eval = FALSE, include = FALSE} -->
<!-- lda <- LDA(x = dtm, k = 2, method = 'VEM', control = list(seed = 1234)) -->
<!-- ``` -->

<!-- Explore it a bit with `tidytext` ([documentation](https://www.tidytextmining.com/topicmodeling.html)) -->

<!-- > tidytext package provides this method for extracting the per-topic-per-word probabilities, called  -->
<!-- β (“beta”), from the model ... For each combination, the model computes the probability of that term being generated from that topic -->

<!-- ```{r lda_topics, eval = FALSE, include = FALSE} -->
<!-- topics <- tidy(lda, matrix = "beta") -->
<!-- topics %>% arrange(-beta) %>% head(50) -->
<!-- ``` -->

<!-- Display top terms for each topic. -->

<!-- ```{r topterms, eval = FALSE, include = FALSE} -->
<!-- top_terms <- topics %>% -->
<!--   group_by(topic) %>% -->
<!--   top_n(10, beta) %>% -->
<!--   ungroup() %>% -->
<!--   arrange(topic, -beta) -->

<!-- top_terms %>% -->
<!--   mutate(term = reorder_within(term, beta, topic)) %>% -->
<!--   ggplot(aes(beta, term, fill = factor(topic))) + -->
<!--   geom_col(show.legend = FALSE) + -->
<!--   facet_wrap(~ topic, scales = "free") + -->
<!--   scale_y_reordered() + -->
<!--   labs( -->
<!--     title = 'Top Terms in Twitter Corpus' -->
<!--     , subtitle = 'The terms that are most common within each topic' -->
<!--   ) -->

<!-- ``` -->


<!-- ```{r betaspread, eval = FALSE, include = FALSE} -->
<!-- beta_spread <- topics %>% -->
<!--   mutate(topic = paste0("topic", topic)) %>% -->
<!--   spread(topic, beta) %>% -->
<!--   filter(topic1 > .001 | topic2 > .001) %>% -->
<!--   mutate(log_ratio = log2(topic2 / topic1)) -->

<!-- beta_spread -->
<!-- ``` -->


<!-- Document topic probabilities. -->

<!-- > Each of these values is an estimated proportion of words from that document that are generated from that topic. For example, the model estimates that only about 25% of the words in document 1 were generated from topic 1. -->

<!-- ```{r doctopics, eval = FALSE, include = FALSE} -->
<!-- documents <- tidy(lda, matrix = "gamma") -->
<!-- documents %>% arrange(document) %>% head(50) -->
<!-- ``` -->






<!-- # Now for some topics -->

<!-- ```{r, eval = FALSE, include = FALSE} -->
<!-- SEED = sample(1:1000000, 1)  # Pick a random seed for replication -->
<!-- k = 10  # Let's start with 10 topics -->
<!-- ``` -->

<!-- ```{r, eval = FALSE, include = FALSE} -->
<!-- # This might take a minute! -->
<!-- models <- list( -->
<!--     CTM       = CTM(dtm, k = k, control = list(seed = SEED, var = list(tol = 10^-4), em = list(tol = 10^-3))), -->
<!--     VEM       = LDA(dtm, k = k, control = list(seed = SEED)), -->
<!--     VEM_Fixed = LDA(dtm, k = k, control = list(estimate.alpha = FALSE, seed = SEED)), -->
<!--     Gibbs     = LDA(dtm, k = k, method = "Gibbs", control = list(seed = SEED, burnin = 1000, -->
<!--                                                                  thin = 100,    iter = 1000)) -->
<!-- ) -->
<!-- ``` -->


<!-- # There you have it. Models now holds 4 topics. See the topicmodels API documentation for details -->

<!-- # Top 10 terms of each topic for each model -->
<!-- # Do you see any themes you can label to these "topics" (lists of words)? -->

<!-- ```{r, eval = FALSE, include = FALSE} -->
<!-- lapply(models, terms, 10) -->
<!-- ``` -->

<!-- # matrix of tweet assignments to predominate topic on that tweet for each of the models, in case you wanted to categorize them -->

<!-- ```{r, eval = FALSE, include = FALSE} -->
<!-- assignments <- sapply(models, topics) -->
<!-- ``` -->




<!-- ### Scrapyard -->

<!-- Configuring session for use with `twitteR` package   -->

<!-- ```{r configtwitteR, eval = FALSE, include = FALSE} -->
<!-- config <- config::get(file = 'config.yml') -->

<!-- my_config <- function() { -->
<!--     ckey = config$apikey -->
<!--     csec = config$apikeysecret -->
<!--     akey = config$accesstoken -->
<!--     asec = config$accesstokensecret -->

<!--     setup_twitter_oauth(ckey, csec, akey, asec)   -->
<!-- } -->

<!-- my_config() -->
<!-- ``` -->

<!-- **twitteR** -->
<!-- First, a few examples of searches from the help docs (excluding results): -->

<!-- ```{r examples, eval = FALSE, include = FALSE} -->
<!-- ## Search between two dates -->
<!-- searchTwitter('tesla', since = '2020-12-01', until = '2020-12-24') -->

<!-- ## Geocoded results -->
<!-- searchTwitter('patriots', geocode = '42.375, -71.1061111, 10mi') -->

<!-- ## Using resultType -->
<!-- searchTwitter('from:GrittyNHL', resultType = "popular", n = 50) -->

<!-- phl_tweets <- searchTwitter('philadelphia+philly', resultType = "recent", n = 15) -->
<!-- head(phl_tweets, 1) %>% str() -->
<!-- ``` -->

<!-- Going to write a function to format the results I care about into a tidy dataframe -->

<!-- ```{r, eval = FALSE, include = FALSE} -->
<!-- getTweets <- function(){ -->
<!--   searchTwitter('philadelphia+philly', resultType = "recent", n = 15) -->
<!-- } -->
<!-- ``` -->







<!-- Now, a convenience function for accessing the text part of a tweet returned by the twitteR API (source: bryangoodrich). -->

<!-- ```{r fun1, eval = FALSE, include = FALSE} -->
<!-- tweet_text <- function(x) x$getText() -->
<!-- ``` -->

<!-- Next, a search function that uses `tweet_text()` to extract the text from each tweet (source: bryangoodrich). -->

<!-- ```{r fun2, eval = FALSE, include = FALSE} -->
<!-- tweet_corpus <- function(search, n = 5000, ...) { -->
<!--     payload <- searchTwitter(search, n = n, ...) -->
<!--     sapply(payload, tweet_text) -->
<!-- } -->
<!-- ``` -->

<!-- Finally, run the search. This may take a few minutes. -->

<!-- ```{r search, eval = FALSE, include = FALSE} -->
<!-- #tweets <- tweet_corpus("energy+electricity", n = 100, lang = 'en') -->
<!-- tweets <- tweet_corpus('philadelphia+philly', n = 1000, lang = 'en') -->
<!-- ``` -->





### Post content

Typical location to start editing since the bibliography chunk is hidden. Make sure that you selected `R Markdown (.Rmd)` as the _format_ option of the post when using the `New Post` *[blogdown](https://CRAN.R-project.org/package=blogdown)* addin.

### R image



```r
## This imaged will be saved in the /post/*_files/ directory
## Use echo = FALSE if you want to hide the code for making the plot
plot(1:10, 10:1)
```

If you prefer not to use the `fig.width` and `fig.height` options in every plot chunk, edit the YAML (the part at the top of the post) with:

```
output:
  blogdown::html_page:
    toc: no
    fig_width: 5
    fig_height: 5
```

The spaces are important.

### Custom image

The easiest option is to use the *[blogdown](https://CRAN.R-project.org/package=blogdown)* _Insert Image_ RStudio addin to add an external image described in this [blog post](http://lcolladotor.github.io/2018/03/07/blogdown-insert-image-addin). You need to use version 0.5.7 or newer to have access to this plugin. 

If you want to add images manually, check this [blog post](http://lcolladotor.github.io/2018/02/17/r-markdown-blog-template/#.WqChJZPwa50) for more details on the image syntax.


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
##  base64enc       0.1-3   2015-07-28 [1] CRAN (R 4.0.0)                         
##  BiocManager     1.30.10 2019-11-16 [1] CRAN (R 4.0.3)                         
##  BiocStyle     * 2.18.1  2020-11-24 [1] Bioconductor                           
##  blogdown        0.20    2020-06-23 [1] CRAN (R 4.0.2)                         
##  bookdown        0.21    2020-10-13 [1] CRAN (R 4.0.3)                         
##  callr           3.4.3   2020-03-28 [1] CRAN (R 4.0.2)                         
##  cli             2.2.0   2020-11-20 [1] CRAN (R 4.0.3)                         
##  colorspace      2.0-0   2020-11-11 [1] CRAN (R 4.0.3)                         
##  config        * 0.3.1   2020-12-17 [1] CRAN (R 4.0.3)                         
##  crayon          1.3.4   2017-09-16 [1] CRAN (R 4.0.2)                         
##  crosstalk       1.1.0.1 2020-03-13 [1] CRAN (R 4.0.2)                         
##  data.table      1.12.8  2019-12-09 [1] CRAN (R 4.0.0)                         
##  desc            1.2.0   2018-05-01 [1] CRAN (R 4.0.2)                         
##  devtools      * 2.3.1   2020-07-21 [1] CRAN (R 4.0.2)                         
##  digest          0.6.27  2020-10-24 [1] CRAN (R 4.0.3)                         
##  dplyr         * 1.0.2   2020-08-18 [1] CRAN (R 4.0.3)                         
##  DT              0.14    2020-06-24 [1] CRAN (R 4.0.2)                         
##  ellipsis        0.3.1   2020-05-15 [1] CRAN (R 4.0.2)                         
##  evaluate        0.14    2019-05-28 [1] CRAN (R 4.0.2)                         
##  fansi           0.4.1   2020-01-08 [1] CRAN (R 4.0.2)                         
##  farver          2.0.3   2020-01-16 [1] CRAN (R 4.0.2)                         
##  fs              1.5.0   2020-07-31 [1] CRAN (R 4.0.3)                         
##  generics        0.1.0   2020-10-31 [1] CRAN (R 4.0.3)                         
##  ggplot2       * 3.3.2   2020-06-19 [1] CRAN (R 4.0.2)                         
##  glue            1.4.2   2020-08-27 [1] CRAN (R 4.0.2)                         
##  gtable          0.3.0   2019-03-25 [1] CRAN (R 4.0.2)                         
##  hms             0.5.3   2020-01-08 [1] CRAN (R 4.0.2)                         
##  htmltools       0.5.0   2020-06-16 [1] CRAN (R 4.0.2)                         
##  htmlwidgets   * 1.5.2   2020-10-03 [1] CRAN (R 4.0.3)                         
##  httr            1.4.2   2020-07-20 [1] CRAN (R 4.0.3)                         
##  janeaustenr   * 0.1.5   2017-06-10 [1] CRAN (R 4.0.3)                         
##  jsonlite        1.7.2   2020-12-09 [1] CRAN (R 4.0.3)                         
##  knitcitations * 1.0.11  2020-12-30 [1] Github (cboettig/knitcitations@23750d5)
##  knitr           1.30    2020-09-22 [1] CRAN (R 4.0.3)                         
##  labeling        0.4.2   2020-10-20 [1] CRAN (R 4.0.3)                         
##  lattice         0.20-41 2020-04-02 [2] CRAN (R 4.0.2)                         
##  lazyeval        0.2.2   2019-03-15 [1] CRAN (R 4.0.2)                         
##  lifecycle       0.2.0   2020-03-06 [1] CRAN (R 4.0.2)                         
##  lubridate     * 1.7.9.2 2020-11-13 [1] CRAN (R 4.0.3)                         
##  magrittr        2.0.1   2020-11-17 [1] CRAN (R 4.0.3)                         
##  Matrix          1.2-18  2019-11-27 [2] CRAN (R 4.0.2)                         
##  memoise         1.1.0   2017-04-21 [1] CRAN (R 4.0.2)                         
##  modeltools      0.2-23  2020-03-05 [1] CRAN (R 4.0.3)                         
##  munsell         0.5.0   2018-06-12 [1] CRAN (R 4.0.2)                         
##  NLP           * 0.2-1   2020-10-14 [1] CRAN (R 4.0.3)                         
##  pillar          1.4.6   2020-07-10 [1] CRAN (R 4.0.3)                         
##  pkgbuild        1.0.8   2020-05-07 [1] CRAN (R 4.0.2)                         
##  pkgconfig       2.0.3   2019-09-22 [1] CRAN (R 4.0.2)                         
##  pkgload         1.1.0   2020-05-29 [1] CRAN (R 4.0.2)                         
##  plotly        * 4.9.2.1 2020-04-04 [1] CRAN (R 4.0.2)                         
##  plyr            1.8.6   2020-03-03 [1] CRAN (R 4.0.2)                         
##  prettyunits     1.1.1   2020-01-24 [1] CRAN (R 4.0.2)                         
##  processx        3.4.4   2020-09-03 [1] CRAN (R 4.0.3)                         
##  ps              1.4.0   2020-10-07 [1] CRAN (R 4.0.3)                         
##  purrr           0.3.4   2020-04-17 [1] CRAN (R 4.0.2)                         
##  R6              2.5.0   2020-10-28 [1] CRAN (R 4.0.3)                         
##  rappdirs        0.3.1   2016-03-28 [1] CRAN (R 4.0.2)                         
##  RColorBrewer  * 1.1-2   2014-12-07 [1] CRAN (R 4.0.0)                         
##  Rcpp            1.0.5   2020-07-06 [1] CRAN (R 4.0.2)                         
##  readr         * 1.4.0   2020-10-05 [1] CRAN (R 4.0.3)                         
##  RefManageR      1.3.0   2020-11-13 [1] CRAN (R 4.0.3)                         
##  remotes         2.2.0   2020-07-21 [1] CRAN (R 4.0.3)                         
##  repr            1.1.0   2020-01-28 [1] CRAN (R 4.0.3)                         
##  reshape2      * 1.4.4   2020-04-09 [1] CRAN (R 4.0.2)                         
##  rlang           0.4.9   2020-11-26 [1] CRAN (R 4.0.3)                         
##  rmarkdown     * 2.5     2020-10-21 [1] CRAN (R 4.0.3)                         
##  rprojroot       2.0.2   2020-11-15 [1] CRAN (R 4.0.2)                         
##  rtweet        * 0.7.0   2020-01-08 [1] CRAN (R 4.0.3)                         
##  scales          1.1.1   2020-05-11 [1] CRAN (R 4.0.2)                         
##  sessioninfo     1.1.1   2018-11-05 [1] CRAN (R 4.0.2)                         
##  skimr         * 2.1.2   2020-07-06 [1] CRAN (R 4.0.3)                         
##  slam            0.1-48  2020-12-03 [1] CRAN (R 4.0.3)                         
##  SnowballC     * 0.7.0   2020-04-01 [1] CRAN (R 4.0.3)                         
##  stringi         1.5.3   2020-09-09 [1] CRAN (R 4.0.3)                         
##  stringr       * 1.4.0   2019-02-10 [1] CRAN (R 4.0.2)                         
##  testthat        2.3.2   2020-03-02 [1] CRAN (R 4.0.2)                         
##  textdata        0.4.1   2020-05-04 [1] CRAN (R 4.0.3)                         
##  tibble          3.0.4   2020-10-12 [1] CRAN (R 4.0.3)                         
##  tidyr         * 1.1.2   2020-08-27 [1] CRAN (R 4.0.3)                         
##  tidyselect      1.1.0   2020-05-11 [1] CRAN (R 4.0.2)                         
##  tidytext      * 0.2.6   2020-09-20 [1] CRAN (R 4.0.3)                         
##  tm            * 0.7-8   2020-11-18 [1] CRAN (R 4.0.3)                         
##  tokenizers      0.2.1   2018-03-29 [1] CRAN (R 4.0.3)                         
##  topicmodels   * 0.2-11  2020-04-19 [1] CRAN (R 4.0.3)                         
##  usethis       * 2.0.0   2020-12-10 [1] CRAN (R 4.0.3)                         
##  utf8            1.1.4   2018-05-24 [1] CRAN (R 4.0.2)                         
##  vctrs           0.3.4   2020-08-29 [1] CRAN (R 4.0.3)                         
##  viridisLite     0.3.0   2018-02-01 [1] CRAN (R 4.0.2)                         
##  widgetframe   * 0.3.1   2017-12-20 [1] CRAN (R 4.0.3)                         
##  withr           2.3.0   2020-09-22 [1] CRAN (R 4.0.3)                         
##  wordcloud     * 2.6     2018-08-24 [1] CRAN (R 4.0.3)                         
##  xfun            0.19    2020-10-30 [1] CRAN (R 4.0.3)                         
##  xml2            1.3.2   2020-04-23 [1] CRAN (R 4.0.2)                         
##  yaml            2.2.1   2020-02-01 [1] CRAN (R 4.0.0)                         
## 
## [1] C:/Users/rober/Documents/R/win-library/4.0
## [2] C:/Program Files/R/R-4.0.2/library
```
