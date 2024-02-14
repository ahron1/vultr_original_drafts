Data Analysis in PostgreSQL

1. Introduction
1. Basic aggregation functions - sums, averages, ranges, percentiles
1. Statistical analysis functions - variances, standard deviations, correlations, regressions
1. JSON aggregation

## Introduction

Data analysis is of increasing importance in modern applications. Sophisticated analytics requires the use of specialized software. However, many commonly used, simpler analyses, such as averages and variances, as well as linear regressions and group-wise analyses, can be run from within PostgreSQL. It is advantageous to be able to do the analysis within the database itself.

Running the analyses within the database ...

## Prerequisites

### Background 

It is assumed you have prior practical exposure to the PostgreSQL database. It is helpful to have a working knowledge of basic statistical analysis.

### Systems

The commands in this article are tested on PostgreSQL 14.5 running on FreeBSD 13.1. SQL queries should work on all recent versions of PostgreSQL and file system based commands should work on all UNIX/Linux based Operating Systems.

## Example Dataset

Most of the builtin PostgreSQL functions used for data analysis are listed under **Aggregate Functions** in the [official PostgreSQL documentation](https://www.postgresql.org/docs/current/functions-aggregate.html). It is helpful to see the functions in action in the form of SQL queries on an actual dataset. Most of this article is organized as hypothetical analyses performed on this dataset. It is strongly recommended to try out the examples while reading through the article. 

This article uses a dataset about Capital Bikeshare, a bicycle-sharing company in Washington, D.C. This dataset is available publicly (behind a free login-wall) on [data.world](https://data.world/data-society/capital-bikeshare-2011-2012). The author has no known affiliation with either data.world or Capital Bikeshare. Kaggle also has a number of datasets freely available for analysis.

### Dataset Description

This dataset is about the number of people who used the bike-sharing service over a 2 year period. The number of users is shown for each hour of each day. Each row corresponds to an hour. The dataset has the following additional information in individual columns:

* season - each of the 4 seasons is mapped to an integer between 1 and 4
* weather_type - weather is encoded as an integer between 1 and 4. The weather, unlike the season, can change hourly; thus, each day can have many weather types
* holiday - whether the day is a holiday
* day_of_the_week - the day of the week that date falls on
* temperature_f - the hourly temperature in Fahrenheit
* temperature_feels_f - the hourly "feels like" temperature in Fahrenheit
* humidity - the humidity for each hour
* wind_speed - the wind speed for each hour
* casual_users, registered_users, total_users - the number of casual (unregistered), registered, and total users of the service for each hour
 
### Get the Data into the Database

Download the CSV file containing the data from the webpage of the dataset (the page has download buttons). For this particular dataset, the data file is `bike data.csv`. Pay attention to any spaces in filenames.

Move this file to the data directory of the `postgres` user. This makes it easier to access the file from within PostgreSQL. On most Linux/UNIX systems, the data directory is `/var/db/postgres`. 

    sudo cp ~/Downloads/bike\ data.csv /var/db/postgres/bike_data.csv 

The above command needs to be done in `sudo` mode because the regular user does not have access to the data directory of PostgreSQL.

Before importing the data into the database, create a new table. The column names and types of the new table should match the columns of the dataset. It is also helpful to use a new database for this.

Open a `psql` command line on the terminal, or use a graphical editor connected to PostgreSQL.

Create a new database:

    create database bike_data;

Switch to the newly created database:

    \c bike_data;

Create a new table: 

    create table data (date timestamp, season smallint, hour smallint, holiday boolean, day_of_the_week smallint, working_day boolean, weather_type smallint, temperature_f decimal, temperature_feels_f decimal, humidity decimal, wind_speed decimal, casual_users integer, registered_users integer, total_users integer);

Copy the data from the CSV file to the table:

    copy data from '/var/db/postgres/bike_data.csv' delimiter ',' csv header;

Check the table: 

    select * from data limit 5;

## Data Preprocessing

It is important to note that the data is on an hourly basis. For many analyses, it is useful to have the data on a daily basis. For instance, the average number of daily users is sometimes a more useful metric than the average number of hourly users.

As a first step, create a (materialized) view with the hourly data transformed into daily data. The number of hourly users over 24 hours is summed to the daily total number of users. The humidity and wind speed for the day is the average of the hourly humidity and wind speeds. Three values for the daily temperatures are considered - the maximum, the minimum, and the average over 24 hours. The value of the `season`, `holiday`, `working_day`, and `day_of_the_week` is the same for hourly data as well as daily data. The hourly value of `weather`, as mentioned earlier, is encoded as an integer between 1 and 4. Averaging this over 24 hours can lead to decimal values. So the `weather` column is omitted in the daily data.

    create materialized view daily_data as
    select 
        date::DATE, 
        sum(casual_users) as casual_users, 
        sum(registered_users) as reg_users, 
        sum(total_users) as total_users, 
        max(temperature_f) as max_temp, 
        min(temperature_f) as min_temp, 
        round(avg(temperature_f),1) as avg_temp, 
        max(temperature_feels_f) as max_temp_feels, 
        min(temperature_feels_f) as min_temp_feels, 
        round(avg(temperature_feels_f), 1) as avg_temp_feels, 
        round(avg(humidity), 1) as avg_humidity, 
        round(avg(wind_speed), 1) as avg_wind_speed, 
        season, 
        holiday, 
        day_of_the_week as day_of_week, 
        working_day 
    from data 
    group by 
        date, season, holiday, day_of_the_week, working_day 
    order by 
        date;

### Closer look

It is important to understand exactly what the data denotes. Ambiguity in the meaning of the input data can lead to inaccurate conclusions. In this case, the hourly number of users can refer to either the number of users who were using the service during that hour. It can also refer to the number of users to activated the service during the hour. In the former case, it is not correct to sum the hourly users to get the number of daily users. The same person can continue using the service for more than an hour. This will lead to double counting. 

## Data Analysis - I 

This section shows ways to do an exploratory analysis of the data itself - this helps in getting a "feel" of the data before digging deeper.

### Count the Number of Rows

The `count` function returns the number of rows that match the conditions in the `WHERE` clause. In the absence of a `WHERE` clause, it returns the total number of rows:

    select count(*) from data;

The above query returns the total number of rows in the table `data`. To get the number of rows where the `season` column has a value of 1:

    select count(*) from data where season = 1;

### Max and Min

### Sum the Values of a Column

The `sum` function returns the sum of all the values of the specified column over the rows that match the conditions in the `WHERE` clause. 

To get the total number of hourly users for a given day (1 Jan 2011):

    select sum(total_users) from data where date='1-1-2011';

**Caveat:** Note, in this case, that the sum of the hourly users is *not* the number of daily users. The same person can continue using the service for more than an hour. This will lead to double counting. This dataset does not have enough information to collate the hourly data into daily data.

### Average

Calculate the average of a number of values using the `AVG()` function. To get the average of the entire temperature_f column:

    select avg(temperature_f) from data;

It is also possible to calculate the average of only a subset of the values in a column. To get the average value of temperature_f for season 1:

    select avg(temperature_f) from data where season = 1;



### Percentiles


## Data Analysis - II 

### Grouping

One of the end goals of analytical exercises is to group the data into different classes to study and compare specific properties of each class. The *grouping operations* in PostgreSQL are often used to achieve this. This section also uses the functions discussed earlier and shows how to use them to draw practical insights from the data.

To get the number of rows for each season:

    select season, count(*) from data group by season;

Suppose you want to find the average number of users for each season. 

    select season, avg(total_users) as average_users from data group by season;

Sort the output of the above query so that the season with highest average number of users is at the top:

    select season, avg(total_users) as average_users from data group by season order by average_users desc;

It is also possible, though not necessary in this case, to do the average computation explicitly:

    select season, sum(total_users) / count(total_users) from data group by season;

Consider an example where in order to better understand seasonality, you need the maximum, minimum, and average temperature, as well as the total number of users for each season:

    select season, max(temperature_f) as max_temp, min(temperature_f) as min_temp, round(avg(temperature_f), 1) as average_temp, sum(total_users) as total_users from data group by season;

### Outliers

In many kinds of numerical analysis, 

What constitutes an outlier is a very subjective and domain-specific decision. For instance, if you are tracking local atmospheric temperatures, and the measuring device had a glitch and recorded 1000 degrees, that data point needs to be discarded from the dataset. Retaining outliers in the data can skew the analysis in the wrong direction.

In this case, consider (hypothetically) that days on which less than 2 people used the service are to be considered outliers. Such days are to be excluded from the computation.

     select season, avg(total_users) as avg from data where total_users > 2 group by season order by avg desc;

Note that the `WHERE` clause applies its conditions before the aggregation operations are performed. It is useful not just for excluding outliers, but also for ....

### Using Aggregate Functions in the `WHERE` clause

Suppose you want to get the details of the day(s) on which the temperature was at its highest. The query below is wrong because it is not allowed to have aggregate functions as part of the `WHERE` clause of a SQL query.

    select * from data where temperature_f = max (temperature_f);

The aggregate function, `MAX`, needs to be used within a sub-query before its output can be used as part of the main query.

    select * from data where temperature_f = (select max (temperature_f) from data);


### Filter




--------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------

    https://stackoverflow.com/questions/52432231/how-to-do-a-linear-regression-in-postgresql

    https://docs.yugabyte.com/preview/api/ysql/exprs/aggregate_functions/function-syntax-semantics/array-string-jsonb-jsonb-object-agg/ 

    https://blog.k-nut.eu/postgres-json-aggregate-functions

    https://towardsdatascience.com/how-to-derive-summary-statistics-using-postgresql-742f3cdc0f44

    https://www.postgresql.org/docs/9.5/functions-aggregate.html

    data sets - https://www.kaggle.com/code/rtatman/datasets-for-regression-analysis 

    https://www.postgresql.org/docs/current/functions-aggregate.html

