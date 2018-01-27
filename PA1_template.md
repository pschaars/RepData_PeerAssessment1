---
output: 
  html_document:
    keep_md: true
---


## Introduction

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a
[Fitbit](http://www.fitbit.com), [Nike
Fuelband](http://www.nike.com/us/en_us/c/nikeplus-fuelband), or
[Jawbone Up](https://jawbone.com/up). These type of devices are part of
the "quantified self" movement -- a group of enthusiasts who take
measurements about themselves regularly to improve their health, to
find patterns in their behavior, or because they are tech geeks. But
these data remain under-utilized both because the raw data are hard to
obtain and there is a lack of statistical methods and software for
processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012
and include the number of steps taken in 5 minute intervals each day.

## Data

The data for this assignment can be downloaded from the course web
site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken




The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this
dataset.


### Loading and preprocessing the data

the github repository was cloned and the directory set


```r
require(data.table)
require(dplyr)
require(ggplot2)
```

1. Load the data (i.e. `fread()`)

```r
unzip("activity.zip")
activity <- fread("activity.csv") %>% mutate(date= as.Date(date))
```


### What is mean total number of steps taken per day?

A histogram of the total number of steps taken each day

```r
total <- activity %>% group_by(date) %>% summarize(dailySteps = sum(steps))
hist(total$dailySteps, breaks = 10)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

The **mean** and **median** total number of steps taken per day

```r
total %>% summarize(mean = mean(dailySteps, na.rm=TRUE), median = median(dailySteps, na.rm=TRUE))
```

```
## # A tibble: 1 x 2
##    mean median
##   <dbl>  <int>
## 1 10766  10765
```


### The average daily activity pattern

A time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
intervalAverage <- activity %>% 
  group_by(interval) %>% 
  summarize(intervalSteps = mean(steps, na.rm=TRUE)) 

ggplot(intervalAverage, aes(x=interval, y=intervalSteps)) +
  geom_line()
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

The 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps

```r
intervalAverage[intervalAverage$intervalSteps==max(intervalAverage$intervalSteps), "interval"]
```

```
## # A tibble: 1 x 1
##   interval
##      <int>
## 1      835
```


### Imputing missing values

Note that there are a number of days/intervals where there are missing
values (coded as `NA`). The presence of missing days may introduce
bias into some calculations or summaries of the data.

The total number of missing values in the dataset (i.e. the total number of rows with `NA`s)

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```



New dataset that is equal to the original dataset but with the missing data filled in.

```r
temp <- activity[is.na(activity$steps), ]
activityImputed <- activity[!(is.na(activity$steps)), ]
activityImputed <- temp %>% 
  select(-steps) %>% 
  left_join(intervalAverage) %>% 
  rename(steps = intervalSteps) %>%
  union_all(activityImputed)
```


Histogram of the total number of steps taken each day and Calculate and report the **mean** and **median** total number of steps taken per day. No sustantial change from ignoring NAs. 

```r
total <- activityImputed %>% group_by(date) %>% summarize(dailySteps = sum(steps))
hist(total$dailySteps, breaks = 10)
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
total %>% summarize(mean = mean(dailySteps), median = median(dailySteps))
```

```
## # A tibble: 1 x 2
##    mean median
##   <dbl>  <dbl>
## 1 10766  10766
```


### Are there differences in activity patterns between weekdays and weekends?

New factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
activityImputed <- activityImputed %>% 
  mutate(weekend = factor(weekdays(date)=="Saturday" | weekdays(date)=="Sunday", labels=c("Weekday", "Weekend")))
total <- activityImputed %>% group_by(interval, weekend) %>% summarize(steps = sum(steps))
```

Panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
ggplot(total, aes(x=interval, y=steps)) + 
  geom_line() + 
  facet_wrap(~weekend)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->
