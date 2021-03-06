# Reproducible Research: Peer Assessment 1

---
title: "Reproducible Research: Peer Assessment 1"
author: "EdGomes"
date: "1 de outubro de 2017"
output: html_document
keep_md: true
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
***

##Guide

1. _Introduction_

2. _Loading and preprocessing the data Load the data (i.e. read.csv()) Process/transform the data (if necessary) into a format suitable for your analysis_

3. _Loading and preprocessing the data_

4. _What is mean total number of steps taken per day?_

5. _What is the average daily activity pattern?_

6. _Imputing missing values_

7. _Are there differences in activity patterns between weekdays and weekends?_

***

## Introduction

This project satisfies the Johns Hopkins Reproducible Research course offered through Coursera, Project 1 requirements. It loads a data set, performs some processing, and produces some charts. The data involved concerns the number of steps taken as recorded by a Fitbit or other similar device.

***

##Loading and preprocessing the data

The data is in an unzipped file named ["activity.csv"](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip). If it has not already been loaded, it is loaded into memory.


```{r}
ArqACT <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
destACT<-"~/ACT.zip"
download.file(ArqACT,destACT, mode = "wb")

```

_Unzip Activity.zip_
```{r}
if(!file.exists('activity.csv')){
   unzip("~/ACT.zip")
}
```

_Get the list of the files Activity.csv and get rid of rows containing missing values and save the subset to a new data frame **LessNA**_
```{r}
ACT <- read.csv("activity.csv")
ACT$date<- as.Date(ACT$date)
LessNA<- subset(ACT, !is.na(ACT$steps))
head(ACT)
```

***

## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.


1.Calculate the total number of steps taken per day
```{r}
stepsDay <- tapply(ACT$steps, ACT$date, sum, na.rm=TRUE)
```


2.If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day.
```{r}
library("ggplot2")
qplot(stepsDay, xlab="Total Steps Per Day", ylab="Frequency Using Binwith 500", binwidth=500)
```

3.Calculate and report the mean and median of the total number of steps taken per day.

```{r}
Mean <- mean(stepsDay)
Median <- median(stepsDay)
```

* _**Mean** = 9354.23_

* _**Median** = 10395_

***

## What is the average daily activity pattern?

```{r}
average <- aggregate(x=list(meanSteps=ACT$steps), by=list(interval=ACT$interval), FUN=mean, na.rm=TRUE)
```


1.Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```{r}
ggplot(data=average, aes(x=interval, y=meanSteps)) +
    geom_line(colour ="dark red", size = 1) +
    xlab("5-minute interval") +
    ylab("average number of steps taken") 
```


2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r}
Max <- which.max(average$meanSteps)
Time <-  gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2", average[Max,'interval'])
```

* _Average contains the maximum number of steps is **8:35**_

***
***

## Imputing missing values

1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```{r}
NAs <- length(which(is.na(ACT$steps)))
```
* _The total number of missing values is **2304**_

2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3.Create a new dataset that is equal to the original dataset but with the missing data filled in.

```{r}
ACTFilled <- ACT
NAFilled <- is.na(ACTFilled$steps)
average <- tapply(LessNA$steps, LessNA$interval, mean, na.rm=TRUE, simplify=T)
ACTFilled$steps[NAFilled] <- average[as.character(ACTFilled$interval[NAFilled])]
summary(ACTFilled)
```

4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```{r}
NEWstepsDay <- tapply(ACTFilled$steps, ACTFilled$date, sum, na.rm=TRUE, simplify=T)
qplot(NEWstepsDay, xlab="Daily Steps", ylab="Frequency", main="The Total Number of Steps Taken Each Day", binwidth=500)
```


```{r}
New_Mean <- mean(NEWstepsDay)
New_Median <- median(NEWstepsDay)
```

* _**New Mean** = 10766.19_

* _**New Median** = 10766.19_

***

## Are there differences in activity patterns between weekdays and weekends?

_For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part._

1.Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```{r}
wday <- function(d) {
    w <- weekdays(d)
    ifelse(w == "sábado"| w== "domingo", "weekend", "weekday")
}
week <- sapply(ACTFilled$date, wday)
ACTFilled$week  <- as.factor(week)
head(ACTFilled$week)

```


***

2.Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```{r}
DIA <- aggregate(steps ~ week+interval, data=ACTFilled, FUN=mean)

library(lattice)
xyplot(steps ~ interval | factor(week),
       layout = c(2, 1),
       xlab="Interval",
       ylab="Number of steps",
       type="l",
       lty=2,
       data=DIA)

```

