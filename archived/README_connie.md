# West Nile Virus Prediction - Predict West Nile virus in mosquitos across the city of Chicago

 - [Problem Statement](#Problem-Statement)
 - [Data Sources](#Data-Sources)
 - [Executive Summary](#Executive-Summary)
 - [Conclusion & Recommendations](#Conclusion-&-Recommendations)
 - [Notebook Contents](#Notebook-Contents)
 

## Problem Statement
In order to efficiently combat the West Nile Virus in Chicago we aim to :-

1. Build a model and make predictions that the city of Chicago can use about when and where when it decides to spray pesticides
2. Conduct a cost-benefit analysis that include annual cost projections for various levels of pesticide coverage (cost) and the effect of these various levels of pesticide coverage (benefit)


--- 
## Data Sources
The sources of the data will be from Kaggle.

Main dataset

These test results are organized in such a way that when the number of mosquitos exceed 50, they are split into another record (another row in the dataset), such that the number of mosquitos are capped at 50. 

The location of the traps are described by the block number and street name. The attributes are mapped into Longitude and Latitude in the dataset. Please note that these are derived locations. For example, Block=79, and Street= "W FOSTER AVE" gives us an approximate address of "7900 W FOSTER AVE, Chicago, IL", which translates to (41.974089,-87.824812) on the map.

Some traps are "satellite traps". These are traps that are set up near (usually within 6 blocks) an established trap to enhance surveillance efforts. Satellite traps are postfixed with letters. For example, T220A is a satellite trap to T220. 

Not all the locations are tested at all times. Also, records exist only when a particular species of mosquitos is found at a certain trap at a certain time. 

Spray Data

The City of Chicago also does spraying to kill mosquitos. We are given the GIS data for their spray efforts in 2011 and 2013. Spraying can reduce the number of mosquitos in the area, and therefore might eliminate the appearance of West Nile virus. 

Weather Data

It is believed that hot and dry conditions are more favorable for West Nile virus than cold and wet. We are provided with the dataset from NOAA of the weather conditions of 2007 to 2014, during the months of the tests. 

Station 1: CHICAGO O'HARE INTERNATIONAL AIRPORT Lat: 41.995 Lon: -87.933 Elev: 662 ft. above sea level
Station 2: CHICAGO MIDWAY INTL ARPT Lat: 41.786 Lon: -87.752 Elev: 612 ft. above sea level

Map Data

The map files mapdata_copyright_openstreetmap_contributors.rds and mapdata_copyright_openstreetmap_contributors.txt are from Open Streetmap and are primarily provided for use in visualizations.

---
## Executive Summary
**INTRODUCTION**

Every year from late-May to early-October, public health workers in Chicago setup mosquito traps scattered across the city. Every week from Monday through Wednesday, these traps collect mosquitos, and the mosquitos are tested for the presence of West Nile virus before the end of the week. The test results include the number of mosquitos, the mosquitos species, and whether or not West Nile virus is present in the cohort. 

**METHODOLOGY**

# Data Cleaning 
Train - dataset is for odd years 2007, 2009, 2011 and 2013, with 10,506 rows and 12 columns. 
Test - dataset is for even years 2008, 2010, 2012 and 2014, with 116,293 rows and 11 columns

Minimal cleaning is required for Train and Test set, as there are no null values in the dataframes. 'Date' column datatype is changed from object to datetime.

Spray - dataset is for years 2011 and 2013, with 2,944 rows and 4 columns

There are 584 null values in 'Time' column, with 541 duplicate rows. All null and duplicate values happened on 2011-09-07, and are dropped from the dataset. 'Date' column datatype is changed from object to datetime.

Weather - dataset is for every year from 2007 to 2013, with 13,710 rows and 22 columns
- There are no null values in all columns. 'Date' column datatype is changed from object to datetime. 
- There are some non-numeric values in columns which we have replaced with numeric data shown below :-
    'M' in Tavg replaced with average value of Tmax and Tmin
    'T' in PrecipTotal replaced with 0.005
    'M' in PrecipTotal replaced with 0
- There are no Sunrise and Sunset informaton for Station 2, hence the weather dataframe is split by stations, with sunrise and sunset values in station 1 copied over to station 2 based on Date.
- A new feature, Day_length is created to capture the number of daylight hours between Sunrise and sunset, and added to weather dataframe in station 1. This feature was subsequently copied over to station 2 based on Date as well.

# EDA
Preliminary check on train dataset shows that data is imbalanced as only 5% of entries are labelled as WnV present.

Trap Locations
There are 136 trap locations across Chicago, and 2 weather stations at the airports to capture weather data, Station 1: Chicago O'Hare International Airport and Station 2: Chicago Midway International Aiport.

Mosquitoes Species 
There are 7 unique species in the Train dataset, but only 3 species are found to spread the West Nile Virus
- Culex Pipiens/Restuans
- Culex Restuans
- Culex Pipiens
The species that do not spread the virus have low counts in the Train set, but high counts in the Test set

Seasonality Effects
WNV cases tend to see a sharp peak in August before dropping. The number of mosquitoes caught show a similar trend where there is a sharp peak before dropping. There is likely a time lag between mosquitoes caught and WNV cases, taking into account the lifecyle of mosquitoes of approximately 7-10 days. WNV cases coincides with the summer months of early June to end August, which also happens to be months where dewpoint, temperature and day length are at its highest / longest. July and August are summer and typically the season when heat and humidity provide the right condition for mosquitoes to breed.

Spray Data
There are a total of 9 spray dates in dataset, 1 in 2011 and 8 in 2013. Spray data seems to be captured once in every 10 seconds resulting in > 2,000 rows of data for only 9 dates. Not all hotspots or locations with mosquitoes are being sprayed which may indicate spraying is being done without prior research.

Weather Data
In the correlation table, temperature and dewpoint seems to have a high and positive correlation predicting the presence of the WNV, while the resultant wind speed and day length are negatively correlated.

The amount of precipitation seems to be fluctuating and does not follow a specific seasonal trend. At first glance, it seemed that June or July are the wettest months, but across years the rainfall can vary greatly month to month. In 2009 Oct, the precipitation looks to be greater than usual, but the entry is only based on one single day. More data points are needed to do a more accurate weather analysis.

# Feature Engineering
Species:
All species which do not carry the West Nile Virus will be regrouped to 'OTHERS'
Convert the categorical feature of Species into dummy variables

Traps:
Cluster all traps based on their locations to simplify analysis (we will try with 30 clusters), using K Means clustering
Convert this categorical feature of Trap Clusters into dummy variables
Clustering was derived on Train dataset and subsequently used to predict clustering on Test

Weather data:
Create new column "Station" to indicate which weather station a trap is closer to. Weather data are then assigned to each trap location after clusting of traps are done.
Weather features are further transformed with: Shifting feature forward for period of 6 days, Looking back 12 days and taking the mean, Looking back 14 days and taking the max
Once transformation is done, add those weather features to Train set

After Feature Engineering, dataset would have a total of 86 features, consisting of:
- Time features (Date, year, month, week etc)
- Location features (Address, Block, Street, Longitude, Latitude etc)
- Mosquito Species (3 dummy variables)
- 30 trap clusters
- 9 weather features
- 27 (9x3) transformed weather features


# Modelling
Handling Imbalanced Data
Using Over/under sampling and SMOTE

Model Comparison and Evaluation
Train data is highly imbalanced, with only 5% of the Train data being positive (Presence of WNV), two metrics will be used: Precision-Recall AUC Score and F1 Score.

**MODEL EVALUATION**
7 models are used below :

| Models                    | PR-AUC_Train | PR_Auc_Test | Generalization | F1_train | F1_test | Generalization  |
|---------------------------|--------------|-------------|----------------|----------|---------|-----------------|
| Baseline Model            | 0.05         | 0.05        | 0.27           | 0.1      | 0.1     | 0.25            |
| O/S + U/S + GradientBoost | 0.28         | 0.25        | 9.33           | 0.34     | 0.28    | 18.95           |
| O/S + U/S + RandomForest  | 0.21         | 0.2         | 4.59           | 0.17     | 0.16    | 5.62            |
| O/S + U/S + LightGBM      | 0.28         | 0.26        | 4.67           | 0.32     | 0.29    | 9.15            |
| Smote + GradientBoost     | 0.3          | 0.26        | 11.83          | 0.24     | 0.27    | 10.82           |
| Smote + RandomForest      | 0.24         | 0.22        | 6.71           | 0.24     | 0.24    | 0.77            |
| Smote + LightGBM          | 0.33         | 0.31        | 6.67           | 0.31     | 0.32    | 0.96            |

We will select the **Smote + LightGBM** as it has good generalisation, f1 and ROC-AUC scores, compared to the other models. 


# Cost Benefit Analysis

| Inaccuracy Costs                                           |                                                                   |
|------------------------------------------------------------|-------------------------------------------------------------------|
| Impact of False Positive indication of West Nile Virus     | Impact of False Negative indication of West Nile Virus            |
| 1. Unnecessary Spraying                                    | 1. Increased proliferation of West Nile Virus disease             |
| 2. Loss of Productivity of Civil Servants                  | 2. Increased strain on health care resources due to rise in cases |
| 3. Causes disruption to daily life in affected communities | 3. Public Health reputational and political risk                  |
| 4. Increased burden on taxpayers                           |

The costs of spraying are a fraction of the Medical and Productivity costs (not to mention the lives lost), which makes the effort well worth the financial investment. Usage of our model would assist in a more target usage of pesticide spray which could also further reduce costs. Money saved for the taxpayer could engender more fiscal confidence in public health system.


---
## Conclusion 

Based on the cost benefit analysis, we recommend to spray and we will further deepdive to spray data analysis below.

Findings from Spray Data
1) Spraying is done in an ad-hoc manner
Data from 2011 and 2013 seems to suggest that it was done without prior research For e.g. in 16 Aug and 22 Aug 2013, spraying was not done on WNV hotspot areas or areas where trap locations are found

2) Spray not effective with time
Number of mosquitoes did not drop within spraying area. Effectiveness of spraying seemed to reduce later on in the months, perhaps due to mosquitoes developing resistance to pesticides over time

3) Spraying not effective in curbing virus
WNV hotspots still remain 2 weeks after spraying Assuming adulticide sprays are applied, which only kills adult mosquitoes, it is not truly effective in reducing virus as mosquito larvae is still alive.

## Considerations & Recommendations
WNV is more prevalent under certain conditions, such as longer daylight hours and higher average temperatures.

Spraying efforts should be focused during June to early July; current spraying efforts are ineffective. Suggest to spray in early June to July, considering the gestation period of mosquitoes resulting in peak WNV cases in August

There may be health issues related to spray chemicals, as pregnant women and children have a greater risk of getting sick from pesticides.

Lastly, we should consider alternatives to spraying, such as larviciding catch basins, which involves dropping tablets in storm drains along the public roads that slowly dissolve over a five-month period to prevent mosquito larvae from hatching, and eliminating standing water by ensuring that swimming pools and construction sites are regularly maintained.



### Notebook Contents
1. [01_data_cleaning_EDA] 
2. [02_feature_engineering]
3. [03_modelling]
4. [04_spray_analysis]
---
