---
title: "PA1_template -- Project 1"
output: html_document
---

# Load libraries to be used

```
library(knitr)
library(ggplot2)
opts_chunk$set(echo = TRUE)
```



# Set the working directory and load the data
```
setwd("C:/Users/Kevin/Desktop/ReproducibleResearch")
activitydata <- read.csv("activity.csv")
```

# What is mean total number of steps taken per day?

## Calculates the sum of steps for each day in dataset, stores it in steps_day function

```
steps_day <- aggregate(steps~date,activitydata, sum)
```

# Make a histogram of the total number of steps taken each day
hist(steps_day$steps,main="Total # of steps per day", xlab="Steps", ylab="# of days", col="brown")


# Calculate the mean, median. Mean = 10,766.19. Median = 10,765
```
mean(steps_day$steps, na.rm=TRUE)
median(steps_day$steps, na.rm=TRUE)
```

# What is the average daily activity pattern?

## Calculates the average of steps by interval
```
steps_interval <- aggregate(steps~interval,activitydata,mean)
```

## Calculates the max. The max number of average daily steps occurs at interval 835 and is equal to 206.1698 steps
```
steps_interval[steps_interval$steps==max(steps_interval$steps),]
```


# Time series plot of the 5-minute interval and the average number of steps taken, averaged across all days

```
ggplot(data=steps_interval, aes(x=interval, y=steps, group=1))+geom_line(color="red", size=1.25) + geom_point(color="blue", size=2, shape=21, fill="white")+labs(x="Interval", y="Average Steps per Day")+ggtitle("Average Daily Activity Pattern")+theme(plot.title=element_text(size=32))
```


# Imputing missing values 

## Count the number of observations WITH an NA in 'steps' variable. There are 2304 missing values in "steps"

```
sum(is.na(activitydata$steps))
```

## 0 missing values in "interval"
```
sum(is.na(activitydata$interval))
```

## 0 missing values in "date"
```
sum(is.na(activitydata$date))
```


# Impute missing values based on the overall dataset mean value for "steps"
```
activity_filled <- activitydata
activity_filled$steps[which(is.na(activity_filled$steps))] <- mean(activitydata$steps,na.rm=TRUE)
```

# Create new data frame with steps per day including missing value replacements
```
activity_filled_steps_day <- aggregate(steps~date,activity_filled,sum)
```

# Create a histogram that shows the total number of steps per day, along with frequencies
```
hist(activity_filled_steps_day$steps, col="green", main="Total steps per day", xlab="Steps")
```

### Find mean and median, which, interestingly equals the same thing. The mean and median total # of steps per day with the missing value replacement strategy is 10,766.19. # This is likely due to the simplicity of the strategy I took. #There were over 2300 values replaced, or nearly 15% of the original dataset, with the same mean step value.
### These values are similar to the values calculated earlier when we ignored the missing values. The mean is exactly the same, which isn't surprising, because we replaced the missing values with the mean step value. The median is #slightly higher (a difference of +1) after replacing the missing values.

```
mean(activity_filled_steps_day$steps)
median(activity_filled_steps_day$steps)
```



# Are there differences in activity patterns between weekdays and weekends?


## Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```
activity_filled$daytype <- weekdays(as.Date(activity_filled$date))


activity_filled$daytype[activity_filled$daytype %in% c("Saturday","Sunday")] <- "Weekend"
activity_filled$daytype[activity_filled$daytype %in% c("Monday","Tuesday","Wednesday","Thursday", "Friday")] <- "Weekday"

activity_filled$daytype <- factor(activity_filled$daytype,c("Weekend","Weekday"))

```

# Calculates the average of steps by interval using the new activity_filled dataset
```
steps_interval_activity_filled <- aggregate(steps~interval+daytype,activity_filled,mean)
```

# Change order of factor variables so that 'Weekend' ends up on top
```
steps_interval_activity_filled$daytype <- factor(steps_interval_activity_filled$daytype, levels=c("Weekend","Weekday"))
```

## Plot 5-minute intervals against average # of steps averaged across all weekend or weekday types
```
ggplot(data=steps_interval_activity_filled, aes(x=interval, y=steps, group=1))+facet_wrap(~daytype,ncol=1)+geom_line(color="red", size=1.25) + geom_point(color="blue",size=2, shape=21, fill="white") + labs(x="Interval", y="Number of Steps per Day") + ggtitle("Average Steps/Day, by Interval \n (Weekend vs. Weekday)")+theme(plot.title=element_text (size=32))
```
