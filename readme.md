# Comparison of parametric and non-parametric models in predicting churn
## John Herr
## Joe Tustin
## Carly Wolfbrandt

### Table of Contents
1. [Objective](#objective)
2. [Exploratory Data Analysis](#eda)
    1. [Dataset](#dataset) 
    2. [Data Cleaning](#cleaning)
    3. [Feature Engineering](#engineering)
3. [Modelling](#model)
    1. [Model Pipeline](#pipeline)
    2. [Model Scoring](#scoring)
    3. [Random Forest](#rf)
    4. [Linear Regression](#lm)

## Objective <a name="objective"></a>

Use rideshare data set to help understand what factors are the best predictors for churn, and offer insights to help improve customer retention.

## Exploratory Data Analysis <a name="eda"></a>

### Dataset <a name="dataset"></a>

A ride-sharing company (Company X) is interested in predicting rider retention. To help explore this question, we used a sample dataset of a cohort of users who signed up for an account in January 2014. The data was pulled on July 1, 2014; we consider a user retained if they were “active” (i.e. took a trip) in the preceding 30 days (from the day the data was pulled). In other words, a user is "active" if they have taken a trip since June 1, 2014.

Here is a detailed description of the data:

***CATEGORICAL***
- `city`: city this user signed up in 
- `phone`: primary device for this user

***NUMERICAL***
- `signup_date`: date of account registration; in the form `YYYYMMDD`
- `last_trip_date`: the last time this user completed a trip; in the form `YYYYMMDD`
- `avg_dist`: the average distance (in miles) per trip taken in the first 30 days after signup
- `avg_rating_by_driver`: the rider’s average rating by their drivers over all of their trips 
- `avg_rating_of_driver`: the rider’s average rating of their drivers over all of their trips 
- `surge_pct`: the percent of trips taken with surge multiplier > 1 
- `avg_surge`: The average surge multiplier over all of this user’s trips 
- `trips_in_first_30_days`: the number of trips this user took in the first 30 days after signing up 
- `weekday_pct`: the percent of the user’s trips occurring during a weekday
- `luxury_car_user`: TRUE if the user took a luxury car in their first 30 days; FALSE otherwise 

**Table 1**: Initial dataset 

|    |   avg_dist |   avg_rating_by_driver |   avg_rating_of_driver |   avg_surge | city           | last_trip_date      | phone   | signup_date         |   surge_pct |   trips_in_first_30_days | luxury_car_user   |   weekday_pct |
|---:|-----------:|-----------------------:|-----------------------:|------------:|:---------------|:--------------------|:--------|:--------------------|------------:|-------------------------:|:------------------|--------------:|
|  0 |       3.67 |                    5   |                    4.7 |        1.1  | King's Landing | 2014-06-17 00:00:00 | iPhone  | 2014-01-25 00:00:00 |        15.4 |                        4 | True              |          46.2 |
|  1 |       8.26 |                    5   |                    5   |        1    | Astapor        | 2014-05-05 00:00:00 | Android | 2014-01-29 00:00:00 |         0   |                        0 | False             |          50   |
|  2 |       0.77 |                    5   |                    4.3 |        1    | Astapor        | 2014-01-07 00:00:00 | iPhone  | 2014-01-06 00:00:00 |         0   |                        3 | False             |         100   |
|  3 |       2.36 |                    4.9 |                    4.6 |        1.14 | King's Landing | 2014-06-29 00:00:00 | iPhone  | 2014-01-10 00:00:00 |        20   |                        9 | True              |          80   |
|  5 |      10.56 |                    5   |                    3.5 |        1    | Winterfell     | 2014-06-06 00:00:00 | iPhone  | 2014-01-09 00:00:00 |         0   |                        2 | True              |         100   |

### Data Cleaning <a name="cleaning"></a>

Table 2 shows the data types and number of null values for each column.

**Table 2**: Initial data type and null value descriptions 

 |   column name |   information | 
 |---:|-----------:|
|avg_dist  |                50000 non-null float64 |
|avg_rating_by_driver   |   49799 non-null float64|
|avg_rating_of_driver  |    41878 non-null float64|
|avg_surge    |             50000 non-null float64|
|city    |                  50000 non-null object|
|last_trip_date    |        50000 non-null object|
|phone       |              49604 non-null object|
|signup_date    |           50000 non-null object|
|surge_pct      |           50000 non-null float64|
|trips_in_first_30_days   | 50000 non-null int64|
|luxury_car_user |          50000 non-null bool|
|weekday_pct    |           50000 non-null float64|

There are 3 columns with null values, `avg_rating_by_driver`, `avg_rating_of_driver` and `phone`. These will need to be dealt with alongisde the incorrectly typed `signup_date` and `last_trip_date` columns.  Furthermore, the 2 categorical values, `city` and `phone` need to be converted to dummy variables.

Figure 1 shows the correlation matrix for the features. 

![](images/correlation_matrix.png)

**Figure 1**: Correlation matrix for the features in the dataset

The only features that seem strongly correlated are `surge_pct` and `avg_surge`. We decided to drop `avg_surge` since the `surge_pct` represents the percent of trips taken with surge multiplier > 1, which would be more indicative of someone who is paying more (and thus probably unhappy) than the `avg_surge` which is the average surge multiplier over all of the user’s trips.


**Table 3**: Cleaned data type and null value descriptions 

 |   column name |   information | 
 |---:|-----------:|
|avg_dist                |  50000 non-null float64|
|avg_rating_by_driver    |  50000 non-null float64|
|avg_rating_of_driver     | 50000 non-null float64|
|avg_surge                 |50000 non-null float64|
|last_trip_date      |      50000 non-null datetime64[ns]|
|signup_date          |     50000 non-null datetime64[ns]|
|surge_pct             |    50000 non-null float64|
|trips_in_first_30_days |   50000 non-null int64|
|luxury_car_user    |       50000 non-null bool|
|weekday_pct         |      50000 non-null float64|
|city_King's Landing  |     50000 non-null uint8|
|city_Winterfell     |      50000 non-null uint8|
|phone_Android        |     50000 non-null uint8|
|phone_iPhone          |    50000 non-null uint8|

### Data Cleaning <a name="cleaning"></a>

In looking at the data, the `signup_date` appears to be a key feature. The longer a user has been active, the more likely they are to continue being an active user. Also, users who use the luxury car service (`luxury_car_user` = True), have expendable income and are more likely to use a car service more often. The `avg_rating_of_driver` field is also important, as users who are consistently rating their drivers highly, are probably more likely to be happy with the product. Furthermore, our 2 categorical features are probably also correlated with whether or not a user churns. The `city` a user lives in, whether it is denser or more spread out, the service has good coverage or bad, etc. could be correlated to their happiness with the product. Also, the `phone` feature is probably useful, however this feature is dominated by iPhone users (by 3x) therefore, there could be some bias in this data.

### Feature Engineering <a name="engineering"></a>

There was still no target value for modeling, since there was no feature corresponding to whether or not a user had churned. The `churn` column was engineered from the values of the `last_trip_date` and the date that the data was pulled, July 1st. If there had been more than 30 days since a user had last ridden and the date the data was pulled, they were said to have churned.

**Table 3**: Final dataset for modeling

|    |   avg_dist |   avg_rating_by_driver |   avg_rating_of_driver |   avg_surge | last_trip_date      | signup_date         |   surge_pct |   trips_in_first_30_days | luxury_car_user   |   weekday_pct |   city_King's Landing |   city_Winterfell |   phone_Android |   phone_iPhone |   days_since_last_ride | churn   |   days_since_customer |
|---:|-----------:|-----------------------:|-----------------------:|------------:|:--------------------|:--------------------|------------:|-------------------------:|:------------------|--------------:|----------------------:|------------------:|----------------:|---------------:|-----------------------:|:--------|----------------------:|
|  0 |       3.67 |                    5   |                    4.7 |        1.1  | 2014-06-17 00:00:00 | 2014-01-25 00:00:00 |        15.4 |                        4 | True              |          46.2 |                     1 |                 0 |               0 |              1 |                     14 | False   |                   157 |
|  1 |       8.26 |                    5   |                    5   |        1    | 2014-05-05 00:00:00 | 2014-01-29 00:00:00 |         0   |                        0 | False             |          50   |                     0 |                 0 |               1 |              0 |                     57 | True    |                   153 |
|  2 |       0.77 |                    5   |                    4.3 |        1    | 2014-01-07 00:00:00 | 2014-01-06 00:00:00 |         0   |                        3 | False             |         100   |                     0 |                 0 |               0 |              1 |                    175 | True    |                   176 |
|  3 |       2.36 |                    4.9 |                    4.6 |        1.14 | 2014-06-29 00:00:00 | 2014-01-10 00:00:00 |        20   |                        9 | True              |          80   |                     1 |                 0 |               0 |              1 |                      2 | False   |                   172 |
|  4 |       3.13 |                    4.9 |                    4.4 |        1.19 | 2014-03-15 00:00:00 | 2014-01-27 00:00:00 |        11.8 |                       14 | False             |          82.4 |                     0 |                 1 |               1 |              0 |                    108 | True    |                   155 |

![](images/pairplot.png)

**Figure 1**: Pairplot for the features in the dataset


**Table 3**: Cleaned data type and null value descriptions 

| feature | importance|
|:-----------------------|-----------:|
| phone_Android          |  0.106989  |
| avg_dist               |  0.0995247 |
| avg_rating_by_driver   |  0.0379053 |
| avg_rating_of_driver   |  0.0144364 |
| avg_surge              |  0.0143644 |
| weekday_pct            |  0.0113378 |
| days_since_customer    | -0.0292387 |
| surge_pct              | -0.0330836 |
| city_Winterfell        | -0.127842  |
| phone_iPhone           | -0.145512  |
| luxury_car_user        | -0.211977  |
| trips_in_first_30_days | -0.224234  |
| city_King's Landing    | -0.34943   |

