# Power Outage Analysis
Portfolio for EECS 398 @ Michigan

# Introduction

This project explores and analyzes a database maintained by the Laboratory for Advancing Sustainable Critical Infrastructure at Purdue containing major power outage data in the continental U.S. from January 2000 to July 2016. 

As a foundation of the U.S.'s infrastructure, power grid failures have the potential to severely disrupt the lives and livelihoods of Americans; in particular, the last five years have seen a number of major failures. In 2021, Texas saw unexpected winter storm systems that overwhelmed their poorly-kept grid, which, in conjunction with poor crisis management by lawmakers and power companies, ended up affecting more than 4.5 million customers. I'm particularly interested in this dataset as a native Californian; PG&E is notorious for their mishandling of line maintenance and outage response. Not only does California tend to experience more outages than most other states in the U.S., but downed PG&E lines have directly resulted in some of the most devastating wildfires in recent years. (This set of fires notably includes the Kincade Fire in Sonoma County and the Dixie Fire, which spanned more than 5 counties in Northern California and is California's second largest fire ever as of December 2024.)

Grid maintenance and outage responses are primarily the responsibility of private companies operated on a (largely) state-by-state basis, and I was therefore curious to explore the dataset and see how outages varied state-by-state and region-by-region. After cleaning the dataset, I first performed some exploratory data analysis to understand the primary locations and causes of the included outages. I then designed a model to predict INSERT MODEL STUFF.

The database contains 1534 rows of data (each row representing an outage) and 57 columns, but I primarily looked at the following columns for the purposes of this report.




# Data Cleaning and Exploratory Data Analysis

## Cleaning

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

## Univariate Analysis

<iframe
  src="assets/nerc-uni.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="assets/nerc-uni-state.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


## Bivariate Analysis

<iframe
  src="assets/nerc-biv.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>



## Interesting Aggregates
| **CLIMATE.REGION**          | **Most Customers Affected on Average** | **Most Customers Affected** | **Longest Outage on Average** | **Longest Overall Outages (Total Minutes)** |
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



## Imputation

# Framing a Prediction Problem


# Baseline Model

# Final Model