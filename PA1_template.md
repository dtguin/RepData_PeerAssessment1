---
title: 'Reproducible Research: Peer Assessment 1'
output:
  html_document:
    keep_md: yes
---
#### Author: David Guinivere
<br>   <!-- bliank line -->
<br>   <!-- bliank line -->

## Loading and preprocessing the data
The data file used for this analysis is: activity.csv 
and MUST be located in your current working directory.

The activity.csv file is read into the R Object "df_activity" as follows:

```r
df_activity <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

In order to calculate the mean total number of steps per day the data (contained in df_activity data frame) needs to be aggregated by day. Which gives use 61 observations.   31 for each day from October 1 through October 31, plus 30 for November 1 through November 31.

```r
#
## Create df_steps from df_activity and drop the interval variable
df_steps <- df_activity
df_steps$interval <- NULL
#
## Data frame aggregated by days
df_days <- data.frame(unique(df_steps$date))
#
## Set resonable variable name
names(df_days) <- "date"
#
## Get sum of steps aggregated by days
suppressWarnings(aggsum <- aggregate(df_steps$steps, by=list(df_steps$date),FUN=sum))
#
## add <<sum of steps>> per day variable to df_day
df_days$sum <- aggsum$x
```
The data frame df_days now has 61 observations, 1 for each day, and 2 variales: date and sum.

Below is a histogram showing the sum of steps taken per day.  I chose 30 bins for my histogram as this is the avewrage number of days per month, and shows a good ratio of noise to precision which shows the data distribution


```r
hist(df_days$sum,breaks=30, main="Histogram showing total number of steps per day.", xlab="Sum of steps.")
```

![plot of chunk histogram_mean_steps](figure/histogram_mean_steps-1.png) 

In order to calculate the mean and median values for the total steps per day, the "NA" values must be removed from the aggregated data frame.  The "NA" values are removed, rather than substituted with 0 (or some other value) so as to minimize bias to the calculations.


```r
#
## Remove the NA values for computing the mean and median number of steps taken per day
df_days <- df_days[which(complete.cases(df_days)),]
#
## "Switch off" scientific notation for mean() and median() calculations
options(scipen=999)
#
## Calculate the mean number of steps per day by taking the mean of each sum of the daily number of steps per day.
mean_steps = round(mean(df_days$sum), 1)
#
## Calculate the median number of steps per day by taking the median of  of each sum of the daily number of steps per day.
median_steps = median(df_days$sum)
```
The mean of the number of steps taken per day is 10766.2.

The median of the number of steps taken per day is 10765.

## What is the average daily activity pattern?

In order to make a time series plot with each 5-minute interval for the x-axis, and the average number of stweps traken, for the y-axis, the data contained in df_activity is aggregated by the 5-minute interval, the 5-minute interval is transformed into POSIXlt values.


```r
suppressWarnings(library(ggplot2))
suppressWarnings(library(scales))
suppressWarnings(library(lubridate))
#
## Clone df_activity into df_int and drop the date variable
df_int <- df_activity
df_int$date <- NULL
#
## Get the complete cases in df_int
df_int <- df_int[which(complete.cases(df_int)),]
#
## aggregate by 5-minute interval
df_int <- aggregate(df_int$steps,by=list(df_int$interval),FUN=mean)
names(df_int) <- c("interval", "steps")
#
## setup $time
df_int$time <- formatC(df_int$interval, width=4, format="d", flag="0")
df_int$time <- gsub('^([0-9]{2})([0-9]+)$', '\\1:\\2', df_int$time)
#
## Prepend "2012-10-01 " which is OK since we won't show it on X-axis
## add ":00" for POSIXlt seconds
df_int$time <- paste("2012-10-01 ",df_int$time,":00",sep="")
#
## Convert to POSIXlt
df_int$time <- as.POSIXlt(df_int$time) 
## Intermediate:  ggplot(data=df_int,aes(x=time,y=steps)) + geom_line()
ggplot(data=df_int,aes(x=time,y=steps)) + 
        geom_line()+scale_x_datetime(labels = date_format("%H:%M"),
                breaks = date_breaks("2 hour")) +
        ggtitle("Average number of steps, per 5-minute interval") +
        xlab("Time") +
        ylab("Steps")
