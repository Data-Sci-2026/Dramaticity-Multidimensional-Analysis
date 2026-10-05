# Project Plan

# Dramaticity and Authenticity: A Multidimensional Analysis of Register in Natural Discourse and TV Show Transcripts

## Summary

One elusive aspect of artificial dialogue in TV shows and movies is a sense of how *dramatic* or *performative* the language is.
This study involves trying to determine what linguistic features make a particular text sound more *dramatic* or *performative* (from now on the term dramatic or dramaticity will be used by default).
More specifically, what are the sorts of linguistic characteristics that make TV shows feel more dramatic than real discourse. 

Using a corpus of realistic speech data and a corpus of television show data, I plan to run a multidimensional analysis to detect linguistic differences that could suggest the dramatic elements of different texts.

## Research Questions and Ultimate Goal

  RQ1: What are the linguistic features that co-vary to distinguish a natural discourse from a TV show discourse?
  
  RQ2: Of these features, which ones can be identified as particular markers of dramaticity. 
  
  RQ3 (optionally): Do specific genres of TV show seem to be more distinctly dramatic compared to natural discourse than others?
  
In understanding the linguistic features that contribute to dramaticity in spoken registers, we gain a more concrete understanding of how finely tuned our brains are to encoding each piece of language and indexing that with a particular association.
I feel that the dramatic nature of certain registers is something that has been underrepresented in the literature.
Therefore, this study will hopefully form a starting point to filling a gap in our understanding of how we perceive dramatic language in speech.

## Overview of the Data

