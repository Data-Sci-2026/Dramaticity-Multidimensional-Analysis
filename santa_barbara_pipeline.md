# Santa Barbara Pipeline
Zach Shultz
2026-10-25

## Prereqs

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

## Intro

This document is a place for process for creating the Santa Barabara
Code pipeline.

## Reading in the data

There were two different file types provided for the Santa Barbara
Corpus- .trn and .cha. I decided to use .trn files since they seemed to
read in more naturally as a data frame. For the purposes of this
document, we can use the test .trn file I used, the first .trn.

``` r
file <- "data/SBCorpus/TRN/SBC001.trn"

#reads in a single trn file and lets R decide the column names
trn_df <-  read_tsv(file, col_names = FALSE)
```

    Rows: 1312 Columns: 3
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: "\t"
    chr (3): X1, X2, X3

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
trn_df
```

    # A tibble: 1,312 × 3
       X1          X2      X3                                                       
       <chr>       <chr>   <chr>                                                    
     1 0.00 9.21   LENORE: ... So you don't need to go ... borrow equipment from an…
     2 9.21 9.52   <NA>    to --                                                    
     3 9.52 14.10  <NA>    ... to do the feet?                                      
     4 14.10 15.78 <NA>    ... [Do the hooves]?                                     
     5 15.01 16.78 LYNNE:  [(H)=] <YWN Well,                                        
     6 16.78 18.32 <NA>    we're gonna have to find somewhere,                      
     7 18.33 18.85 <NA>    to get,                                                  
     8 18.85 20.69 <NA>    (Hx) ... something (Hx) YWN>.                            
     9 20.69 21.19 DORIS:  .. So,                                                   
    10 21.19 21.74 <NA>    [~Mae-] --                                               
    # ℹ 1,302 more rows

Originally, I named the columns. However, I later realized that some of
the files had different delimiters which changed the number of columns
as a result from 3 to 2 or 4. To fix this, I simply had R select the
final column, which was always the words.

Also worth noting that I originally did not specify the delimiter. This
was a big mistake. The last file has a weird unicode symbol thrown in
for some reason and it took a while to figure out that I just needed to
specify the delimiter (and add some regex to clean up the unicode).

## Cleaning the Data

Unfortunately, the problems did not end there. These files also had some
minor corruption of some kind that caused null embeddings. Five of the
files had a ‘c’ character replaced with a null character. Using
notepad++, I was able to find these characters by searching `\x{0000}`
in the regex search. These changes are reflected in the trns in the
`data` folder.

Additionally, the files have numerous inconsistencies in terms of tab
delimiters, comment typos, and various other issues. As a result, some
work had to be done on the files before they could even be safely
transformed into data frames.

To this end, the following function was created:

``` r
clean_files <- function(file) {

#removes comments
file <- file[!str_detect(file, "^000")]

#removes mid line comments 
file <- str_remove(file, "000000000.*")

#indents an extra line
file <- str_replace(file, "[A-Z]+: (\\d\\d?\\.)", "\n\\1") |> unlist() 

#removes initial tabs
file <- str_remove(file, "^\t")

#removes white space on the right side of text
file <- str_trim(file, side = "right")

#removes speaker names followed by : with an optional _digit pattern between
file <- str_remove_all(file, "[A-Z]+_?\\d?:")
#removes speaker names not followed by colon as long as they are longer than 4
file <- str_remove_all(file, "[A-Z]{4,}")

#removes all time stamps
file <- str_remove_all(file, "\\d+\\.\\d+\t\\d+\\.\\d+")
file <- str_remove_all(file, "\\d+\\.\\d+ \\d+\\.\\d+")

#removes all tabs
file <- str_remove_all(file, "\t")
}
```

Below, we can see the results on a single file. We now have only text
remaining and only one column to deal with.

``` r
trn_df <- clean_files(read_lines(file))

trn_df[1:20]
```

     [1] " ... So you don't need to go ... borrow equipment from anybody,"
     [2] "        to --"                                                  
     [3] "        ... to do the feet?"                                    
     [4] "        ... [Do the hooves]?"                                   
     [5] "      [(H)=] <YWN Well,"                                        
     [6] "           we're gonna have to find somewhere,"                 
     [7] "        to get,"                                                
     [8] "        (Hx) ... something (Hx) YWN>."                          
     [9] "  .. So,"                                                       
    [10] "        [~Mae-] --"                                             
    [11] "  [I'm gonna] (Hx) --"                                          
    [12] "  [2~Mae ~Lynne XX2]"                                           
    [13] "  [2(H) We're not2] gonna do the feet today,"                   
    [14] "        I'm gonna wait till like,"                              
    [15] "        early in the morning=,"                                 
    [16] "        .. to do those,"                                        
    [17] "        cause y- --"                                            
    [18] "        I mean you get s=o ti=red."                             
    [19] "        (H) ... n- you just,"                                   
    [20] "        ... it takes % --"                                      

And here is our single file read in from the cleaned data.

``` r
file <- "private/SBCorpus_cleaned/SBC001.trn"

