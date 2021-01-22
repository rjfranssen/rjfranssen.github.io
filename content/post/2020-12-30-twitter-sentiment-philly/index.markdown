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







2021-01-21 | _26 min_


This is a walkthrough of using the Twitter API with the `rtweet` R package. It borrows from a few difference sources, including:
* [ropensci/rtweet documentation](https://github.com/ropensci/rtweet)  
* [here](https://gist.github.com/bryangoodrich/7b5ef683ce8db592669e)  
* [Tidy Text Mining with R](https://www.tidytextmining.com/topicmodeling.html)  
* [Twitter Data in R Using Rtweet: Analyze and Download Twitter Data](https://www.earthdatascience.org/courses/earth-analytics/get-data-using-apis/use-twitter-api-r/) 
* [Comparing Twitter archives](https://www.tidytextmining.com/twitter.html)  
* [Mining NASA metadata](https://www.tidytextmining.com/nasa.html)  
* [Analyzing usenet text](https://www.tidytextmining.com/usenet.html)  

### Load libraries



### Configure Twitter API access

There are a couple different ways to authenticate your session. The `rtweet` package makes this pretty easy by enabling browser-based authentication. I prefer to authenticate in code. Check the `rtweet` vignette for connection options.  

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

```


Optionally, save this df to a local directory so you come back to this later without having to hit the Twitter API; `Rds` and `Rdata` files load super fast. You can also add these file types to your `.gitignore` and `config.toml` if you don't want to push them to your online git repo.


```r
saveRDS(tweets_df, file = "tweets_df.Rds", compress = 'xz')
```

Reading back in the `.Rds` file..


```r
tweets_df <- readRDS("tweets_df.Rds")

tweets_df %>% head()
## # A tibble: 6 x 97
##   user_id status_id created_at          screen_name text  source
##   <chr>   <chr>     <dttm>              <chr>       <chr> <chr> 
## 1 157556~ 13520120~ 2021-01-20 21:56:15 NBCPhilade~ Wher~ Socia~
## 2 157556~ 13512856~ 2021-01-18 21:49:44 NBCPhilade~ Pres~ Socia~
## 3 157556~ 13501436~ 2021-01-15 18:11:35 NBCPhilade~ JUST~ Socia~
## 4 157556~ 13516454~ 2021-01-19 21:39:16 NBCPhilade~ UPDA~ Socia~
## 5 157556~ 13499459~ 2021-01-15 05:06:07 NBCPhilade~ Phil~ Socia~
## 6 157556~ 13498216~ 2021-01-14 20:52:18 NBCPhilade~ The ~ Socia~
## # ... with 91 more variables: display_text_width <dbl>,
## #   reply_to_status_id <chr>, reply_to_user_id <chr>,
## #   reply_to_screen_name <chr>, is_quote <lgl>, is_retweet <lgl>,
## #   favorite_count <int>, retweet_count <int>, quote_count <int>,
## #   reply_count <int>, hashtags <list>, symbols <list>, urls_url <list>,
## #   urls_t.co <list>, urls_expanded_url <list>, media_url <list>,
## #   media_t.co <list>, media_expanded_url <list>, media_type <list>,
## #   ext_media_url <list>, ext_media_t.co <list>, ext_media_expanded_url <list>,
## #   ext_media_type <chr>, mentions_user_id <list>, mentions_screen_name <list>,
## #   lang <chr>, quoted_status_id <chr>, quoted_text <chr>,
## #   quoted_created_at <dttm>, quoted_source <chr>, quoted_favorite_count <int>,
## #   quoted_retweet_count <int>, quoted_user_id <chr>, quoted_screen_name <chr>,
## #   quoted_name <chr>, quoted_followers_count <int>,
## #   quoted_friends_count <int>, quoted_statuses_count <int>,
## #   quoted_location <chr>, quoted_description <chr>, quoted_verified <lgl>,
## #   retweet_status_id <chr>, retweet_text <chr>, retweet_created_at <dttm>,
## #   retweet_source <chr>, retweet_favorite_count <int>,
## #   retweet_retweet_count <int>, retweet_user_id <chr>,
## #   retweet_screen_name <chr>, retweet_name <chr>,
## #   retweet_followers_count <int>, retweet_friends_count <int>,
## #   retweet_statuses_count <int>, retweet_location <chr>,
## #   retweet_description <chr>, retweet_verified <lgl>, place_url <chr>,
## #   place_name <chr>, place_full_name <chr>, place_type <chr>, country <chr>,
## #   country_code <chr>, geo_coords <list>, coords_coords <list>,
## #   bbox_coords <list>, status_url <chr>, name <chr>, location <chr>,
## #   description <chr>, url <chr>, protected <lgl>, followers_count <int>,
## #   friends_count <int>, listed_count <int>, statuses_count <int>,
## #   favourites_count <int>, account_created_at <dttm>, verified <lgl>,
## #   profile_url <chr>, profile_expanded_url <chr>, account_lang <lgl>,
## #   profile_banner_url <chr>, profile_background_url <chr>,
## #   profile_image_url <chr>, lat <dbl>, lng <dbl>, tweet_datetime <dttm>,
## #   tweet_date <date>, tweet_year <dbl>, tweet_month <dbl>, tweet_day <int>
```


### Examining data

Now going to identify a few basic things, but first, this df has 92 columns. What are some of the columns that we're most likely to use? The `skimr` package is **really** good at doing this and I often use it in addition to `summary()`.


```r
#summary(tweet_df)
skim(tweets_df)
```


And what does the table look like?



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
  head(2)
## # A tibble: 2 x 18
##   text  lang  hashtags symbols favorite_count retweet_count   lat   lng name 
##   <chr> <chr> <list>   <list>           <int>         <int> <dbl> <dbl> <chr>
## 1 Wher~ en    <chr [1~ <chr [~              0             0    NA    NA NBC1~
## 2 Pres~ en    <chr [1~ <chr [~            100             6    NA    NA NBC1~
## # ... with 9 more variables: screen_name <chr>, followers_count <int>,
## #   friends_count <int>, location <chr>, description <chr>, listed_count <int>,
## #   statuses_count <int>, created_at <dttm>, account_created_at <dttm>
```

### Sample stats & visualizations  

What is our total tweet *count*?   


```r
nrow(tweets_df) %>% print()
## [1] 1981
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
##   screen_name  text                           favorite_count created_at         
##   <chr>        <chr>                                   <int> <dttm>             
## 1 ScottPresler "Unarmed man murdered while w~           2899 2021-01-18 17:35:01
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

<img src="{{< blogdown/postref >}}index_files/figure-html/statsthree-1.png" width="672" />


In what *languages* are they tweeting?  

Languages are recorded here as [alpha-3 / ISO 639.2 codes](https://www.loc.gov/standards/iso639-2/php/code_list.php).  


```r
tweets_df %>%
  group_by(lang) %>%
  summarize(num_tweets = n()) %>%
  arrange(-num_tweets) %>% # can also use slice_max()
  head(5)
## # A tibble: 5 x 2
##   lang  num_tweets
##   <chr>      <int>
## 1 en          1907
## 2 und           44
## 3 cy             9
## 4 da             3
## 5 et             3

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
  scale_y_continuous(breaks = seq(0, 450, by=50), limits = c(0, 450), expand = c(0, 0)) + # use expand() to remove empty space between axis and bars
  geom_hline(yintercept = seq(0, 450, by = 25), color = 'white')
```

<img src="{{< blogdown/postref >}}index_files/figure-html/statsfive-1.png" width="672" />


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
  labs(title = "Daily Tweets for 'Philadelphia' (ggplot2 + plotly)") +
  xlab("") +
  ylab("Number of Tweets") 

pltly1 <- plotly::ggplotly(plt1)
```


_Note: Because I'm writing this in .Rmarkdown, I use the below snippet to save the html widget to file and then bring it back in via an iframe. The main reason is that `.Rmarkdown` files are processed by `Pandoc`, output as `.markdown` and then processed by Hugo/Goldmark before becoming `html` files. The `.Rmd` files are processed by `Pandoc` and output as `html`; so, while the `.Rmd` is widget-friendly, I was losing the Hugo/Goldmark styling associated with my Hugo theme (e.g. TOCs, code highlighting). This is why all my posts that involve code are written in `.Rmarkdown`. See the [blogdown docs](https://bookdown.org/yihui/blogdown/output-format.html) for a better explanation._

**Update!** Exporting and bringing an html file via an iframe is no longer necessary with `blogdown 1.0`! I'm going to see the code as a reference if I need to do it again to avoid css conflicts, but giant thanks to the blogdown team for making this workaround obsolete. See the [release notes including markdown comparisons here](https://blog.rstudio.com/2021/01/18/blogdown-v1.0/).



```r
widget <- pltly1
widgetfile <- 'pltly1.html'

htmlwidgets::saveWidget(widget = widget
                        , selfcontained = TRUE
                        , file = widgetfile)

cat(paste0('<iframe src= "', widgetfile, '" width="100%" height="400" style="border: none;"></iframe>'))
```

<iframe src= "pltly1.html" width="100%" height="400" style="border: none;"></iframe>


```r
# Using dygraph
# convert series to an xts object 
tweets_ts <- xts::xts(tweets_per_day$num_tweets, order.by = tweets_per_day$tweet_date)
# Create the dygraph
library(dygraphs)

dygraph1 <- dygraph(tweets_ts, main = "Daily Tweets for 'Philadelphia' (dygraph)") %>%
  dySeries("V1", label = "Tweet Count")
```

<iframe src= "dygraph1.html" width="100%" height="400" style="border: none;"></iframe>


### Tweet Map

And *where* are people tweeting?

Of the 1981 tweets that we have, 111 are geotagged. We'll plot those using [leaflet](https://rstudio.github.io/leaflet/). I'm assigning the tweet `text` to each point's `label` and `popup` arguments.  


```r
library(leaflet)

tweet_map <- leaflet(tweets_df) %>%
  addTiles() %>%
  addMarkers(lng = ~lng
                   , lat = ~lat
                   , label = ~as.character(text)
                   , popup = ~as.character(text)
                   )
```

<iframe src= "tweet_map.html" width="100%" height="400" style="border: none;"></iframe>


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

tidy_books %>% head() %>% knitr::kable()
```



|book                | linenumber| chapter|word        |
|:-------------------|----------:|-------:|:-----------|
|Sense & Sensibility |          1|       0|sense       |
|Sense & Sensibility |          1|       0|and         |
|Sense & Sensibility |          1|       0|sensibility |
|Sense & Sensibility |          3|       0|by          |
|Sense & Sensibility |          3|       0|jane        |
|Sense & Sensibility |          3|       0|austen      |


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

tidy_books %>% head() %>% knitr::kable()
```



|book         |chapter    | linenumber|word  |
|:------------|:----------|----------:|:-----|
|Philadelphia |2021-01-20 |          1|where |
|Philadelphia |2021-01-20 |          1|was   |
|Philadelphia |2021-01-20 |          1|this  |
|Philadelphia |2021-01-20 |          1|when  |
|Philadelphia |2021-01-20 |          1|we    |
|Philadelphia |2021-01-20 |          1|were  |


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

_Note: When I last ran this code, the second word listed under "joy" was "white". I don't believe that the word "white" is synonymous with "joy" or should be interpreted as joyous, but I left it here to demonstrate an assumption that is built into this particular lexicon, and that we must rely on context to judge the sentiment of a word and to beware of these occurrences in our analyses. I plan to append this post with methods to evaluate words based on context._


```r
tidy_books %>%
  filter(book == "Philadelphia") %>%
  inner_join(nrc_joy) %>%
  count(word, sort = TRUE) %>%
  head(10)
## # A tibble: 10 x 2
##    word             n
##    <chr>        <int>
##  1 good            47
##  2 white           46
##  3 food            41
##  4 love            30
##  5 music           25
##  6 inauguration    24
##  7 vote            21
##  8 birthday        19
##  9 resources       19
## 10 grant           18
```

Using `tidyr`, we can continue to examine how sentiment changes throughout the book. This is more relevant to the novels examples from the docs, but I'm including it here as an exercise. This example will use the `bing` lexicon. We need some type of interval - the example creates an index from 80 line chunks, but I'm going to update this to use the `chapter` value, which now represents the date of each tweet allowing to see changes in sentiment over time.

From the docs:  

> ... First, we find a sentiment score for each word using the Bing lexicon and inner_join(). 
> Next, we count up how many positive and negative words there are in defined sections of each book. We define an index here to keep track of where we are in the narrative; this index (using integer division) counts up sections of 80 lines of text.  
> The %/% operator does integer division (x %/% y is equivalent to floor(x/y)) so the index keeps track of which 80-line section of text we are counting up negative and positive sentiment in.  
> Small sections of text may not have enough words in them to get a good estimate of sentiment while really large sections can wash out narrative structure. For these books, using 80 lines works well, but this can vary depending on individual texts, how long the lines were to start with, etc. We then use spread() so that we have negative and positive sentiment in separate columns, and lastly calculate a net sentiment (positive - negative).  


```r
philly_sentiment <- tidy_books %>%
  inner_join(get_sentiments("bing"), by = c("word" = "word")) %>%
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

<img src="{{< blogdown/postref >}}index_files/figure-html/sentimentsplotone-1.png" width="672" />




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
  inner_join(get_sentiments("afinn"), by = c("word" = "word")) %>% 
  #group_by(index = linenumber %/% 80) %>% # Can use 80-line chunks
  group_by(index = chapter) %>% # or other type of index value, like date
  summarise(sentiment = sum(value)) %>% 
  mutate(method = "AFINN") %>%
  ungroup()

## rbinding bing and nrc
bing_and_nrc <- bind_rows(
  philly_tweets %>% 
    inner_join(get_sentiments("bing"), by = c("word" = "word")) %>%
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

<img src="{{< blogdown/postref >}}index_files/figure-html/sentimentplottwo-1.png" width="672" />



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

bing_word_counts %>% head(10) %>% knitr::kable()
```



|word       |sentiment |  n|
|:----------|:---------|--:|
|like       |positive  | 73|
|corruption |negative  | 68|
|best       |positive  | 61|
|good       |positive  | 47|
|well       |positive  | 45|
|free       |positive  | 44|
|killed     |negative  | 42|
|right      |positive  | 37|
|great      |positive  | 36|
|ready      |positive  | 32|


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

<img src="{{< blogdown/postref >}}index_files/figure-html/sentplotfour-1.png" width="672" />



### Wordclouds  

I don't find wordclouds terrible useful, but they're not terrible for a background graphic or an attention grabber.

First, let's add some custom stop words (i.e. words to exclude from the analysis). If you want to know why, run the next chunk and see how URLs (and obviously our original search terms) throw off our term frequency matrix.   


```r
custom_stop_words <- bind_rows(tibble(word = c('philadelphia', 'philly', 't.co', 'https', 'amp'),
                                      lexicon = c("custom")),
                               stop_words)

custom_stop_words %>% head() %>% knitr::kable()
```



|word         |lexicon |
|:------------|:-------|
|philadelphia |custom  |
|philly       |custom  |
|t.co         |custom  |
|https        |custom  |
|amp          |custom  |
|a            |SMART   |



```r
library(wordcloud)

tidy_books %>%
  anti_join(custom_stop_words) %>%
  count(word) %>%
  with(wordcloud(word, n, max.words = 100, col = brewer.pal(8, "Dark2")))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/wordcloudone-1.png" width="672" />

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

<img src="{{< blogdown/postref >}}index_files/figure-html/wordcloudtwo-1.png" width="672" />


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
## 1 Philadelphia 2021-01-13           106  2478         0.0428
```






### Acknowledgements


This blog post was made possible thanks to:

* *[knitr](https://bioconductor.org/packages/3.12/knitr)* 
* *[rtweet](https://bioconductor.org/packages/3.12/rtweet)* <a id='cite-rtweet-package'></a>(<a href='https://joss.theoj.org/papers/10.21105/joss.01829'>Kearney, 2019</a>)
* *[rmarkdown](https://bioconductor.org/packages/3.12/rmarkdown)* 
* *[NLP](https://bioconductor.org/packages/3.12/NLP)* <a id='cite-Hornik_2020'></a>(<a href='https://CRAN.R-project.org/package=NLP'>Hornik, 2020</a>)
* *[tm](https://bioconductor.org/packages/3.12/tm)* 
* *[RColorBrewer](https://bioconductor.org/packages/3.12/RColorBrewer)* <a id='cite-Neuwirth_2014'></a>(<a href='https://CRAN.R-project.org/package=RColorBrewer'>Neuwirth, 2014</a>)
* *[wordcloud](https://bioconductor.org/packages/3.12/wordcloud)* <a id='cite-Fellows_2018'></a>(<a href='https://CRAN.R-project.org/package=wordcloud'>Fellows, 2018</a>)
* *[topicmodels](https://bioconductor.org/packages/3.12/topicmodels)* <a id='cite-Grun_2011'></a>(<a href='https://doi.org/10.18637/jss.v040.i13'>Grün and Hornik, 2011</a>)
* *[SnowballC](https://bioconductor.org/packages/3.12/SnowballC)* <a id='cite-Bouchet-Valat_2020'></a>(<a href='https://CRAN.R-project.org/package=SnowballC'>Bouchet-Valat, 2020</a>)
* *[config](https://bioconductor.org/packages/3.12/config)* <a id='cite-Allaire_2020'></a>(<a href='https://CRAN.R-project.org/package=config'>Allaire, 2020</a>)
* *[dplyr](https://bioconductor.org/packages/3.12/dplyr)* <a id='cite-Wickham_2020'></a>(<a href='https://CRAN.R-project.org/package=dplyr'>Wickham, François, 
             Henry, and Müller, 2020</a>)
* *[tidytext](https://bioconductor.org/packages/3.12/tidytext)* <a id='cite-Silge_2016'></a>(<a href='http://dx.doi.org/10.21105/joss.00037'>Silge and Robinson, 2016</a>)
* *[tidyr](https://bioconductor.org/packages/3.12/tidyr)* <a id='cite-Wickham_2020a'></a>(<a href='https://CRAN.R-project.org/package=tidyr'>Wickham, 2020</a>)
* *[ggplot2](https://bioconductor.org/packages/3.12/ggplot2)* <a id='cite-Wickham_2016'></a>(<a href='https://ggplot2.tidyverse.org'>Wickham, 2016</a>)
* *[lubridate](https://bioconductor.org/packages/3.12/lubridate)* <a id='cite-Grolemund_2011'></a>(<a href='https://www.jstatsoft.org/v40/i03/'>Grolemund and Wickham, 2011</a>)
* *[readr](https://bioconductor.org/packages/3.12/readr)* <a id='cite-Wickham_2020ab'></a>(<a href='https://CRAN.R-project.org/package=readr'>Wickham and Hester, 2020</a>)
* *[skimr](https://bioconductor.org/packages/3.12/skimr)* <a id='cite-Waring_2020'></a>(<a href='https://CRAN.R-project.org/package=skimr'>Waring, Quinn, McNamara, Arino de la Rubia, et al., 2020</a>)
* *[BiocStyle](https://bioconductor.org/packages/3.12/BiocStyle)* 
* *[blogdown](https://CRAN.R-project.org/package=blogdown)* <a id='cite-Xie_2017'></a>(<a href='https://github.com/rstudio/blogdown'>Xie, Hill, and Thomas, 2017</a>)
* *[devtools](https://CRAN.R-project.org/package=devtools)* <a id='cite-Wickham_2020abc'></a>(<a href='https://CRAN.R-project.org/package=devtools'>Wickham, Hester, and Chang, 2020</a>)
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
