---
title: "Reproducible Research - Course Project 1"
author: "elgalvan"
date: "24/3/2020"
output: 
  html_document:
    keep_md: true
---



## Loading and preprocessing the data
### 1. Code for reading in the dataset and/or processing the data
Show any code that is needed to

Load the data (i.e. read.csv())
Process/transform the data (if necessary) into a format suitable for your analysis


```r
### Reading file activity,csv
activity <- read.csv("data/activity.csv", header = T)

### Set field Date as datatype Date
activity$date <- as.Date(activity$date, format = "%Y-%m-%d")

### Show data
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

## What is mean total number of steps taken per day?
### 2. Histogram of the total number of steps taken each day

For this part of the assignment, you can ignore the missing values in the dataset.

Calculate the total number of steps taken per day
If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
Calculate and report the mean and median of the total number of steps taken per day


```r
### Filter NA data

activity_filtered <- activity[!is.na(activity$steps),] 

### Group by date
activity_group <- group_by(activity_filtered, date)

### Sum
activity_tidy <- summarise_each(activity_group, list(sum))

### Making Plot
hist(activity_tidy$steps, main = "Total number of steps in a day", xlab = "Total number of steps")
```

![](report_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

## Mean and median number of steps taken each day



The mean is 10766. The median is 10765.
 
## What is the average daily activity pattern?
### Time series plot of the average number of steps taken

Make a time series plot (i.e. type = "l" of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
### Group by interval
activity_group_steps <- group_by(activity_filtered, interval)

### Mean
activity_tidy_steps <- summarise_each(activity_group_steps, list(mean))

### find interval with this max of steps
interval_max_steps <- activity_tidy_steps$interval[which.max(activity_tidy_steps$steps)]

### Making Plot
plot(activity_tidy_steps$interval, activity_tidy_steps$steps, type="l", xlab="", ylab="Global Active Power")
```

![](report_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

### The 5-minute interval that, on average, contains the maximum number of steps
The interval that contains the maximum number of steps is 835

## Imputing missing values
### Code to describe and show a strategy for imputing missing data

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NA)

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
### Sum NAs Values
activity_na_totals <- sum(is.na(activity$steps))
```

1. The total NAs values is 2304

2. Strategy for filling in all of the missing values in the dataset.

```r
### Select columns
activity_filtered_1 <- activity[!is.na(activity$steps), c(1, 3)] 

### Group by date
activity_group_steps_1 <- group_by(activity_filtered_1, interval)

### Mean
activity_tidy_date <- summarise_each(activity_group_steps_1, list(mean))

### Select Only records whit NAs values
activity_filtered_2 <- activity[is.na(activity$steps),]

### Join
activity_join <- inner_join(activity_filtered_2, activity_tidy_date, by = "interval")[, c(4, 2, 3)]

### Replace names columns
names(activity_join) <- c("steps", "date", "interval")
```

3. New Dataset

```r
### Merge data
activity_complete <- rbind(activity_filtered, activity_join)
```

4. Make plot and inform mean and median

```r
### Group by Date
activity_complete_group <- group_by(activity_complete, date)

### Sum
activity_complete_tidy <- summarise_each(activity_complete_group, list(sum))

### Make a plot
hist(activity_complete_tidy$steps, main = "Total number of steps in a day", xlab = "Total number of steps")
```

![](report_files/figure-html/unnamed-chunk-8-1.png)<!-- -->



## The new mean is 10766. The new median is 10766.
### 

Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
### Set name day
activity_filtered$day <- weekdays(activity_filtered$date)
activity_filtered$day_type <- weekdays(activity_filtered$date)

### Replace data
activity_filtered$day_type <- gsub("Monday|Tuesday|Wednesday|Thursday|Friday", "weekday", activity_filtered$day_type)
activity_filtered$day_type <- gsub("Saturday|Sunday", "weekend", activity_filtered$day_type)

### Select Columns
activity_filtered_2 <- activity_filtered[, c(5, 3, 1)]
activity_group_2 <- group_by(activity_filtered_2, interval, day_type)

### Mean
activity_tidy_2 <- summarise_each(activity_group_2, list(mean))

### Make Plot
g <- ggplot(activity_tidy_2, aes(x = interval , y = steps)) + geom_line() + facet_grid(day_type~.) + labs(title = "Average number of steps Vs 5-min Interval Times Series", y="Average Number of Steps", x = "5-min Interval Times Series")
print(g)
```

![](report_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
