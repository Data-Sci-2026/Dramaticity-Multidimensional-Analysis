# Santa Barbara Pipeline
Zach Shultz
2026-10-25

prereqs

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
file <- "private/SBCorpus/TRN/SBC001.trn"

trn_df <-  read_delim(file, delim = "\t", col_names = FALSE)
```

    Rows: 1312 Columns: 3
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: "\t"
    chr (3): X1, X2, X3

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#reads in a single trn file and lets R decide the column names

trn_df
```

    # A tibble: 1,312 × 3
       X1          X2         X3                                                    
       <chr>       <chr>      <chr>                                                 
     1 0.00 9.21   "LENORE: " "... So you don't need to go ... borrow equipment fro…
     2 9.21 9.52   "        " "to --"                                               
     3 9.52 14.10  "        " "... to do the feet?"                                 
     4 14.10 15.78 "        " "... [Do the hooves]?"                                
     5 15.01 16.78 "LYNNE:  " "    [(H)=] <YWN Well,"                               
     6 16.78 18.32 "        " "   we're gonna have to find somewhere,"              
     7 18.33 18.85 "        " "to get,"                                             
     8 18.85 20.69 "        " "(Hx) ... something (Hx) YWN>."                       
     9 20.69 21.19 "DORIS:  " ".. So,"                                              
    10 21.19 21.74 "        " "[~Mae-] --"                                          
    # ℹ 1,302 more rows

Originally, I named the columns. However, I later realized that some of
the files had different delimiters which changed the number of columns
as a result from 3 to 2 or 4. To fix this, I simply had R select the
final column, which was always the words.

Also worth noting that I originally did not specify the delimiter. This
was a big mistake. The last file has a weird unicode symbol thrown in
for some reason and it took a while to figure out that I just needed to
specify the delimiter (and add some regex to clean up the unicode).

## Cleaning the text

The next step was to create a function called `clean()` that would clean
each line of the transcript. This process started with identifying
symbols that were not part of the actual dialogue and then figuring out
what they meant. I listened to audio versions of the first .trn file to
get a sense of any transcription markings I wasn’t familiar with.
Ultimately, I ended up with the following function for cleaning the
lines.

It was necessary to create two functions inside of `clean()` in order to
repeat phrases that were marked with a \# words \# pattern. These are
described in detail below and in the final code.

``` r
repetitions <- function(string) {
  group <- string |>
  str_extract_all("([1-9])[^X]*\\1")
  #find the pattern we're looking for
  times <- group |>
  str_extract("[1-9]")
  #figure out the marking of how many times it should occur
  times <- as.numeric(times)
  #turn that marking into an actual number
  multiple <- strrep(group, times)
  #multiply that string by the multiple
  group <- string |>
  str_extract_all("([1-9])[^X]*\\1")
  #redefine the pattern we're looking for because group had to be renamed before (this is probably redundant but I'm confused rn so I don't wanna figure out the better way)
  string |>
    str_replace_all("([1-9])[^X]*\\1", multiple)
  #replace the initial string with what we want
}
enact_repeats <- function(string) {
  if_else(str_detect(string, "([1-9])[^X]*\\1"), repetitions(string), string)
}
#enact_repeats finds strings surrounded by numbers and enacts repetitions function (explained above)
clean <- function(string) {
  string |>
  str_replace_all("\x92", "'") |>
  #replaces a specific unicode symbol with apostrophe's because ONE of the files uses this
  str_replace_all("[1-9]X+[1-9]", "") |>
  #before anything, get rid of the #X# combinations cause that causes problems
  enact_repeats()|>
  str_replace_all("\\([^)]*\\)", "") |>
  #replace a ( followed by any number of any characters but a ) and then a )
  str_replace_all("<[^ ]*", "") |>
  #replace any < followed by any number of characters but space
  str_replace_all("[^ ]*>", "") |>
  #replace any number of characters but a space immediately before a >
  str_replace_all("_. |_", " ") |>
  #replace an underscore followed by a single character or an underscore by itself
  str_replace_all("~[^ ]*", "") |>
  #replace a tilde followed by any character but a space with an empty string
  str_remove_all("[\\[\\]+%@=*X\\)\\(]") |>
  #remove each of the symbols inside, all of them
  str_remove_all("\\.\\.+") |>
  #remove two or more literal dots, all of them
  str_replace_all("!(\\w)", "\\1") |>
  str_remove_all("--+") |>
  #remove two or more dashes, all of them
  str_replace_all("^ | $", "") |>
  #remove all spaces at the beginning or end of the line
  str_replace_all("[1-9]", " ") |>
  #replace spaces at the beginning and end of a line with no spaces
  str_replace_all("[ ]+", " ") |>
  #replace any space larger than one spaces with single spaces
  str_replace_all(" \\.", "\\.") |>
  str_replace_all(" \\,", "\\,") 
  #replaces any space before a period or comma with no space
}
```

