# Project Plan

# Dramaticity and Authenticity: A Multidimensional Analysis of Register between Natural Discourse and TV Show Transcripts

## Summary

One component of a dialogue is how 'dramatic' or 'performative' the language seems.
This study involves trying to determine what linguistic features make a particular text more 'dramatic' or 'performative'.
More specifically, what are the sorts of linguistic characteristics that make tv shows feel more 'performative' than real discourse. 
Using a corpus of realistic speech data and a corpus of television show data, I plan to run a multidimensional analysis to detect lexical and grammatical differences that could suggest the 'dramatic' elements of different texts.

## Overview of the Data

The data for this analysis will come from two corpora.
First, I plan to draw natural discourse from a subcorpus or sampling of the [Santa Barbara Corpus of Spoken American English](https://linguistics.ucsb.edu/research/santa-barbara-corpus-spoken-american-english).
The corpus is made up of sixty different transcripts composing 249,000 words. 
It also contains useful metadata such as name, gender, age, hometown, home state, current state, education, years of education, occupation, and ethnicity.

Second, the TV show data will come from the [Scraps from the Loft](https://scrapsfromtheloft.com/tv-series-transcripts/).
This website is a massive assembling of 3,264 completed episode transcripts from 315 different series.


## The Analysis

I intend to run [Biber's 1988 multidimensional analysis (MDA) on register](https://www.jstor.org/stable/43267884). 
The MDA contains five potential dimensions containing 67 linguistic features. 
Using a python package, I will use a data frame comprised of name (representing categorical variables, in this case natural or TV show) and text from the data described in the previous section to return a data frame of tagged features.
Importantly, these tagged features are not perfect and I should take care that the data is formatted to help the tagger be as correct as possible.
This data frame of tagged features will then be again transformed into a data frame listing a score on each of the 67 linguistic features for each text (ie the data frame will have 68 columns and n rows where n is the number of documents).
Then, using a scree plot, I will determine the number of factors that need to be calculated. 
Using an R package called `mda.biber`, I will take this data frame of scores and compute their factor weights (for the number of factors determined above) in a process explained at length [here](https://cran.r-project.org/web/packages/mda.biber/vignettes/introduction.html).

Once these dimension scores are determined, I intend to identify the linguistic features which correlate and suggest specifically the 'dramatic' nature of the text based on previous work and observations by Al-Surmi, Bednarek, Quaglio, and others. 

It is possible that this analysis will not be suited to the specifics of the study. 
Regardless, my hope is that the linguistic features laid out for the analysis will be helpful as they get at the question of register in a comprehensive way.

As a rough hypothesis, I would predict that the TV show corpus as a whole will feature more 'dramatic' language however that will likely vary based on genre and context.

## Data Wrangling

The biggest challenge in this process will almost certainly be retrieving the necessary data from Scraps from the Loft and transforming it into an adequate data frame for the analysis above. 
To a lesser extent, doing the same for the Santa Barbara Corpus may also be a challenge. 
Since each data source is quite unique, it is important to have two different pipelines for gathering the necessary data. 
The Road Map section outlines the steps (separated into three different submissions) that will be necessary to wrangle the data.

## Road Map

In order to break this project up into manageable pieces, below is a specific description of SMART goals for each project report. 
SMART goals must be: Specific, Measurable, Achievable, Relevant, and Time Bound.

#### Progress Report 1

GOAL:
For the first project report, I will have a complete data frame schematized as below:

| ID (TV Show or Natural) |                                   Text                                         |
|-------------------------|--------------------------------------------------------------------------------|
|       TV Show 1         | "What did you say? I told you I'm through. What are you even doing here?..."   |
|-------------------------|--------------------------------------------------------------------------------|
|       TV Show 2         | "What did you say? I told you I'm through. What are you even doing here?..."   |
|-------------------------|--------------------------------------------------------------------------------|
|       TV Show 3         | "What did you say? I told you I'm through. What are you even doing here?..."   |
|-------------------------|--------------------------------------------------------------------------------|
|       TV Show 4         | "What did you say? I told you I'm through. What are you even doing here?..."   |
|-------------------------|--------------------------------------------------------------------------------|
|       Natural 1         | "Huh? I said it's. Oh alright. Yeah. Did you hear? They just said who...."     |
|-------------------------|--------------------------------------------------------------------------------|
|       Natural 2         | "Huh? I said it's. Oh alright. Yeah. Did you hear? They just said who...."     |
|-------------------------|--------------------------------------------------------------------------------|
|       Natural 3         | "Huh? I said it's. Oh alright. Yeah. Did you hear? They just said who...."     |
|-------------------------|--------------------------------------------------------------------------------|
|       Natural 4         | "Huh? I said it's. Oh alright. Yeah. Did you hear? They just said who...."     |
|-------------------------|--------------------------------------------------------------------------------|

Creating this will require a pipeline which scales up the following process for a single file (different for each data source):

Santa Barabra Corpus
Download a transcript as a .txt file --> load it into R --> process the text to eliminate non-text characters --> combine into a single continuous string --> add to a data frame as one cell of a text column

Scraps from the Loft
Use [webscraping](https://djvill.github.io/r4ds/webscraping.html) to get a particular page (the transcript of one episode of a tv show) into a text format --> process the text to eliminate non-text characters --> combine into a single continuous string --> add to a data frame as one cell of a text column

#### Progress Report 2

GOAL: 
For this project report, I will take all of the required steps for preparing an MDA and finalize the license for the project, and record the features of the important data frames.

Below is a pipeline which describes in more detail the required processing to run the MDA.

Take the df we have and run it through spacy parsing to get the parts of speech --> token and type counts, and grammatical relations --> run it through pybiber to get 67 linguistic features --> load into R and eliminate columns with **only** zeros --> create a scree plot --> determine necessary number of dimensions --> create a correlation matrix to ensure data is well suited 

Below is a schema for the general shape of the important dfs to be recorded:

Data Frame of Spacy Parsing

| ID (TV Show or Natural)  | sentence_id |token_id  |	 token   | lemma  |  pos	 | tag	 | head_token_id	| dep_rel |   
|       TV Show 1          |     1       |    1     |   "what" | "what" | "ADV"  | "RB"  |      3         |  x      |
|       TV Show 1          |     1       |    2     |   "did"  | "do"   | "VERB" | "VBD" |      3         |  x      |
|       TV Show 1          |     1       |    3     |   "you"  | "you"  | "ADV"  | "PRN" |      3         |  x      |
|       TV Show 1          |     1       |    4     |   "say"  | "say"  | "VERB" | "VBD" |      3         |  x      |
|       Natural 1          |     1       |    1     |   "huh"  | "huh"  | "ADV"  | "RB"  |      1         |  x      |


Data frame of 67 linguistic features

|doc_id |	f_01_past_tense |	f_02_perfect_aspect |	f_03_present_tense | f_04_place_adverbials | f_05_time_adverbials |

## Concluding Remarks

The big hiccups I foresee are 1) the accuracy of the spacy tagging particularly given odd discourse features (might be the biggest problem in the natural discourse) and 2) complications in webscraping which is something I haven't done before and seems quite complex. 

Necessary Citations:
Biber 1988
Santa Barabara Corpus
Scraps from the Loft
pybiber script
mda.biber package