#reads in a single trn file and lets R decide the column names
trn_df <-  read_tsv(file, col_names = FALSE)
```

    Rows: 1312 Columns: 1
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: "\t"
    chr (1): X1

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
trn_df
```

    # A tibble: 1,312 × 1
       X1                                                            
       <chr>                                                         
     1 ... So you don't need to go ... borrow equipment from anybody,
     2 to --                                                         
     3 ... to do the feet?                                           
     4 ... [Do the hooves]?                                          
     5 [(H)=] <YWN Well,                                             
     6 we're gonna have to find somewhere,                           
     7 to get,                                                       
     8 (Hx) ... something (Hx) YWN>.                                 
     9 .. So,                                                        
    10 [~Mae-] --                                                    
    # ℹ 1,302 more rows

In order to iterate, we will need a list of all the file paths.

``` r
#here we go into the file TRN and pull out all of the full names for the files
files = list.files("data/SBCorpus/TRN", full.names = TRUE)
```

``` r
new_files <- files |>
  #first, we read the lines of every file
  map(read_lines) |>
  #then, we clean all of them
  map(clean_files)
```

Once this has been done, `new_files` is a list of the full documents.
Now, we need to store all of them back onto file paths. First, though,
we need a new folder. We do not want to overwrite the old files since we
might want to use them later on. Instead, we can use the same file names
but store them in a distinct place.

This code creates a private folder if there is no such folder already.

``` r
if(!dir.exists("private")) 
  dir.create("private")
```

And this code creates a new folder within private to store the cleaned
files.

``` r
#make a new folder for the fixed files
dir.create("private/SBCorpus_cleaned")
```

    Warning in dir.create("private/SBCorpus_cleaned"): 'private\SBCorpus_cleaned'
    already exists

Finally, we have to create the paths for these files based on the old
ones inside of our new folder, then map our new files onto new paths and
write out the lines into those names.

``` r
#create a list of file paths
new_paths <- file.path("private/SBCorpus_cleaned",
  #this gets the name of each file from the relative path
  basename(files))

#map those files onto file paths and the lines onto those names
invisible(map2(new_files, new_paths, write_lines))
```

We now have our new files that can be read through a mapping function as
dfs without issue.

## Cleaning the text

The next step was to create a function called `clean()` that would clean
each line of the transcript. This process started with identifying
symbols that were not part of the actual dialogue and then figuring out
what they meant. I listened to audio versions of the first .trn file to
get a sense of any transcription markings I wasn’t familiar with.
Ultimately, I ended up with the following function for cleaning the
lines.

``` r
clean <- function(string) {
  string |>
  #replaces a very specific unicode symbol with apostrophe's because ONE of the files uses this
  str_replace_all("\x92", "'") |>
  #before anything, get rid of the #X# combinations cause that causes problems
  str_replace_all("[1-9]X+[1-9]", "") |>
  #replace a ( followed by any number of any characters but a ) and then a )
  str_replace_all("\\([^)]*\\)", "") |>
  #replace any < followed by any number of characters but space
  str_replace_all("<[^ ]*", "") |>
  #replace any number of characters but a space immediately before a >
  str_replace_all("[^ ]*>", "") |>
  #replace an underscore followed by a single character or an underscore by itself
  str_replace_all("_. |_", " ") |>
  #replace a tilde followed by any character but a space with an empty string
  str_replace_all("~[^ ]*", "") |>
  #remove each of the symbols inside, all of them
  str_remove_all("[\\[\\]+%@#=*X\\)\\(]") |>
  #remove two or more literal dots, all of them
  str_remove_all("\\.\\.+") |>
  #removes all exclamation points that precede a word
  str_replace_all("!(\\w)", "\\1") |>
  #remove two or more dashes, all of them
  str_remove_all("--+") |>
  #remove all spaces at the beginning or end of the line
  str_replace_all("^ | $", "") |>
  #removes numbers
  str_replace_all("[1-9]", " ") |>
  #replace any space larger than one spaces with single spaces
  str_replace_all("[ ]+", " ") |>
  #replaces any space before a period or comma with no space
  str_replace_all(" \\.", "\\.") |>
  str_replace_all(" \\,", "\\,") 
}
#clean cleans up a data frame column made up of transcription markings to make it as close as possible to only dialogue. 
```