In order to clean the column, we first select only the last column
(defined by `columns`) and then use the function outlined above.

``` r
columns <- ncol(trn_df)

trn_wrds <- trn_df |>
  select(words = columns)
```

    Warning: Using an external vector in selections was deprecated in tidyselect 1.1.0.
    ℹ Please use `all_of()` or `any_of()` instead.
      # Was:
      data %>% select(columns)

      # Now:
      data %>% select(all_of(columns))

    See <https://tidyselect.r-lib.org/reference/faq-external-vector.html>.

``` r
clean_trn_wrds <- trn_wrds |>
  mutate(clean_words = clean(words))
```

    Warning: There was 1 warning in `mutate()`.
    ℹ In argument: `clean_words = clean(words)`.
    Caused by warning in `stri_extract_first_regex()`:
    ! argument is not an atomic vector; coercing

``` r
clean_trn_wrds
```

    # A tibble: 1,312 × 2
       words                                                            clean_words 
       <chr>                                                            <chr>       
     1 "... So you don't need to go ... borrow equipment from anybody," "So you don…
     2 "to --"                                                          "to"        
     3 "... to do the feet?"                                            "to do the …
     4 "... [Do the hooves]?"                                           "Do the hoo…
     5 "    [(H)=] <YWN Well,"                                          " Well,"    
     6 "   we're gonna have to find somewhere,"                         " we're gon…
     7 "to get,"                                                        "to get,"   
     8 "(Hx) ... something (Hx) YWN>."                                  " something…
     9 ".. So,"                                                         "So,"       
    10 "[~Mae-] --"                                                     ""          
    # ℹ 1,302 more rows

Above, we see the clean version in the second column!

Once each individual line was cleaned, the code below as implemented to
get rid of white space, empty lines, and make sure only the
`clean_words` column was selected.

``` r
trn_text <- clean_trn_wrds |>
  select(!words) |>
  #get rid of messy version of each line
  mutate(clean_words = trimws(clean_words)) |>
  #trim white space 
  filter(!clean_words == '') |>
  #filter out empty lines
  mutate(trn = "x",
         .before = clean_words)
  #add a column fill in for unifying these features

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
  summarize(clean_words = str_flatten(clean_words, " "))
  #returns a single cell data frame of all of the dialogue combined with spaces

trn_text
```

    # A tibble: 1 × 1
      clean_words                                                                   
      <chr>                                                                         
    1 So you don't need to go borrow equipment from anybody, to to do the feet? Do …

This successfully collapses the data frame into a single observation.

## Making the id column

In order to make an id column that kept track of which .trn file was
being observed, the code below was implemented.

