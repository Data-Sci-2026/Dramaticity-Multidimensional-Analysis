# Scraps from the Loft Pipeline
Zach Shultz
2026-10-25

## Prereqs

Note that the polite package was not available from CRAN when this
project was being created. If that is still the case, an older version
of the package can still be downloaded to utilize the polite features.

``` r
library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.2.1     ✔ readr     2.2.0
    ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ✔ ggplot2   4.0.3     ✔ tibble    3.3.1
    ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
    ✔ purrr     1.2.2     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(rvest)
```

    Attaching package: 'rvest'

    The following object is masked from 'package:readr':

        guess_encoding

``` r
library(polite)
```

## Intro

This document is a place for the pipeline to create the Scraps from the
Loft data frame.

## Reading in the Data

The code below shows how to do this with html filling in for a link to
the page with the transcript. For the purposes of this document, we can
look at the test page I used, the Adults Episode 4 transcript. Note that
the polite package has not been incorporated yed since this is only one
file. For loading multiple files, be sure to be polite (shown below)!

``` r
html <- "https://scrapsfromtheloft.com/tv-series/adults-s01e04-house-rules-transcript/"

#reads in the website
show <- read_html(html)
```

## Finding the right Elements and Transformation

Using selector gadget, I was able to find the correct elements I wanted
to target fairly quickly as shown in the code below.

``` r
show_script <- show |>
  #find all elements on the web page that are actually dialogue
  html_elements(".single-content p") |>
  #turn those elements into only text
  html_text2()

#turn our character vector into a df
show_df <- data.frame(show_script) 
```

I also transformed this into a data frame. While this probably wasn’t
strictly necessary, it made it easier to borrow code from the [Santa
Barbara Pipeline](santa_barbara_pipeline.qmd) which saved time and
energy. Hopefully, it also makes these twin pipelines easier to follow.

## Cleaning the Data

The next step was to eliminate the first four lines. Luckily, in every
transcript the first four lines are dedicated to the same structure,
making it easy to leave only the text we want.

``` r
show_df <- show_df |>
  #removes the first four rows which are not actually dialogue from the show
  slice(-(1:4))
```

``` r
#defines a function for cleaning each string in our df
  clean <- function(string) {
 string |>
  #remove a [ followed by any number of any characters and then a ]
  str_replace_all("\\[.*?\\]", "") |>
  #removes a ♪ followed by any number of characters followed by ♪
  str_replace_all("♪[^♪]*♪", "") |>
  #removes a ( followed by any number of characters followed by )
  str_replace_all("\\([^)]*\\)", "") |>
  #removes all remaining music notes
  str_remove_all("♪") |>
  #removes all *** that might've snuck in
  str_remove_all("\\*") |>
  #removes capital letter words (with periods and apostrophe's inside for words like R.J. and BELINDA'S) that begin a line and are followed by a colon!
  str_replace("^[A-Z\\.’]+( [A-Z\\.]+)? ?[1-9]?:", "")
}
```

## The Final Function and Iteration

The polite function has been incorporated in the code below.