In order to clean the column, we first select only the last column
(defined by `columns`) and then use the function outlined above.

``` r
columns <- ncol(trn_df)

trn_wrds <- trn_df |>
  select(words = all_of(columns))

clean_trn_wrds <- trn_wrds |>
  mutate(clean_words = clean(words))

trn_df
```

    # A tibble: 1,312 × 1
       X1                                                            
       <chr>                                                         
     1 ... So you don't need to go ... borrow equipment from anybody,
     2 to --                                                         
     3 ... to do the feet?                                           
     4 ... [Do the hooves]?                                          
     5 [(H)=] <YWN Well,                                             
     6 we're gonna have to find somewhere,                           
     7 to get,                                                       
     8 (Hx) ... something (Hx) YWN>.                                 
     9 .. So,                                                        
    10 [~Mae-] --                                                    
    # ℹ 1,302 more rows

Above, we see the clean version in the second column!

Once each individual line was cleaned, the code below as implemented to
get rid of white space, empty lines, and make sure only the
`clean_words` column was selected.

``` r
trn_text <- clean_trn_wrds |>
  #get rid of messy version of each line
  select(!words) |>
  #trim white space 
  mutate(clean_words = trimws(clean_words)) |>
  #filter out empty lines
  filter(!clean_words == '') |>
  mutate(trn = "x",
  #add a column fill in for unifying these features
         .before = clean_words)

trn_text
```

    # A tibble: 1,285 × 2
       trn   clean_words                                           
       <chr> <chr>                                                 
     1 x     So you don't need to go borrow equipment from anybody,
     2 x     to                                                    
     3 x     to do the feet?                                       
     4 x     Do the hooves?                                        
     5 x     Well,                                                 
     6 x     we're gonna have to find somewhere,                   
     7 x     to get,                                               
     8 x     something.                                            
     9 x     So,                                                   
    10 x     I'm gonna                                             
    # ℹ 1,275 more rows

## Merging the text

With the text being clean, the last line of code in the previous chunk
was added to prepare for the following code.

``` r
trn_text <- trn_text |>
  #returns a single cell data frame of all of the dialogue combined with spaces
  summarize(clean_words = str_flatten(clean_words, " "))

trn_text
```

    # A tibble: 1 × 1
      clean_words                                                                   
      <chr>                                                                         
    1 So you don't need to go borrow equipment from anybody, to to do the feet? Do …

This successfully collapses the data frame into a single observation.
When mapped onto all the files, it will combine each of their lines into
a single string for each of them.

## Making the id column

In order to make an id column that kept track of which .trn file was
being observed, the code below was implemented.

``` r
#takes the thee numbers attached to each .trn
trn <- str_extract(file, "\\d+")

trn_df <- trn_text |>
  mutate(id = str_flatten(c("trn", trn), "_"),
#creates a full name for each doc as trn_###
         .before = "clean_words")

trn_df
```

    # A tibble: 1 × 2
      id      clean_words                                                           
      <chr>   <chr>                                                                 
    1 trn_001 So you don't need to go borrow equipment from anybody, to to do the f…

## The final Function and Iteration