The data for this analysis will come from two corpora.
First, I plan to draw natural discourse from the [Santa Barbara Corpus of Spoken American English](https://linguistics.ucsb.edu/research/santa-barbara-corpus-spoken-american-english).
The corpus is made up of sixty transcripts composing 249,000 words. 
It also contains useful metadata such as name, gender, age, hometown, home state, current state, education, years of education, occupation, and ethnicity.
It is not likely I will make use of this for this analysis, but it is good to be aware of. 

Second, the TV show data will come from [Scraps from the Loft](https://scrapsfromtheloft.com/tv-series-transcripts/).
This website is a massive assembling of 3,264 completed episode transcripts from 315 different series. 
I plan to select sixty episodes, each one from a different series, so that the number of documents is equal and, given the similarity in format, the word counts should be relatively balanced as well. 


## The Analysis

I intend to run [Biber's 1988 multidimensional analysis (MDA) on register](https://www.jstor.org/stable/43267884). 
The MDA contains five potential dimensions containing 67 linguistic features. 
First, I will create a data frame comprised of "name"" (representing categorical variables, in this case "natural"" or "TV show"") and uninterrupted text from the sources described in the previous section.
Using a python package, `pybiber`, I will create another data frame of tagged features.
Importantly, these tagged features are not perfect and I should take care that the data is formatted to help the tagger be as accurate as possible.
This data frame of tagged features will then be again transformed into a data frame listing a score on each of the 67 linguistic features for each sentence in each text.
The data frame will have 68 columns and n rows where n is the number of total sentences in all documents.
Then, using a scree plot, I will determine the number of factors that need to be calculated. 
Using an R package called `mda.biber`, I will take this data frame of scores and compute their factor weights (for the number of factors determined above) in a process explained at length [here](https://cran.r-project.org/web/packages/mda.biber/vignettes/introduction.html).

Once these dimension scores are determined, I intend to identify the linguistic features which correlate and suggest specifically the 'dramatic' nature of the text based on previous work and observations by Al-Surmi, Bednarek, Quaglio, and others. 
It is possible that this analysis will not be suited to the specifics of the study. 
Regardless, my hope is that the linguistic features laid out for the analysis will be helpful as they get at the question of register in a comprehensive way.

I predict that the TV show corpus as a whole will feature more 'dramatic' language however that will likely vary based on genre and context.

## Data Wrangling

The biggest challenge in this process will almost certainly be retrieving the necessary data from Scraps from the Loft and transforming it into an adequate data frame for the analysis above. 
To a lesser extent, doing the same for the Santa Barbara Corpus will also be a challenge. 
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
|       TV Show 2         | "What did you say? I told you I'm through. What are you even doing here?..."   |  
|       TV Show 3         | "What did you say? I told you I'm through. What are you even doing here?..."   |
|       TV Show 4         | "What did you say? I told you I'm through. What are you even doing here?..."   |
|       Natural 1         | "Huh? I said it's. Oh alright. Yeah. Did you hear? They just said who...."     |
|       Natural 2         | "Huh? I said it's. Oh alright. Yeah. Did you hear? They just said who...."     |
|       Natural 3         | "Huh? I said it's. Oh alright. Yeah. Did you hear? They just said who...."     |
|       Natural 4         | "Huh? I said it's. Oh alright. Yeah. Did you hear? They just said who...."     |
|        ...              |                                   ...                                          |

Creating this will require perfecting the pipelines below and then iterating that pipeline for multiple files:

Santa Barbara Corpus

Download a transcript as a .txt file --> load it into R --> process the text to eliminate non-text characters and add punctuation where needed (sentence boundaries must be evident) --> combine into a single continuous string --> add to a data frame as one cell of a text column

Scraps from the Loft

Use [webscraping](https://djvill.github.io/r4ds/webscraping.html) to get a particular page (the transcript of one episode of a tv show) into a text format --> load it into R --> process the text to eliminate non-text characters --> combine into a single continuous string --> add to a data frame as one cell of a text column

**Iteration process**:

I am unsure of how to implement the iteration for the Santa Barbara Corpus. 
Once all sixty files are downloaded as .txt files, it will simply be a matter of mapping to read in multiple files. 
However, I am uncertain whether I simply need to manually download each file.

For Scraps from the Loft, I assume once I understand the nested structure of the website, I can iterate to load in the different episode transcripts I want. 
However, while I have begun reading the chapter on webscraping, I still do not know exactly how it works so this portion of the iteration is also something I need to figure out as fast as possible. 

#### Progress Report 2

GOAL: 
For this project report, I will take all of the required steps for preparing an MDA and finalize the license for the project, and record the features of the important data frames.

Below is a pipeline which describes in more detail the required processing to run the MDA.

Take the df we have and run it through spacy parsing to get the parts of speech --> run it through pybiber to get 67 linguistic features --> load into R and eliminate columns with **only** zeros --> create a scree plot --> determine necessary number of dimensions --> create a correlation matrix to ensure data is well suited 

Below is a schema for the general shape of the important dfs to be recorded:

Data Frame of Spacy Parsing

| ID (TV Show or Natural)  | sentence_id |token_id  |	 token   | lemma  |  pos	 | tag	 | head_token_id	| dep_rel | 
|--------------------------|-------------|----------|----------|--------|--------|-------|----------------|---------|
|       TV Show 1          |     1       |    1     |   "what" | "what" | "ADV"  | "RB"  |      3         |  x      |
|       TV Show 1          |     1       |    2     |   "did"  | "do"   | "VERB" | "VBD" |      3         |  x      |
|       TV Show 1          |     1       |    3     |   "you"  | "you"  | "ADV"  | "PRN" |      3         |  x      |
|       TV Show 1          |     1       |    4     |   "say"  | "say"  | "VERB" | "VBD" |      3         |  x      |
|       Natural 1          |     1       |    1     |   "huh"  | "huh"  | "ADV"  | "RB"  |      1         |  x      |
|         ...              |    ...      |    ...   |    ...   |  ...   |  ...   |  ...  |      ...       |  ...    |


Data frame of 67 linguistic features

|  doc_id   |	f_01_past_tense |	f_02_perfect_aspect |	f_03_present_tense | f_04_place_adverbials | f_05_time_adverbials | f_... |
|-----------|-----------------|---------------------|--------------------|-----------------------|----------------------|-------|
| TV Show 1 |     double      |        double       |        double      |          double       |        double        | double|
| Natural 1 |     double      |        double       |        double      |          double       |        double        | double|
|    ...    |      ...        |        ...          |          ...       |          ...          |         ...          |  ...  |


#### Progress Report 3

GOAL:
For this project report, I will actually run the MDA, create several graphs to interpret the output, and identify co-occuring features which suggest "dramticity"

Below is a pipeline for the steps needed to finish running the MDA

Use `mda.loadings()` to get a factor summary (dependent on the # of factors identified as needed) --> use `arrange()` to put these in descending order 

Analysis:

- use stickplot, heatmap, and boxplot to analyze results 

- based on past literature and Biber 1988 try to highlight features that suggest "dramaticity"

## Concluding Remarks

The big hiccups I foresee are 1) the accuracy of the spacy tagging particularly given odd discourse features (might be the biggest problem in the natural discourse) 
2) complications in webscraping which is something I haven't done before and seems quite complex.
Aside from these concerns, another aspect of this project which is subject to change is how I code the TV show data throughout the analysis (hinted at in the optional third research question). 
It would be possible to create discrete categorical variables within the TV show transcripts by giving them a label like 'genre'. 
This would allow for an analysis that compares how 'dramaticity' may differ depending on the specific genre of TV show. 
Most likely, this is beyond the scope of the study, but it is something I want to keep in mind. 
Lastly, a small issue will be in determining the linguistic variables which represent 'dramaticity'.
As of now, I have been vague about how that process will work, and it is something I am hoping more reading of the literature and a more comprehensive understanding of MDA will help with.


Necessary Citations:

Biber 1988

Santa Barabara Corpus

Scraps from the Loft

pybiber script

mda.biber package