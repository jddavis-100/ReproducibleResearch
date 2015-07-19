---
title: "Reproducible Research Fitness Assignment 1"
output: html_document
---

This assignment involves analysis of activity data from a activity monitor (e.g. a fitbit) worn daily.  The individual's steps were measured in 5 second intervals.  Some data is missing and so it was imputed using a function I show within this document.  The imputation function was chosen to randomly generate numbers within a reasonable distribution of the mean of this data set.

The first question was how to load and prepare the data.  I did so as follows:

```{r}
activity <- read.csv("~/Documents/Reproducible Research/activity.csv", na.strings="NA")

#mean number of steps taken will need to exclude NA
activity$steps <- as.numeric(activity$steps)
activity$interval <- as.numeric(activity$interval)

```{r, echo=FALSE}
```

A histogram of the raw data as well as summary statistics (including mean and median), were included in the initial analyses:

```{r}
StepsTotal <- aggregate(steps ~ date, data=activity, sum, na.rm=TRUE)
hist(x=StepsTotal$steps, col = "blue", breaks=50, xlab="Steps Per Day", main="Daily Number of Steps", cex=1, cex.lab=0.75, cex.axis=0.75, cex.main=0.95, cex.sub=0.75, font=2, font.lab=2)
summary(StepsTotal$steps)
```

Next a time-series plot was made which shows the 5-minute interval average of number of steps taken and averaged across all days.  You will also see which 5-minute interval across all days contained the maximum number of steps.  We also calculate the number of missing values for steps.

```{r}
library(lattice)
activity$date <- as.character(activity$date)
time_series <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)
plot(row.names(time_series), time_series, type = "l", xlab = "5 minute Intervals", ylab="Average All Days", main="Average Steps Taken", col="blue", cex=1, cex.axis=0.75, cex.lab=0.75, cex.main=0.95, font.lab=2, font=2)
max_interval <- which.max(time_series) #queries time series for max steps
names(max_interval) #gives the 5-min interval with max steps
sum(is.na(activity)) #calculates the total number of missing values

```

Next to give more accurate assessment for time-series analyses I created a function to impute data for the steps.  This is a randomized function that generates data points within a subset of already present points.  See below.  Also see Statistics reference site of Columbia University: http://www.stat.columbia.edu/~gelman/arm/missing.pdf

```{r}
#call the function random.imp to make random values to add to the missing data points in steps

source("~/Documents/Reproducible Research/imputeValues.R")
#R function to imput missing values for daily steps analysis
random.imp <- function (a){
    missing <- is.na(a)
    n.missing <- sum(missing)
    a.obs <- a[!missing]
    imputed <- a
    imputed[missing] <- sample(a.obs, n.missing, replace=TRUE)
    return(imputed)
}

```

After the function is called, the data can be appended as a new column to the dataframe.  This allows you to analyze the same data set using imputed data.  I also report the new mean and median based on the imputed data.

```{r}
df.steps <- random.imp(activity$steps) #now you have data to add to missing points
activity$imputed = paste(random.imp(activity$steps)) #pasted a column into the dataframe with the randomly imputed data based on original steps data

####repeat of analysis with imputed data
activity$imputed <- as.numeric(activity$imputed)
StepsTotal <- aggregate(imputed ~ date, data=activity, sum, na.rm=TRUE)
hist(x=StepsTotal$imputed, col = "blue", breaks=50, xlab="Steps Per Day + Imputed", main="Daily Number of Steps with Imputed Values", cex=1, cex.lab=0.75, cex.axis=0.75, cex.main=0.95, cex.sub=0.75, font=2, font.lab=2)
summary(StepsTotal$imputed)

```

Next, using the real and imputed data, we determine a time-series for the number of steps taken on average on the weekend versus the weekday by this individual.

```{r}
activity$date <- as.Date(activity$date) #now you need to change activity date to POSIXt date object
activity$date <- strptime(paste(activity$date), format="%Y-%m-%d", tz="UTC")
activity$weekday <- paste(weekdays(activity$date)) #add the day of the week based on time-stamp
weekdays <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
activity$weekday = as.factor(ifelse(is.element(weekdays(as.Date(activity$date)), weekdays), "Weekday", "Weekend"))
StepsImputedInterval <- aggregate(imputed ~ interval + weekday, activity, mean)
library(lattice)
xyplot(StepsImputedInterval$imputed ~ StepsImputedInterval$interval | StepsImputedInterval$weekday, main="Average Steps (Imputed) per Day by Interval", xlab="Interval (5 seconds each)", ylab="Imputed Steps", layout=c(1,2), type="l", cex=1, cex.axis=0.75, font=2, font.lab=2, font.main=2, font.sub=2, font.lab=2)
```

