---
layout: post
title: "Cyclistic Case Study"
subtitle: "A study of a Chicago based fictional bike rental company"
background: '/img/posts/01.jpg'
---

``` r
install.packages("tidyverse", repos = "http://cran.us.r-project.org")
install.packages("data.table", repos = "http://cran.us.r-project.org")
install.packages("lubridate", repos = "http://cran.us.r-project.org")

library(tidyverse)
library(dplyr)
library(data.table)
library(lubridate)
```

## 1. Case Study Description


The following document is a record of the steps that I’ve taken in doing
a case study over the *fictional* company Cyclistic, which offers a bike
sharing service. We’ll be attempting to answer the question of how
annual and casual members use the service differently and how this
analysis can help drive decision making that results in more casual
members adopting an annual membership.

The license to use the public data has been provided by Lyft Bikes and
Scooters, LLC (“Bikeshare”) operates the City of Chicago’s (“City”)
Divvy bicycle sharing service The license to use this public dataset can
be found [here](https://ride.divvybikes.com/data-license-agreement)

## 2. Importing and Combining the Data


After downloading the last twelve months worth of data from the source,
the data has to be imported into R. Using RStudio’s Desktop version,
we’ve set the source folder to be the location where the data was
stored. This will make referencing the location while reading the .csv
in much more simple.

### Importing the Data

``` r
Dec_2021 <- read.csv("Dec_2021.csv")
Jan_2022 <- read.csv("Jan_2022.csv")
Feb_2022 <- read.csv("Feb_2022.csv")
Mar_2022 <- read.csv("Mar_2022.csv")
Apr_2022 <- read.csv("Apr_2022.csv")
May_2022 <- read.csv("May_2022.csv")
Jun_2022 <- read.csv("Jun_2022.csv")
Jul_2022 <- read.csv("Jul_2022.csv")
Aug_2022 <- read.csv("Aug_2022.csv")
Sep_2022 <- read.csv("Sep_2022.csv")
Oct_2022 <- read.csv("Oct_2022.csv")
Nov_2022 <- read.csv("Nov_2022.csv")
```

### Checking Data Structure

Checking the structure of the data to make sure that all the columns
will align when the tables are combined.

``` r
str(Dec_2021)
str(Jan_2022)
str(Feb_2022)
str(Mar_2022)
str(Apr_2022)
str(May_2022)
str(Jun_2022)
str(Jul_2022)
str(Aug_2022)
str(Sep_2022)
str(Oct_2022)
str(Nov_2022)
```

### Combining the Datasets

The columns in the twelve data sets already align in their typing, so we
can proceed with combining the twelve months into one yearly data set.

``` r
Yearly_Summary <- bind_rows(
  Dec_2021, Jan_2022, Feb_2022, Mar_2022, Apr_2022, May_2022, Jun_2022, 
  Jul_2022, Aug_2022, Sep_2022, Oct_2022, Nov_2022
  )
```

## 3. Preparing the Data


### Converting Formats

The format for the rows *started_at* and *ended_at* are currently in the
character format. Since we’ll use these values for analysis of the
numerical value they possess in relation to dates and times, they’ll
need to be converted into a usable date format. For this we’ll use the
lubridate package.

``` r
# Changing started_at to a date format
Yearly_Summary$started_at <- as_datetime(
  Yearly_Summary$started_at, format = '%Y-%m-%d %H:%M:%S'
  )

# Changing ended_at to a date format
Yearly_Summary$ended_at <- as_datetime(
  Yearly_Summary$ended_at, format = "%Y-%m-%d %H:%M:%S"
)

#Sorting the sheet by the started_at column
Yearly_Summary <- Yearly_Summary %>% 
  arrange(started_at)
```

### Calculating Ride Lengths

Creating the ride length column and calculating the ride lengths by
using difftime.

``` r
Yearly_Summary$ride_length <- difftime(
  Yearly_Summary$ended_at,
  Yearly_Summary$started_at,
  units = "secs"
)

Yearly_Summary$ride_length <- as.numeric(
  as.character(Yearly_Summary$ride_length)
  )
```

### Creating Columns for YMD and Time of Day

Creating columns to separate out year, month, week and day. Also
creating a column for the time of day.

``` r
Yearly_Summary$year <- format(
  Yearly_Summary$started_at,
  "%Y"
  )

Yearly_Summary$month <- format(
  Yearly_Summary$started_at,
  "%m"
  )

Yearly_Summary$week <- format(
  Yearly_Summary$started_at,
  "%W"
  )

Yearly_Summary$day <- format(
  Yearly_Summary$started_at,
  "%d"
  )

Yearly_Summary$day_of_week <- format(
  Yearly_Summary$started_at,
  "%A"
  )

Yearly_Summary$YMD <- format(
  Yearly_Summary$started_at,
  "%Y-%m-%d"
  )

Yearly_Summary$ToD <- format(
  Yearly_Summary$started_at,
  "%H:%M:%S"
  )
```

## 4. Cleaning the Data


### Removing instances of rides with time less than 0

``` r
Yearly_Summary_cleaned <- Yearly_Summary %>%
  filter(!(ride_length < 0))
```

### Removing Incomplete Rows

``` r
Yearly_Summary_cleaned <- Yearly_Summary_cleaned %>%
    filter(
      !(is.na(start_station_name) |
          start_station_name == "")
      ) %>% 
  
  filter(
    !(is.na(end_station_name) |
        end_station_name == "")
    )
```

### Removing Duplicates

No duplicates were found in the ride ids.

``` r
ride_id_check <- Yearly_Summary_cleaned %>%
  count(ride_id) %>%
  filter(n > 1)
```

## 5. Saving the Data

``` r
# Cleaned dataset
write.csv(Yearly_Summary_cleaned, "C:\\Users\\Brandon\\Documents\\Portfolio\\Cyclistic\\2 - Cleaned Data\\Yearly_Summary_cleaned.csv", row.names=FALSE)
```
