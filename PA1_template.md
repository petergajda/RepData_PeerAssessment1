    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(ggplot2)
    library(lattice)

Loading and preprocessing the data
----------------------------------

With the following codechunk the working directory is set and the
variable "activity" is created, which contains the data. In the final
step, all NA values are removed.

    # set working directory
    setwd("D:/Projects/R/Coursera/Week 4/data2/")

    # create variable activity
    data <- read.csv("activity.csv")

    # remove all NA in the variable activity
    activity <- data[ with (data, { !(is.na(steps)) } ), ]

What is mean total number of steps taken per day?
-------------------------------------------------

Creating a histogram with the total Number of steps by day.

    # aggregate steps by date
    steps <- aggregate(activity$steps, by=list(date=activity$date), FUN=sum)

    # histogram of Number of steps by day
    hist(steps$x, main = "Total number of steps per day", xlab = "Steps per day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

The mean of the total steps per day is 10766.19, the median is 10765.

    # mean of total number of steps
    mean(steps$x)

    ## [1] 10766.19

    # median of total number of steps
    median(steps$x)

    ## [1] 10765

What is the average daily activity pattern?
-------------------------------------------

Creating a time series plot with the average number of steps of all
days.

    # create variable steps_interval
    steps_interval <- aggregate(activity$steps, by=list(interval=activity$interval), FUN=mean)


    # create time series plot 
    plot(steps_interval$interval, steps_interval$x, type='l', 
         main="Average number of steps of all days", xlab="Interval", 
         ylab="Average number of steps")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png)

The interval 835 contains the highest average of steps with around
206.2.

    steps_interval %>% filter(steps_interval$x == max(steps_interval$x))

    ##   interval        x
    ## 1      835 206.1698

Imputing missing values
-----------------------

The difference of the length of the steps-variable of the original an
na-removed dataset indicates the number of NAs. It is 2304.

    length(data$steps) - length(activity$steps)

    ## [1] 2304

With the following method all missing values get assigned the mean of
the 5-minute interval of each day. For that purpose a new dataset
"data2" is created.

    data2 <- data

    for (i in 1:nrow(data2)) {
            if(is.na(data2$steps[i])) {
                    mean_steps <- steps_interval$x[which(steps_interval$interval == data$interval[i])]
                    data2$steps[i] <- mean_steps
            }
    }       

    # aggregate steps by date
    steps2 <- aggregate(data2$steps, by=list(date=data2$date), FUN=sum)

    # histogram of Number of steps by day
    hist(steps2$x, main = "Total number of steps per day", xlab = "Steps per day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-10-1.png)

The mean of the total steps per day is 10766.19, the median is 10766.19
(before: 10765).

    # mean of total number of steps
    mean(steps2$x)

    ## [1] 10766.19

    # median of total number of steps
    median(steps2$x)

    ## [1] 10766.19

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Preparing dataset "data2" with indicator of weekday or weekend.

    data2$weekdays <- weekdays(as.Date(data2$date))
    data2$days <- c("weekday")
    data2$days <- ifelse(grepl("Sonntag", data2$weekdays), "weekend", data2$days)
    data2$days <- ifelse(grepl("Samstag", data2$weekdays), "weekend", data2$days)
    head(data2)

    ##       steps       date interval weekdays    days
    ## 1 1.7169811 2012-10-01        0   Montag weekday
    ## 2 0.3396226 2012-10-01        5   Montag weekday
    ## 3 0.1320755 2012-10-01       10   Montag weekday
    ## 4 0.1509434 2012-10-01       15   Montag weekday
    ## 5 0.0754717 2012-10-01       20   Montag weekday
    ## 6 2.0943396 2012-10-01       25   Montag weekday

Creating variable "steps\_interval" for plotting Number of steps by
Interval in comparison of weekend and weekdays.

    steps_interval <- aggregate(data2$steps, by=list(interval=data2$interval, days=data2$days), FUN=mean)

    with(steps_interval, xyplot(x ~ interval | days, type = "l", xlab = "Interval", ylab = "Number of Steps", layout=c(1,2)))

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-14-1.png)