``` r
get_clean_script <- function(url) {
  
  #reads in the website POLITELY
  #let's the website know we're taking
  session <- bow(url)
  #takes slowly
  html <- polite::scrape(session)

            #removes the first part of the html link which is identical across transcripts
  series <- str_remove(url, "https://scrapsfromtheloft.com/tv-series/") |>
    #replaces -s followed by a digit followed by any number of characters and then a - with a single -
    str_replace("-s\\d.+-", "-") |>
    #removes remaining /
    str_remove("/") |>
    #replaces all -'s with _'s
    str_replace_all("-", "_")
  
  show_script <- html |>
  #find all elements on the web page that are actually dialogue
  html_elements(".single-content p") |>
  #turn those elements into only text
  html_text2()
  
  #turn our character vector into a df
  show_df <- data.frame(show_script) 
  
  show_df <- show_df |>
  #removes the first four rows which are not actually dialogue from the show
  slice(-(1:4))
  
  #defines a function for cleaning each string in our df
  clean <- function(string) {
 string |>
  #remove a [ followed by any number of any characters and then a ]
  str_replace_all("\\[.*?\\]", "") |>
  #removes a ♪ followed by any number of characters followed by ♪
  str_replace_all("♪[^♪]*♪", "") |>
  #removes a ( followed by any number of characters followed by )
  str_replace_all("\\([^)]*\\)", "") |>
  #removes all remaining music notes
  str_remove_all("♪") |>
  #removes all *** that might've snuck in
  str_remove_all("\\*") |>
  #removes capital letter words (with periods and apostrophe's inside for words like R.J. and BELINDA'S) that begin a line and are followed by a colon!
  str_replace("^[A-Z\\.’]+( [A-Z\\.]+)? ?[1-9]?:", "")
}
  
  show_df <- show_df |>
    mutate(show_script = 
             #cleans up all the lines with the function above
             clean(show_script)) |>
    mutate(show_script = 
             #removes extra white space
             trimws(show_script)) |>
    #first attempt to remove the extra line!
    mutate(show_script = replace_when(show_script, str_detect(show_script, "S\\d\\dE\\d\\d") ~ "")) |>
    #removes lines with no text
    filter(!show_script == '') 

  show_df <- show_df |>
  #returns a single cell data frame of all of the dialogue combined with spaces
  summarize(show_script = str_flatten(show_script, " "))
  
  show_df <- show_df |>
  mutate(id = series,
         #adds a new column that gives the id based on the series
         .before = "show_script") |>
  #renames to clean_words so we can rbind this thing with the santa barbara df eventually making one big df!!!
  rename("clean_words" = show_script)

show_df
}
```

## Deciding on TV shows and Iteration

To compile a list of TV shows I would want to include, I compiled a list
of the available TV shows that would be relevant to the study. Their
relevance was determined using criteria in the [Project Plan
document](project_plan.md). I also included the genre of each show
(derived from the Wikipedia site of the TV show) which allowed me to
ensure a nearly even number of drama TV shows and comedy TV shows. Since
there were only 27 comedy TV shows that met the criteria, I included all
of them. I then randomly selected 33 of the remaining drama TV shows
with the following code.

Note that of the tv shows I selected, 3 were later found to not meet the
criteria (or to have formatting issued in the transcript) so these were
replaced with 3 different drama tv shows. One of these was a comedy,
which brought the total down to 27.

``` r
#just chose a random seed so that we get the same random result every time we run this
set.seed(20)

dramas <- tv_show_df |>
  #filters only shows that included the genre drama and not the genre comedy
  filter(str_detect(genre, "drama") & !str_detect(genre, "comedy")) |>
  #randomly selects 32 of these
  slice_sample(n = 32)

dramas
```

However, the above code is just to explain the process. Some TV shows
had to be replaced, so for an exact reference for the list used with
genres included, the `genre_df` in the `data` folder can be used.

