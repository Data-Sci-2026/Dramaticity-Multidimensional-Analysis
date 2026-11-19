# Mutli-Dimensional Analysis

This document serves as an explanation of the fundamentals of Multi-Dimensional Analysis. 
Additionally, this is a place to discuss particularly important decisions related to the mda made throughout the process that will be necessary for any similar project. 

### Defining a Feature Loading

Feature loadings are the scores of particular features.
The size of the loading is equivalent to the size of correlation among other features, telling us how much these features occur together.
Therefore feature loadings close to 1 indicate a high degree of correlation and features close to -1 indicate inverse co-occurence (Biber, p. 80).

### Defining a Dimension

Dimensions are descriptions of text that are "continuums of variation" (p. 9)
For example, the present project is looking at the continuum of undramatic to dramatic. 
Any text could be placed on this continuum. 
A dimension can also be a linguistic feature, such as "verbal to nominal" (p. 12).
But for a multidimensional analysis, these smaller linguistic features are grouped together in order to define larger dimensions. 

The underlying idea here is that when linguistic features co-occur, that co-occurence is caused by some underlying dimension. 
If the number of pronouns in a document seems to co-occur with simple clauses, we could say that they are part of an underlying dimension of formal-informal for instance. (Biber, p. 11)

### Creating a Features List

For the present study, we are comparing two texts which should be relatively similar in terms of linguistic features.
TV show transcripts are meant to simulate real conversations.
They are both spoken language (as opposed to written) and they are both less likely to be particularly formal. 
However, there are significant differences between these two types of dialogue. 
For the present MDA, the difference focused on is dramaticity. 
In order to accomplish this, we need to decide on a list of linguistic features that might influence how 'dramatic' a text sounds. 
Fortunately for us, Biber has already come up with a very useful list of 67 features that defines multiple dimensions across English texts.


### Deciding on the number of dimensions

While much of the practical advice is 'baked in' to the packages we are using, it is still important to understand what is happening behind the curtain. 

A principal factor analysis goes through all of the factors and finds the largest possible grouping of shared variance.
A scree plot is a useful tool for visualizing this distribution. 
The plot displays eigenvalues, which are "indices of the amount of variance accounted for by each factor." (Biber, p. 82)
Basically, we want to create an ideal grouping of feature weights such that the most informative dimensions can be realized.
In creating too many dimensions (including 'scree'), we risk creating 'worthless' dimensions that do not say anything about the underlying texts.
Creating too few is considered even worse. 
Biber suggests a larger factor set if deciding between a smaller and larger set (Biber, p. 84). 
Once the factors are decided, factor rotation is performed. 
The packages used in this analysis automatically use a 'Promax' rotation, which is recommended when working with texts. 
Rotation simplifies the features present in a factor making each factor easier to interpret. 

### The Follow Up

Once the features that co-occur have been determined, that is simply the beginning. 
In order to justify arguing these features as evidence of "dramaticity" differences, we need to take a closer look.
As Biber says, "any interpretations need to be verified and refined by analysis of the co-occuring features in particular texts" (p. 19).
To that end, looking at both graphical displays *and* the samples of the features in question can be used to get a better grasp of the dimensions. 
The point of the dimension is that there is some underlying difference between the texts (Biber, p. 91). 
The trick is justifying these differences. 