``` r
trn <- str_extract(file, "\\d+")
#takes the thee numbers attached to each .trn

trn_df <- trn_text |>
  mutate(id = str_flatten(c("trn", trn), "_"),
         .before = "clean_words")
#creates a full name for each doc as trn_###

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

#reading in a file as a data frame

trn <- str_extract(file, "\\d+")
  
trn_df <-  read_delim(file, delim = "\t", col_names = FALSE)
#read in a file as a df

columns <- ncol(trn_df)
#ok we're trying something much simpler than we had before. At this point the pattern can be described as "select a column number equal to the number of columns because we always want the last column. So if we just get ourselves that variable and then just select that, we should be fine

trn_wrds <- trn_df |>
  select(words = columns)

# Function Definitions

#repetitions takes a given string and multiplies it by numbers that surround the string
repetitions <- function(string) {
  group <- string |>
  str_extract_all("([1-9])[^X]*\\1")
  #find the pattern we're looking for
  times <- group |>
  str_extract("[1-9]")
  #figure out the marking of how many times it should occur
  times <- as.numeric(times)
  #turn that marking into an actual number
  multiple <- strrep(group, times)
  #multiply that string by the multiple
  group <- string |>
  str_extract_all("([1-9])[^X]*\\1")
  #redefine the pattern we're looking for because group had to be renamed before (this is probably redundant but I'm confused rn so I don't wanna figure out the better way)
  string |>
    str_replace_all("([1-9])[^X]*\\1", multiple)
  #replace the initial string with what we want
}
enact_repeats <- function(string) {
  if_else(str_detect(string, "([1-9])[^X]*\\1"), repetitions(string), string)
}
#enact_repeats finds strings surrounded by numbers and enacts repetitions function (explained above)
clean <- function(string) {
  string |>
  str_replace_all("\x92", "'") |>
  #replaces a very specific unicode symbol with apostrophe's because ONE of the files uses this
  str_replace_all("[1-9]X+[1-9]", "") |>
  #before anything, get rid of the #X# combinations cause that causes problems
  enact_repeats()|>
  str_replace_all("\\([^)]*\\)", "") |>
  #replace a ( followed by any number of any characters but a ) and then a )
  str_replace_all("<[^ ]*", "") |>
  #replace any < followed by any number of characters but space
  str_replace_all("[^ ]*>", "") |>
  #replace any number of characters but a space immediately before a >
  str_replace_all("_. |_", " ") |>
  #replace an underscore followed by a single character or an underscore by itself
  str_replace_all("~[^ ]*", "") |>
  #replace a tilde followed by any character but a space with an empty string
  str_remove_all("[\\[\\]+%@=*X\\)\\(]") |>
  #remove each of the symbols inside, all of them
  str_remove_all("\\.\\.+") |>
  #remove two or more literal dots, all of them
  str_replace_all("!(\\w)", "\\1") |>
  str_remove_all("--+") |>
  #remove two or more dashes, all of them
  str_replace_all("^ | $", "") |>
  #remove all spaces at the beginning or end of the line
  str_replace_all("[1-9]", " ") |>
  #removes numbers
  str_replace_all("[ ]+", " ") |>
  #replace any space larger than one spaces with single spaces
  str_replace_all(" \\.", "\\.") |>
  str_replace_all(" \\,", "\\,") 
  #replaces any space before a period or comma with no space
}
#clean cleans up a data frame column made up of transcription markings to make it as close as possible to only dialogue. 

#Using clean() to clean up a data frame

clean_trn_wrds <- trn_wrds |>
  mutate(clean_words = clean(words))

#Getting the rows of dialogue into a single cell of dialogue

trn_text <- clean_trn_wrds |>
  select(!words) |>
  #get rid of messy version of each line
  mutate(clean_words = trimws(clean_words)) |>
  #trim white space 
  filter(!clean_words == '') |>
  #filter out empty lines
  mutate(trn = "x",
         .before = clean_words)
  #add a column fill in for unifying these features

trn_text <- trn_text |>
  summarize(clean_words = str_flatten(clean_words, " "))
  #returns a single cell data frame of all of the dialogue combined with spaces

trn_df <- trn_text |>
  mutate(id = str_flatten(c("trn", trn), "_"),
         .before = "clean_words")

trn_df
}
```

The code below was used to create a list of every file name from the zip
folder.

``` r
files = list.files("SBCorpus/TRN", full.names = TRUE)
#here we go into the file TRN and pull out all of the full names for the files
files
```

And finally, the code below was used to map this function onto every
file in the files list.

``` r
list_files <- files |>
  map(get_clean_wrds) 

all_trns <- do.call(rbind, list_files)

all_trns
```

## Overview of the Data

The resulting data frame is made up of the sixty observations of
individual trn files. The file number is listed under the `id` column.
In the `clean_wrds` column, the clean text of each file is included. In
total, we have approximately 249,000 word tokens in the text section.