``` r
#creating a function to get the cleaned up words into a data frame all together
get_clean_wrds <- function(file) {

#creates a variable that we can use to make file names later
trn <- str_extract(file, "\\d+")
  
#read in a file as a df
trn_df <-  read_tsv(file, col_names = FALSE, show_col_types = FALSE)

#ok we're trying something much simpler than we had before. At this point the pattern can be described as "select a column number equal to the number of columns because we always want the last column. So if we just get ourselves that variable and then just select that, we should be fine
columns <- ncol(trn_df)

trn_wrds <- trn_df |>
  select(words = all_of(columns))


clean <- function(string) {
  string |>
  #replaces a very specific unicode symbol with apostrophe's because ONE of the files uses this
  str_replace_all("\x92", "'") |>
  #before anything, get rid of the #X# combinations cause that causes problems
  str_replace_all("[1-9]X+[1-9]", "") |>
  #replace a ( followed by any number of any characters but a ) and then a )
  str_replace_all("\\([^)]*\\)", "") |>
  #replace any < followed by any number of characters but space
  str_replace_all("<[^ ]*", "") |>
  #replace any number of characters but a space immediately before a >
  str_replace_all("[^ ]*>", "") |>
  #replace an underscore followed by a single character or an underscore by itself
  str_replace_all("_. |_", " ") |>
  #replace a tilde followed by any character but a space with an empty string
  str_replace_all("~[^ ]*", "") |>
  #remove each of the symbols inside, all of them
  str_remove_all("[\\[\\]+%@#=*X\\)\\(]") |>
  #remove two or more literal dots, all of them
  str_remove_all("\\.\\.+") |>
  #removes all exclamation points that precede a word
  str_replace_all("!(\\w)", "\\1") |>
  #remove two or more dashes, all of them
  str_remove_all("--+") |>
  #remove all spaces at the beginning or end of the line
  str_replace_all("^ | $", "") |>
  #removes numbers
  str_replace_all("[1-9]", " ") |>
  #replace any space larger than one spaces with single spaces
  str_replace_all("[ ]+", " ") |>
  #replaces any space before a period or comma with no space
  str_replace_all(" \\.", "\\.") |>
  str_replace_all(" \\,", "\\,") 
}
#clean cleans up a data frame column made up of transcription markings to make it as close as possible to only dialogue. 

#Using clean() to clean up a data frame

clean_trn_wrds <- trn_wrds |>
  mutate(clean_words = clean(words))

#Getting the rows of dialogue into a single cell of dialogue

trn_text <- clean_trn_wrds |>
  #get rid of messy version of each line
  select(!words) |>
  #trim white space 
  mutate(clean_words = trimws(clean_words)) |>
  #filter out empty lines
  filter(!clean_words == '') |>
  mutate(trn = "x",
  #add a column fill in for unifying these features
         .before = clean_words)

trn_text <- trn_text |>
  #returns a single cell data frame of all of the dialogue combined with spaces
  summarize(clean_words = str_flatten(clean_words, " "))

trn_df <- trn_text |>
  mutate(id = str_flatten(c("trn", trn), "_"),
         .before = "clean_words")

trn_df
}
```

The code below was used to create a list of every file name from the new
folder folder.

``` r
cleaned_files <- list.files("private/SBCorpus_cleaned", full.names = TRUE)
```

And finally, the code below was used to map this function onto every
file in the files list.

``` r
list_files <- cleaned_files |>
  map(get_clean_wrds) 

sb_files <- bind_rows(list_files)
```

With a private folder in place, we can write the csv file there so that
we do not have to recreate the file in the future.

``` r
write_csv(sb_files, "private/sb_files.csv")
```

## Overview of the Data

The resulting data frame is made up of the sixty observations of
individual trn files. The file number is listed under the `id` column.
In the `clean_wrds` column, the clean text of each file is included. In
total, we have approximately 249,000 word tokens in the text section.

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
     [1] lubridate_1.9.5 forcats_1.0.1   stringr_1.6.0   dplyr_1.2.1    
     [5] purrr_1.2.2     readr_2.2.0     tidyr_1.3.2     tibble_3.3.1   
     [9] ggplot2_4.0.3   tidyverse_2.0.0

    loaded via a namespace (and not attached):
     [1] bit_4.6.0          gtable_0.3.6       jsonlite_2.0.0     crayon_1.5.3      
     [5] compiler_4.6.1     tidyselect_1.2.1   parallel_4.6.1     scales_1.4.0      
     [9] yaml_2.3.12        fastmap_1.2.0      R6_2.6.1           generics_0.1.4    
    [13] knitr_1.51         pillar_1.11.1      RColorBrewer_1.1-3 tzdb_0.5.0        
    [17] rlang_1.3.0        utf8_1.2.6         stringi_1.8.9      xfun_0.60         
    [21] S7_0.2.2           bit64_4.8.4        otel_0.2.0         timechange_0.4.0  
    [25] cli_3.6.6          withr_3.0.3        magrittr_2.0.5     digest_0.6.39     
    [29] grid_4.6.1         vroom_1.7.1        rstudioapi_0.19.0  hms_1.1.4         
    [33] lifecycle_1.0.5    vctrs_0.7.3        evaluate_1.0.5     glue_1.8.1        
    [37] farver_2.1.2       rmarkdown_2.31     tools_4.6.1        pkgconfig_2.0.3   
    [41] htmltools_0.5.9   
