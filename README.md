# Analyzing Power Outages: A Regional Exploration
Portfolio for EECS 398 @ Michigan
By Maya Segal (masegal@umich.edu)

## Introduction

This project explores and analyzes a database maintained by the Laboratory for Advancing Sustainable Critical Infrastructure at Purdue containing major power outage data in the continental U.S. from January 2000 to July 2016. 

As a foundation of the U.S.'s infrastructure, power grid failures have the potential to severely disrupt the lives and livelihoods of Americans; in particular, the last five years have seen a number of major failures. In 2021, Texas saw unexpected winter storm systems that overwhelmed their poorly-kept grid, which, in conjunction with poor crisis management by lawmakers and power companies, ended up affecting more than 4.5 million customers. I'm particularly interested in this dataset as a native Californian; PG&E is notorious for their mishandling of line maintenance and outage response. Not only does California tend to experience more outages than most other states in the U.S., but downed PG&E lines have directly resulted in some of the most devastating wildfires in recent years. (This set of fires notably includes the Kincade Fire in Sonoma County and the Dixie Fire, which spanned more than 5 counties in Northern California and is California's second largest fire ever as of December 2024.)

Grid maintenance and outage responses are primarily the responsibility of private companies operated on a (largely) state-by-state basis, and I was therefore curious to explore the dataset and see how outages varied state-by-state and region-by-region.  Specifically, I wanted to explore how outages are handled by each of the different commissions in NERC, which includes looking at the number of customers affected by outages by region, the length of outages in each region, and what's causing outages in each region (i.e. what could the maintenance commissions/companies be better looking out for?)After cleaning the dataset, I first performed some exploratory data analysis to understand the primary locations and causes of the included outages. I then designed a model to predict the cause category of an outage based on some key indicators (region, duration, year, customers affected, etc.).

The database contains 1534 rows of data (each row representing an outage) and 57 columns, but I primarily looked at the following columns for the purposes of this report.

|Column                |Description|
|---                |---        |
|`'YEAR'`                |Outage year|
|`'MONTH'`                |Outage month|
|`'U.S._STATE'`                |State in which the outage occurred|
|`'NERC.REGION'`                |North American Electric Reliability Corporation region of the outage|
|`'CLIMATE.REGION'`                |U.S. Climate regions as specified by National Centers for Environmental Information|
|`'ANOMALY.LEVEL'`                |Oceanic El Niño/La Niña (ONI) index|
|`'OUTAGE.START.DATE'`                |Day of the year when the outage event started|
|`'OUTAGE.START.TIME'`                |Time of the day when the outage event started|
|`'OUTAGE.RESTORATION.DATE'`                |Day of the year when power was restored to all the customers|
|`'OUTAGE.RESTORATION.TIME'`                |Time of the day when power was restored to all the customers|
|`'CAUSE.CATEGORY'`                |Categories of all the events causing the major power outages|
|`'OUTAGE.DURATION'`                |Duration of outage events (in minutes)|
|`'CUSTOMERS.AFFECTED'`                |Number of customers affected by the power outage event|
|`'POPDEN_URBAN'`                |Population density of the urban areas (in persons-per-square-mile)|


## Data Cleaning and Exploratory Data Analysis

### Cleaning

1. I began by dropping all rows/columns of overhead — this includes the first row of the dataframe, which specified the units of each column rather than containing data, as well as the first column, which imported junk values as a result of the read into pandas.
2. I created a singular start time and singular restoration time for every outage by combining the date and time for both the start and restoration, `OUTAGE.START.DATE` + `OUTAGE.START.TIME` and `OUTAGE.RESTORATION.DATE` + `OUTAGE.RESTORATION.TIME`, and casting it with pd.to_datetime to `OUTAGE.START.DT` and `OUTAGE.RESTORATION.DT`. I also created an `OUTAGE.LENGTH` column, which mirrors the `OUTAGE.DURATION` column but lives as a time object rather than numeric value of minutes.
3. I converted some columns into number types from strings in order to use them for future calculations and sorting operations.
4. I dropped columns I didn't want to focus on for this exploration (largely numbers relating to the urban/rural make-up of areas, as well as price and customer information).

The head of the dataframe is displayed below (I'll note that there's no order to the dataframe at this point — further sorting, aggregating, etc. can be found below).

| YEAR | U.S._STATE | NERC.REGION | CLIMATE.REGION     | ANOMALY.LEVEL | CAUSE.CATEGORY     | OUTAGE.DURATION | CUSTOMERS.AFFECTED | POPDEN_URBAN | OUTAGE.START.DT      | OUTAGE.RESTORATION.DT | OUTAGE.LENGTH     |
|------|------------|-------------|--------------------|---------------|--------------------|-----------------|---------------------|--------------|----------------------|-----------------------|-------------------|
| 2011 | Minnesota  | MRO         | East North Central | -0.3          | severe weather     | 3060.0          | 70000.0             | 2279.0       | 2011-07-01 17:00:00  | 2011-07-03 20:00:00   | 2 days 03:00:00   |
| 2014 | Minnesota  | MRO         | East North Central | -0.1          | intentional attack | 1.0             | NaN                 | 2279.0       | 2014-05-11 18:38:00  | 2014-05-11 18:39:00   | 0 days 00:01:00   |
| 2010 | Minnesota  | MRO         | East North Central | -1.5          | severe weather     | 3000.0          | 70000.0             | 2279.0       | 2010-10-26 20:00:00  | 2010-10-28 22:00:00   | 2 days 02:00:00   |
| 2012 | Minnesota  | MRO         | East North Central | -0.1          | severe weather     | 2550.0          | 68200.0             | 2279.0       | 2012-06-19 04:30:00  | 2012-06-20 23:00:00   | 1 days 18:30:00   |
| 2015 | Minnesota  | MRO         | East North Central | 1.2           | severe weather     | 1740.0          | 250000.0            | 2279.0       | 2015-07-18 02:00:00  | 2015-07-19 07:00:00   | 1 days 05:00:00   |

### Univariate Analysis

To start, I looked at the distribution of a handful of the given columns. Below, I wanted to take a look at the overall number of outages per NERC region — NERC regions represent the division of national grid oversight, so I wanted to see if there were any regions with particularly high numbers. It turns out that the Western Electricity Coordinating Council and ReliabilityFirst, which represent the Western U.S. and Midwest respectively, saw the most outages.

<iframe
  src="assets/nerc-uni.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Out of curiosity, I looked at the same distribution but with states as the divisor rather than NERC regions to see if there were any standouts. It certainly makes sense based on size that California and Texas have outrageously high representation when it comes to number of outages, but it's also worth thinking about if those outages are disproportionate to their size — just by paying attention to current events, it's no secret that California and Texas specifically tend to struggle with their power grids. It's also interesting to note here that, as indicated by the color distribution, representing the NERC regions (one state can be covered by multiple NERC regions), Texas is almost entirely covered by their own NERC commission: TRE. Among the larger states in the U.S., Texas and Florida appear to be among very few states to have their own NERC regions.

<iframe
  src="assets/nerc-uni-state.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


### Bivariate Analysis

While there are a few more bivariate analyses in the notebook, I wanted to take a more detailed look at the univariate distributions by NERC region shown above. The distribution below visualizes the total number of customers affected by outages in each NERC region. The x axis is still laid out in descending order (left to right) of number of outages, but the bars represent customers affected, where each block within a bar is the block of customers affected in a single specific outage. There are a number of interesting observations to be made here, one of which is that the RFC seems to have many power outages that don't affect very varying numbers of customers, whereas regions like the TRE, NPCC, and FRCC appear to have had one or a couple of major outages that disproportionately contribute to their total number of affected customers.

<iframe
  src="assets/nerc-biv.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>



### Interesting Aggregates

The pivot table below highlights, for each climate region, what the cause was behind some of the most significant outages in the dataset, where significance is measured by the most customers affected by one outage and the longest single outage. Additionally, the table shows what cause category, on average, is responsible for the most affected customers and longest outages.

| **Climate Region** | **Most Customers Affected on Average** | **Most Customers Affected** | **Longest Outage on Average** | **Longest Overall Outages (Total Minutes)** |
|----------------------------|---------------------------------|--------------------------------|----------------------------|---------------------------|
| Central                    | system operability disruption   | severe weather                 | fuel supply emergency      | severe weather            |
| East North Central         | system operability disruption   | severe weather                 | fuel supply emergency      | severe weather            |
| Northeast                  | system operability disruption   | severe weather                 | fuel supply emergency      | severe weather            |
| Northwest                  | severe weather                  | severe weather                 | severe weather             | severe weather            |
| South                      | system operability disruption   | severe weather                 | fuel supply emergency      | severe weather            |
| Southeast                  | severe weather                  | severe weather                 | public appeal              | severe weather            |
| Southwest                  | system operability disruption   | system operability disruption  | severe weather             | severe weather            |
| West                       | severe weather                  | severe weather                 | fuel supply emergency      | severe weather            |
| West North Central         | severe weather                  | severe weather                 | severe weather             | severe weather            |



### Imputation

As I approached the prediction problem, I had to impute a few columns that I wanted to use for my model. I imputed the following columns:

`CUSTOMERS.AFFECTED`: To impute the number of affected customers, I decided to group the dataset first by the US State and then by cause category, then taking the mean of the resulting groups and filling NaNs with that value. I did this to account for the nearest outage scenario — while NERC regions effectively cover large areas of the U.S., states more accurately encapsulate the population, immediate response, density, etc. for an outage. 

<iframe
  src="assets/ca-imp.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

`OUTAGE.DURATION`: To impute the outage duration, I grouped once again by the US State and then by cause category (this is a particularly good indicator for duration, as storms will take longer than quick maintenance/system failure issues!), but then took the median of the group, as there were significant outliers in dataset in spite of an otherwise very tight distribution, and I didn't want the outliers greatly affecting the imputed values.

<iframe
  src="assets/od-imp.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

`ANOMALY.LEVEL`: As anomaly level is an indicator of climate, I grouped by climate region and cause category (cause category was necessary here to make sure I wasn't falsely imputing based on climate when the outage had nothing to do with climatic factors) and imputed with the mean of the group.

<iframe
  src="assets/al-imp.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


## Framing a Prediction Problem
My goal was to predict the cause category of a given outage based on a handful of information that might be known at the time of an outage, either during or after the outage — this is a multiclass classification problem. During the outage, it would obviously be beneficial to know the cause — it would be much easier to solve the problem given a known cause. However, it could also be beneficial to understand how to identify a specific cause based on information obtained after an outage, as the cause might not be clear at the time of the outage, knowing what broke the grid in hindsight could lead to better, more specific fixes. Therefore, at the time of prediction, I assume that we have access to metrics including but not limited to customers affected and outage duration. While there are a number of available metrics in the notebook classification report, I mainly looked at the F1 score to measure the efficacy of the model, as the variety of causes isn't quite balanced and both false negatives and positives are bear the same weight, making it a good candidate for F1 scores.

## Baseline Model
For the baseline model, I used a Random Forest Classifier and the following features to predict the `CAUSE.CATEGORY`.


- **`NERC.REGION`** (nominal): What region of oversight does the outage fall under? What other outages were the responsibility of this commission?

- **`CUSTOMERS.AFFECTED`** (quantitative): How many customers were affected by the outage?


For the nominal feature, I used sklearn's OneHotEncoder, and for the numeric feature, I used StandardScaler. While I just expected this to be a jumping off point for the model, it actually performed decently well, resulting in a weighted average of a 0.67 F1 score and 0.75 precision score. I think the baseline model wasn't bad, but could certainly be better — it takes in two of the more significant indicators of the nature of an outage: where it is and how many people it affected. (You could probably guess the cause of an outage based on these features with decent accuracy — a large affected customer population in Florida is more likely to be severe weather, whereas a small affected customer population in Pennsylvania is more likely to be an equipment failure.) That said, we have more features at our disposal that appear to be relevant to the cause category, and I therefore explored more with the final model.


## Final Model
My final model included the following features to predict the `CAUSE.CATEGORY` (I also indicate the unknowns each feature serves to fill in/how they contribute to the model's accuracy):

- **`NERC.REGION`** (nominal): What region of oversight does the outage fall under? What other outages were the responsibility of this commission?
  
- **`CUSTOMERS.AFFECTED`** (quantitative): How many customers were affected by the outage?

- **`YEAR`** (ordinal): When was the outage? *Note that year is a tricky variable because it does represent (hopefully) a sort of measurable progression factor for the grid's performance despite being ordinal.*

- **`ANOMALY.LEVEL`** (quantitative): How rare/unexpected was the outage?

- **`POPDEN_URBAN`** (quantitative): How dense is the population of affected customers?

- **`OUTAGE.DURATION`** (quantitative): How long did the outage last?



I chose to include these additional features because, as seen by the questions they each answer, they all provide further information that could hint at the reason for a given outage. I again used a Random Forest Classifier. For preprocessing, I used PolynomialFeatures for the population density and affected customers variables, hoping to capture any interaction between the two. I used the StandardScaler for the remaining numeric features (`CUSTOMERS.AFFECTED`, `YEAR`, `ANOMALY.LEVEL`, `POPDEN_URBAN`, 'OUTAGE.DURATION`) so as to standardize them all and prevent bias as a result of their varying magnitudes. I used a OneHotEncoder once again on `NERC.REGION`.

I used GridSearchCV to tune hyperparameters, and found that the best parameters for the classifier include:

- **min_samples_split**: 4 (we want to balance how often we split the trees to avoid under/overfitting)
- **max_depth**: 10 (to ensure the depth doesn't result in under or overfitting the data)
- **n_estimators**: 31 (finding the sweet spot of number of trees to aggregate)
- **class_weight**: 'balanced' (will balancing the dataset improve performance (instead of None)?)

To assess the performance of this model, we can look to the F1 score of 0.84 (and a precision of 0.86). Based on the F1 score, we can see a significant improvement from the baseline model, indicating the final model is more successful.