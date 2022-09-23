# Subreddit Classifier with Webscraping, NLP and ML

 - [Problem Statement](#Problem-Statement)
 - [Data Sources](#Data-Sources)
 - [Executive Summary](#Executive-Summary)
 - [Notebook Contents](#Notebook-Contents)
 - [Data Dictionary](#Data-Dictionary)
 - [Conclusion & Recommendations](#Conclusion-&-Recommendations)
 

## Problem Statement
In this project, we will be using webscraping, APIs, Natural Language Processing (NLP) and classfication modelling to classify the subreddit posts r/bicycling and r/motorcycles.

We will follow the data science process to answer the classification problem.
1. Define the problem
2. Gather & clean the data
3. Explore the data
4. Model the data
5. Evaluate the model
6. Answer the problem

--- 
## Data Sources
The sources of the data will be from Kaggle.

Main dataset

These test results are organized in such a way that when the number of mosquitos exceed 50, they are split into another record (another row in the dataset), such that the number of mosquitos are capped at 50. 

The location of the traps are described by the block number and street name. For your convenience, we have mapped these attributes into Longitude and Latitude in the dataset. Please note that these are derived locations. For example, Block=79, and Street= "W FOSTER AVE" gives us an approximate address of "7900 W FOSTER AVE, Chicago, IL", which translates to (41.974089,-87.824812) on the map.

Some traps are "satellite traps". These are traps that are set up near (usually within 6 blocks) an established trap to enhance surveillance efforts. Satellite traps are postfixed with letters. For example, T220A is a satellite trap to T220. 

Please note that not all the locations are tested at all times. Also, records exist only when a particular species of mosquitos is found at a certain trap at a certain time. In the test set, we ask you for all combinations/permutations of possible predictions and are only scoring the observed ones.

Spray Data

The City of Chicago also does spraying to kill mosquitos. You are given the GIS data for their spray efforts in 2011 and 2013. Spraying can reduce the number of mosquitos in the area, and therefore might eliminate the appearance of West Nile virus. 

Weather Data

It is believed that hot and dry conditions are more favorable for West Nile virus than cold and wet. We provide you with the dataset from NOAA of the weather conditions of 2007 to 2014, during the months of the tests. 

Station 1: CHICAGO O'HARE INTERNATIONAL AIRPORT Lat: 41.995 Lon: -87.933 Elev: 662 ft. above sea level
Station 2: CHICAGO MIDWAY INTL ARPT Lat: 41.786 Lon: -87.752 Elev: 612 ft. above sea level

Map Data

The map files mapdata_copyright_openstreetmap_contributors.rds and mapdata_copyright_openstreetmap_contributors.txt are from Open Streetmap and are primarily provided for use in visualizations (but you are allowed to use them in your models if you wish).

---
## Executive Summary
**INTRODUCTION**

Every year from late-May to early-October, public health workers in Chicago setup mosquito traps scattered across the city. Every week from Monday through Wednesday, these traps collect mosquitos, and the mosquitos are tested for the presence of West Nile virus before the end of the week. The test results include the number of mosquitos, the mosquitos species, and whether or not West Nile virus is present in the cohort. 

**METHODOLOGY**

The work was done in 3 separate notebooks. The first one focused on webscraping and getting the relevant data from Reddit, before some initial cleaning was done to ensure the quality of the data. The second notebook focused on preprocessing and EDA by using NLP to make sense of the words from the text. The third notebook focused on building and evaluating the models for the problem.

Firstly, data gathering and initial cleaning was performed with one or more of these steps
- Webscraping
- Reading and displaying datasets
- Combining necessary text data together
- Remove null values
- Dropping duplicates
- Feature engineering

Further preprocessing and EDA was then done after the initial cleaning to get a final dataset
- Create stop word list
- Create stemmed words using Porter Stemmer
- Create lemmatized words using WordNetLemmatizer
- Draw WordCloud to visualise
- Explore word and character length of posts
- Finding most common words in the original text
- Using n-grams to find common phrases

In the last notebook, train-test split was done on the final data set. The main text, stemmed text and lemmatized text were each passed into the 4 models: CountVectorizer with Multinomial Naive Bayes, TfidfVectorizer with Multinomial Naive Bayes, CountVectorizer with Random Forest Classifier and TfidfVectorizer with Random Forest Classifier. The performance of the models were then compared against each other using various metrics discussed below.

**SIGNIFICANT FINDINGS**

The models were tuned using RandomizedSearchCV to save time to get a good working model. Only results from the models with the stemmed text are shown in the table below.

|              | train_score | test_score | generalisation | precision | f1_score | roc_auc_score |
|-------------:|------------:|-----------:|---------------:|----------:|---------:|--------------:|
| cvec_nb_stem |      0.9270 |     0.8773 |         5.3614 |    0.8467 |   0.8712 |        0.9472 |
| tvec_nb_stem |      0.9410 |     0.8773 |         6.7694 |    0.8568 |   0.8694 |        0.9503 |
| cvec_rf_stem |      0.9968 |     0.8442 |        15.3090 |    0.8365 |   0.8303 |        0.9176 |
| tvec_rf_stem |      0.9957 |     0.8529 |        14.3417 |    0.8569 |   0.8374 |        0.9199 |

We will select the **CountVectorizer with the Multinomial Naive Bayes model** as it has good generalisation, accuracy, precision, f1 scores and ROC-AUC scores, compared to the other models. We will apply this model after Porter Stemming, whose overall performance is better than when on its base form or after lemmatizing.

The Random Forest models show signs of overfitting and further work would have to be done to optimise and tune the hyperparameters. The accuracy, precision and f1 scores are also not as high as when done with the Naive Bayes models.


### Notebook Contents
1. [Problem Statement](#Problem-Statement)
2. [Import Libraries](#Import-Libraries)
3. [Data Dictionary](#Data-Dictionary)
4. [Exploratory Data Analysis](#Exploratory-Data-Analysis)
5. [Get Dummies on Categorical columns](#Get-Dummies)
6. [Model Preparation and Fitting](#Model-Preparation)
---

## Data Dictionary

The data comes from Kaggle.

| Feature   | Type   | Description                                                                                                                                                  |
|-----------|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| subreddit | object | The subreddit where the post was taken from. This is either 'bicycling' or 'motorcycles'.                                                                    |
| text      | object | This is the combined text that was extracted directly from the subreddit's post selftext and title columns. Duplicates and special texts have been removed.  |
| stem_text | object | This column contains the Porter Stemmed text sentences that have been stemmed from the original clean tokens.                                                |
| lem_text  | object | This column contains the lemmatized text sentences that have been lemmatized from the original clean tokens.                                                 |
| bike      | int64  | This is a binary column that indicates which subreddit the post belongs to. 0 for bicycling and 1 for motorcycles.                                           |


---
## Conclusion & Recommendations 
Overall, we are able to get quite a good working model with over 85% accuracy. This project helped me to walk through the steps of how to webscrap directly from the website, analyze the words from the texts, to applying the transformation and classifier models needed to generate a good accuracy score. Many overlapping concepts were used in this project, including using NLP tools, web APIs, classifier models and evaluation techniques like using the ROC-AUC curve and confusion matrix. However, if given more time and data to answer the problem, my recommendations would be to:

- There were still some spam posts left over, so more thorough cleaning would have to be done. Some posts also had weird string characters left despite one round of cleaning.
- Possibly test out on other models other than Naive Bayes and Random Forest.
- We could also explore new features within Reddit such as the upvotes and downvotes, as well as post comments.
- As this is a fairly balanced dataset, the metrics are easy to compare. In the future, if we were to encounter highly imbalanced dateset, the use of sensitivity and specificity would be of greater use. The ROC-AUC curve cannot be used and would have to be substituted with the Precision Recall curve.