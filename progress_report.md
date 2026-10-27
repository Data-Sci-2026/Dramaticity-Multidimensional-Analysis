# Progress Reports

## Progress report 1

At this point in the project, I have collected the data I want from both the Santa Barbara Corpus and the Scraps from the Loft website as detailed in my [project_plan](project_plan.md).
As planned, I have created a pipeline for each set of transcripts which cleans up the files, formats each transcript's text as a single cell, and combines all of them into a single data frame. 
It is easiest to explain the progress for each source individually. 

## Santa Barbara Pipeline

For the [Santa Barbara pipeline](santa_barbara_pipeline.md), I downloaded a zip file of all 60 trn files. 
When a file was read in as a data frame, it had each line in a different cell. 
I wrote a function, `clean()`, that would go through each line and remove/transform transcription symbols. 
Then I placed that within a larger function, `get_clean_wrds()`, that would also remove white space, empty lines, and flatten the data frame into a single row.
Finally, I mapped `get_clean_wrds()` over all 60 files and used `rbind()` to form a single data frame where each observation is a file with `id` and `clean_words` as the two columns. 

## Scraps from the Loft pipeline

For the [Scraps from the Loft pipeline](scraps_from_loft_pipeline.md), I used `rvest` to read in the html code and specify only the elements that held text. 
Then, I wrote a function called `clean()` that could go through each line and remove some unwanted formatting. 
As before, I then placed that within a larger function, `get_clean_script()`, that would again flatten the text of each transcript into a single cell. 
As an added step for this pipeline, I had to decide on which TV shows to include. 
I decided to try to divide the TV shows as evenly as possible between comedy and drama genres.
Ultimately, after randomly selecting from shows which fit the criteria described in my [project_plan](project_plan.md), I had 27 comedies and 33 dramas. 
Finally, I mapped `get_clean_script()` over all of the html links to those TV show transcripts and got a single data frame where each observation is a file with `id` and `clean_words` as the two columns.

## One Big Data Frame

As a final step, I combined the two data frames from these pipelines into a single df called `all_transcripts`. 
The end result is a data frame that has a name for each file in one column (called id) and the text for that particular file written in a way that can be parsed for the next step of the project. 

## Next Steps

The next step will be running this large data frame through the python pipeline described in the [project_plan](project_plan.md).
One step will be to strip the id section down so that it only tags for "transcript" (Scraps from the Loft) and "trn" (Santa Barbara). 
At this point, I will also need to do some more thorough decision making about the multidimensional analysis.
In particular, I need to decide if I am only comparing natural discourse to TV show discourse or if I am making multiple levels within TV show discourse. 