``` r
genre_df <- read_csv("data/genre_reference_df.csv")
```

    Rows: 60 Columns: 3
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (2): shows, genre
    dbl (1): index

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
genre_df
```

    # A tibble: 60 × 3
       shows              genre index
       <chr>              <chr> <dbl>
     1 the lincoln lawyer drama     1
     2 reasonable doubt   drama     2
     3 9-1-1              drama     3
     4 grey's anatomy     drama     4
     5 reacher            drama     5
     6 marshalls          drama     6
     7 the madison        drama     7
     8 sherrif county     drama     8
     9 ransom canyon      drama     9
    10 breaking bad       drama    10
    # ℹ 50 more rows

The next important step was the compilation of the html links to feed
through this function. Unfortunately, since each link referred to a
specific episode and there was no clear way to know which episodes were
included, this process had to be done manually. This list can also be
found in the `data` folder.

First, we will need to read in the full vector of episode html links:

``` r
episode_links = readLines("data/html")
```

    Warning in readLines("data/html"): incomplete final line found on 'data/html'

``` r
episode_links
```

     [1] "https://scrapsfromtheloft.com/tv-series/lincoln-lawyer-s04e03-forty-hours-transcript/"                                        
     [2] "https://scrapsfromtheloft.com/tv-series/reasonable-doubt-s04e03-aint-no-love-in-the-heart-of-the-city-transcript/"            
     [3] "https://scrapsfromtheloft.com/tv-series/9-1-1-s10e02-fire-in-the-hole-transcript/"                                            
     [4] "https://scrapsfromtheloft.com/tv-series/greys-anatomy-s17e03-my-happy-ending-transcript/"                                     
     [5] "https://scrapsfromtheloft.com/tv-series/reacher-s01e03-spoonful-transcript/"                                                  
     [6] "https://scrapsfromtheloft.com/tv-series/marshals-s02e03-falling-sky-transcript/"                                              
     [7] "https://scrapsfromtheloft.com/tv-series/the-madison-s01e03-watch-her-fall-transcript/"                                        
     [8] "https://scrapsfromtheloft.com/tv-series/sheriff-country-s02e03-ulysses-transcript/"                                           
     [9] "https://scrapsfromtheloft.com/tv-series/ransom-canyon-s02e03-that-dog-wont-hunt-transcript/"                                  
    [10] "https://scrapsfromtheloft.com/tv-series/breaking-bad-s01e03-and-the-bags-in-the-river-transcript/"                            
    [11] "https://scrapsfromtheloft.com/tv-series/the-white-lotus-s01e03-mysterious-monkeys-transcript/"                                
    [12] "https://scrapsfromtheloft.com/tv-series/ozark-s04e03-city-on-the-make-transcript/"                                            
    [13] "https://scrapsfromtheloft.com/tv-series/the-pitt-s01e03-900-am-transcript/"                                                   
    [14] "https://scrapsfromtheloft.com/tv-series/bad-monkey-s01e03-nobody-said-he-was-alvin-einstein-transcript/"                      
    [15] "https://scrapsfromtheloft.com/tv-series/lucifer-s05e10-bloody-celestial-karaoke-jam-transcript/"                              
    [16] "https://scrapsfromtheloft.com/tv-series/dexter-original-sin-s01e06-the-joy-of-killing-transcript/"                            
    [17] "https://scrapsfromtheloft.com/tv-series/percy-jackson-and-the-olympians-s01e03-we-visit-the-garden-gnome-emporium-transcript/"
    [18] "https://scrapsfromtheloft.com/tv-series/the-last-of-us-s01e03-long-long-time-transcript/"                                     
    [19] "https://scrapsfromtheloft.com/tv-series/beauty-in-black-s03e03-throne-of-blood-transcript/"                                   
    [20] "https://scrapsfromtheloft.com/tv-series/untamed-s01e03-el-o-win-transcript/"                                                  
    [21] "https://scrapsfromtheloft.com/tv-series/seal-team-s01e03-boarding-party-transcript/"                                          
    [22] "https://scrapsfromtheloft.com/tv-series/evil-s04e03-how-to-slaughter-a-pig-transcript/"                                       
    [23] "https://scrapsfromtheloft.com/tv-series/tulsa-king-s03e03-the-g-and-the-og-transcript/"                                       
    [24] "https://scrapsfromtheloft.com/tv-series/anna-pigeon-s01e03-the-preacher-transcript/"                                          
    [25] "https://scrapsfromtheloft.com/tv-series/visionquest-s01e03-reboot-transcript/"                                                
    [26] "https://scrapsfromtheloft.com/tv-series/outer-banks-s05e03-square-grouper-transcript/"                                        
    [27] "https://scrapsfromtheloft.com/tv-series/the-bear-s01e03-brigade-transcript/"                                                  
    [28] "https://scrapsfromtheloft.com/tv-series/the-boys-s03e07-here-comes-a-candle-to-light-you-to-bed-transcript/"                  
    [29] "https://scrapsfromtheloft.com/tv-series/death-and-other-details-s01e03-troublesome-transcript/"                               
    [30] "https://scrapsfromtheloft.com/tv-series/the-rainmaker-s02e03-chokepoint-transcript/"                                          
    [31] "https://scrapsfromtheloft.com/tv-series/the-perfect-couple-s01e03-the-perfect-family-transcript/"                             
    [32] "https://scrapsfromtheloft.com/tv-series/abbott-elementary-s06e03-tryouts-transcript/"                                         
    [33] "https://scrapsfromtheloft.com/tv-series/adults-s01e02-spitroast-transcript/"                                                  
    [34] "https://scrapsfromtheloft.com/tv-series/sugar-s01e03-shibuya-crossing-transcript/"                                            
    [35] "https://scrapsfromtheloft.com/tv-series/best-medicine-2026-s02e03-crate-expectations-transcript/"                             
    [36] "https://scrapsfromtheloft.com/tv-series/boots-s01e03-the-confidence-course-transcript/"                                       
    [37] "https://scrapsfromtheloft.com/tv-series/brothers-s01e03-little-woody-transcript/"                                             
    [38] "https://scrapsfromtheloft.com/tv-series/chad-powers-s01e03-3rd-quarter-transcript/"                                           
    [39] "https://scrapsfromtheloft.com/tv-series/the-chair-s01e03-town-hall-transcript/"                                               
    [40] "https://scrapsfromtheloft.com/tv-series/the-chair-company-s01e03-brownderbyhistoricvids-transcript/"                          
    [41] "https://scrapsfromtheloft.com/tv-series/curb-your-enthusiasm-s12e03-vertical-drop-horizontal-tug-transcript/"                 
    [42] "https://scrapsfromtheloft.com/tv-series/elle-s01e03-youre-not-the-girl-i-thought-you-were-transcript/"                        
    [43] "https://scrapsfromtheloft.com/tv-series/elsbeth-s04e03-consciously-unsistered-transcript/"                                    
    [44] "https://scrapsfromtheloft.com/tv-series/georgie-mandys-first-marriage-s02e03-will-and-dead-mans-wife-transcript/"             
    [45] "https://scrapsfromtheloft.com/tv-series/eternally-yours-s01e03-the-mistake-transcript/"                                       
    [46] "https://scrapsfromtheloft.com/tv-series/hacks-s01e03-a-gig-is-a-gig-transcript/"                                              
    [47] "https://scrapsfromtheloft.com/tv-series/the-hawk-s01e03-roses-thorns-transcript/"                                             
    [48] "https://scrapsfromtheloft.com/tv-series/i-love-la-s01e03-girls-girl-transcript/"                                              
    [49] "https://scrapsfromtheloft.com/tv-series/its-always-sunny-in-philadelphia-s18e03-the-gang-gets-tested-transcript/"             
    [50] "https://scrapsfromtheloft.com/tv-series/leanne-s01e03-oh-knoxville-transcript/"                                               
    [51] "https://scrapsfromtheloft.com/tv-series/a-man-on-the-inside-s02e03-family-weekend-transcript/"                                
    [52] "https://scrapsfromtheloft.com/tv-series/modern-love-s02e03-strangers-on-a-dublin-train-transcript/"                           
    [53] "https://scrapsfromtheloft.com/tv-series/this-is-us-s05e12-both-things-can-be-true-transcript/"                                
    [54] "https://scrapsfromtheloft.com/tv-series/nobody-wants-this-s02e03-the-unethical-therapist-transcript/"                         
    [55] "https://scrapsfromtheloft.com/tv-series/peacemaker-s02e03-another-rick-up-my-sleeve-transcript/"                              
    [56] "https://scrapsfromtheloft.com/tv-series/scrubs-2026-s02e03-my-exs-ex-transcript/"                                             
    [57] "https://scrapsfromtheloft.com/tv-series/shameless-s11e01-this-is-chicago-transcript/"                                         
    [58] "https://scrapsfromtheloft.com/tv-series/tires-s01e03-sales-contest-transcript/"                                               
    [59] "https://scrapsfromtheloft.com/tv-series/ted-s01-e03-ejectile-dysfunction-transcript/"                                         
    [60] "https://scrapsfromtheloft.com/tv-series/young-sheldon-s07e03-a-strudel-and-a-hot-american-boy-toy-transcript/"                

Once this list of links was compiled, I was able to iterate using the
following mapping code:

``` r
x <- episode_links |>
  map(get_clean_script) 

