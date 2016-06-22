# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Load the activity data from the link below.


```r
library(data.table)
dt <- fread("curl https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip | funzip")
```

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing values are coded as 𝙽𝙰)
* **date**: The date on which the measurement was taken in YYYY-MM-DD format
* **interval**: Identifier for the 5-minute interval in which measurement was taken

## What is mean total number of steps taken per day?

Remove the NA values for this step of the analysis.


```r
dt.na <- na.omit(dt)
```

The total number of steps in the day can be got by summing the steps variable.


```r
total.steps <- sum(dt$steps)
```

The histogram below shows the distribution of the number of steps taken.


```r
hist(dt.na$steps, col = "darkslategray3", border = "darkslategray4", main="Histogram of the number of steps taken per interval", , xlab="Number of steps in the interval", ylab="Number of intervals")
```

![](PA1_template_files/figure-html/hist-1.png)<!-- -->



The mean number of steps taken is 37.4 and the median number of steps is 0.

## What is the average daily activity pattern?

Below shows a timeseries plot of the 5-minute intervals and the average number of steps taken across the recorded days.


```r
library(plyr)
```

```
## Warning: package 'plyr' was built under R version 3.2.5
```

```r
by.interval <- ddply(dt, .(interval), summarize, mean_steps=round(mean(steps, na.rm = TRUE), 1))
with(by.interval, plot(interval, mean_steps, type="p", pch=18, col="indianred4", main="Timeseries plot for average number of steps in each interval"))
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->


## Imputing missing values

In order to assess the bias caused by missing values, first the number of missing values was computed.


```r
sum(is.na(dt))
```

```
## [1] 2304
```

To assess how much these are biasing the data, I have replaced the NA values with the average value for that interval. 


```r
dt.filled <- dt
dt.filled$steps <- replace(dt.filled$steps, (is.na(dt.filled$steps) && dt.filled$interval == by.interval$interval), by.interval$mean_steps)
```

The new values are then compared to the histogram above.


```r
hist(dt.filled$steps, col = "darkslategray3", border = "darkslategray4", main="Histogram of the number of steps taken per interval", xlab="Number of steps in the interval after filling the NA", ylab="Number of intervals")
```

![](PA1_template_files/figure-html/hist2-1.png)<!-- -->



After recalculating the mean and median with the new data set, the mean is still 37.4 and the median number of steps remains as 0.


## Are there differences in activity patterns between weekdays and weekends?

In order to differentiate betwen the number of steps on weekdays and weekends, first the data set is split in two by the day.


```r
dt.filled$day <- factor(weekdays(as.Date(dt.filled$date, format="%Y-%m-%d")), levels=c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"), labels=c("weekday", "weekday", "weekday", "weekday", "weekday", "weekend", "weekend"))
```

```
## Warning in `levels<-`(`*tmp*`, value = if (nl == nL) as.character(labels)
## else paste0(labels, : duplicated levels in factors are deprecated
```

Below is a panel plot which decribes the split of days between weekday and weekend by the average number of steps taken in each interval.


```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.2.4
```

```r
g <- ggplot(dt.filled, aes(interval, steps, col="indianred4"))
g + geom_point() + facet_grid(. ~ day) + guides(color=FALSE)
```

![](PA1_template_files/figure-html/plot.panel-1.png)<!-- -->