```

![plot of chunk daily_activity_plot](figure/daily_activity_plot-1.png) 

```r
#
## Now find the interval with the maximum average number of steps
#
## Max steps
max_steps_value <- max(df_int$steps)
#
## Determine which interval has the max # of steps
max_steps_interval <- df_int$time[which(df_int$steps==max(df_int$steps))]
max_steps_interval <- as.character(max_steps_interval)
max_steps_interval <- substr(max_steps_interval, 12, 16)
```

The 08:35 interval has the maximum number of steps averaged across all days.

The value for this interval is 206.17.

## Imputing missing values

The total number of missing values in the data set is 2304.

The strategy for imputing the missing data, is to replace the missing steps value with the mean of the 5-minute interval, as averaged across all days for the specific interval, as shown in the code chunk below:


```r
#
## Make a copy of the original data set, where the NA values will be imputed
df_impute <- df_activity
#
## Fill in the df_impute[,"steps"] missing values with the mean of the steps for that interval
for (idx in 1:length(df_impute$steps)) {
        if (is.na(df_impute[idx,"steps"])) {
                interval <- df_impute[idx, "interval"]
                df_impute[idx,"steps"] <- df_int[which(df_int$interval == interval), "steps"]
        }
}
#
## Create df_steps from df_activity and drop the interval variable
df_steps <- df_impute
df_steps$interval <- NULL
#
## Data frame aggregated by days, gives one observation per day
df_days_impute <- data.frame(unique(df_steps$date))
#
## Set resonable variable name
names(df_days_impute) <- "date"
#
## Get sum of steps aggregated by days
aggsum <- aggregate(df_steps$steps, by=list(df_steps$date),FUN=sum)
#
## add <<sum of steps>> per day variable to df_days_impute
df_days_impute$sum <- aggsum$x
hist(df_days_impute$sum,breaks=30, main="Histogram showing total number of steps per day with imputed data.", xlab="Sum of steps.")
```

![plot of chunk imputing_missing_values](figure/imputing_missing_values-1.png) 

```r
#
## Repeat the original histogram... use the df_days data frame from part 1
hist(df_days$sum,breaks=30, main="Histogram showing total number of steps per day.", xlab="Sum of steps.")
```

![plot of chunk imputing_missing_values](figure/imputing_missing_values-2.png) 

```r
#
## Calculate the mean number of steps per day by taking the mean of each sum of the daily number of steps per day.
mean_steps_imputed <- round(mean(df_days$sum), 1)
#
## Calculate the median number of steps per day by taking the median of  of each sum of the daily number of steps per day.
median_steps_imputed <- round(median(df_days$sum),0)
```
The mean of the number of steps taken per day using imputed data is 10766.2 versus the "un-imputed" mean 10766.2.

The median of the number of steps taken per day using imputed data is 10765,  versus the "un-imputed" median 10765.

The histogram using the imputed data, has greater frequecy for the  10000 bin.     However the histograms have roughly the same distribution of steps per interval when averaged over all days.


## Are there differences in activity patterns between weekdays and weekends?

Starting with the imputed data set (df_impute), add a factor variable for "weekday", or "weekend".


```r
#
##
df_pattern <- df_impute
#
## Create a POSIXlt variable for tha date.  Pad interval with leading 0's
df_pattern$day <- formatC(df_pattern$interval, width=4, format = "d", flag = "0")
# Insert ":" in the 3rd position
df_pattern$day <- gsub('^([0-9]{2})([0-9]+)$', '\\1:\\2', df_pattern$day)
# Prepend the datge plus a space
df_pattern$day <- paste(df_pattern$date,df_pattern$day)
# Convert variable to POSIXlt
df_pattern$day <- as.POSIXlt(df_pattern$day)
# Finally get the weekday name.
df_pattern$day_name <- weekdays(df_pattern$day)
#
## Create the character variable $day_type to be either "weekday" or "weekend"
for (idx in 1:length(df_pattern$day_name)) {
        if (identical(as.character(df_pattern[idx,"day_name"]),"Sunday")) {
                df_pattern[idx,"day_type"] <- "Weekend"
        } else if (identical(as.character(df_pattern[idx,"day_name"]),"Saturday")) {
                df_pattern[idx,"day_type"] <- "Weekend"
        } else {
                df_pattern[idx,"day_type"] <- "Weekday"
        }
}
#
## Convert $day_type to a factor variable
df_pattern$day_type <- as.factor(df_pattern$day_type)
## Plot a time series graph, faceted by $day_type
ggplot(df_pattern, aes(x=interval, y=steps)) + geom_line(col="cyan") +
        facet_grid(day_type ~ .) +
        ggtitle("Average steps by interval, faceted by Weekday or Weekend") +
        xlab("Interval") +
        ylab("Steps")
```

![plot of chunk create_day_type](figure/create_day_type-1.png) 