y <- do.call(rbind, x)

y <- y |>
    #adding an index column to the df
    mutate(index = row_number()) |>
    #combining show_df to the final table so that we have access to the genre
    left_join(genre_df) |>
    #get rid of shows and the index as we don't need them anymore
    select(-c(shows, index)) |>
    #replace transcript with wahtever the genre is
    mutate(id = str_replace(id, "transcript", genre))
```

    Joining with `by = join_by(index)`

``` r
y <- y |>
  #get rid of genre
  select(-genre)

tv_show_transcripts <- y
```

This code creates a private folder if there is no such folder already.

``` r
if(!dir.exists("private")) 
  dir.create("private")
```

With a private folder in place, we can write the csv file there so that
we do not have to recreate the file in the future.

``` r
write_csv(y, "private/tv_show_transcripts.csv")
```

## Overview of the Data

The resulting data frame is made up of the sixty observations of
individual scraped webpages The TV show title is listed under the `id`
column. In the `clean_wrds` column, the clean text of each page is
included. Importantly, these are divided up roughly evenly my genre. I
intend to add the genre information to the ID and to get a summary of
how many tokens are in each observation as immediate next steps.

``` r
sessionInfo()
```

    R version 4.6.1 (2026-06-24 ucrt)
    Platform: x86_64-w64-mingw32/x64
    Running under: Windows 11 x64 (build 26200)

    Matrix products: default
      LAPACK version 3.12.1

    locale:
    [1] LC_COLLATE=English_United States.utf8 
    [2] LC_CTYPE=English_United States.utf8   
    [3] LC_MONETARY=English_United States.utf8
    [4] LC_NUMERIC=C                          
    [5] LC_TIME=English_United States.utf8    

    time zone: America/New_York
    tzcode source: internal

    attached base packages:
    [1] stats     graphics  grDevices utils     datasets  methods   base     

    other attached packages:
     [1] polite_0.1.4    rvest_1.0.5     lubridate_1.9.5 forcats_1.0.1  
     [5] stringr_1.6.0   dplyr_1.2.1     purrr_1.2.2     readr_2.2.0    
     [9] tidyr_1.3.2     tibble_3.3.1    ggplot2_4.0.3   tidyverse_2.0.0

    loaded via a namespace (and not attached):
     [1] utf8_1.2.6         generics_0.1.4     xml2_1.6.0         stringi_1.8.9     
     [5] hms_1.1.4          digest_0.6.39      magrittr_2.0.5     evaluate_1.0.5    
     [9] grid_4.6.1         timechange_0.4.0   RColorBrewer_1.1-3 fastmap_1.2.0     
    [13] jsonlite_2.0.0     robotstxt_0.7.15   httr_1.4.9         selectr_0.8-0     
    [17] scales_1.4.0       cli_3.6.6          crayon_1.5.3       rlang_1.3.0       
    [21] bit64_4.8.4        withr_3.0.3        cachem_1.1.0       yaml_2.3.12       
    [25] otel_0.2.0         parallel_4.6.1     tools_4.6.1        tzdb_0.5.0        
    [29] memoise_2.0.1      ratelimitr_0.4.2   curl_8.0.0         assertthat_0.2.1  
    [33] vctrs_0.7.3        R6_2.6.1           lifecycle_1.0.5    bit_4.6.0         
    [37] fs_2.1.0           vroom_1.7.1        usethis_3.2.2      pkgconfig_2.0.3   
    [41] pillar_1.11.1      gtable_0.3.6       Rcpp_1.1.2         glue_1.8.1        
    [45] xfun_0.60          tidyselect_1.2.1   rstudioapi_0.19.0  knitr_1.51        
    [49] spiderbar_0.2.5    farver_2.1.2       htmltools_0.5.9    rmarkdown_2.31    
    [53] compiler_4.6.1     S7_0.2.2          
