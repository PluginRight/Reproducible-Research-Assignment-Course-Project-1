---
title: "Reproducible Research Assignment 1"
author: "Samuel Danilola"
date: "22/02/2022"
output: html_document
---

```{r setup, include=FALSE}
library(knitr)
knitr::opts_chunk$set(echo = TRUE)
```

## R Markdown

This is an assignment that uses a personal activity monitoring data. The device collects data on an individuals activity at 5-minute interval all throught the day. he data consists of two months of data from an anonymous individual collected during the months of October and November, 2012.

This document is my attempt at responding to the Project Assignment 1 in the Coursera Reproducible Research course.

*LOAD THE REQUIRED PACKAGES*

```{r echo = TRUE} 
library(dplyr)
library(lubridate)
library(ggplot2)
```

*LOAD THE DATA*

```{r echo = TRUE}
activity <- read.csv("activity.csv", header = TRUE, sep = ',', colClasses = c("numeric", "character", "integer"))
```
*CONVERT DATE COLUMN TO DATE*

```{r echo = TRUE}
activity$date <- ymd(activity$date)
```

1. Calculating the mean total numberof steps taken per day

```{r echo = TRUE}
steps <- activity %>%
  filter(!is.na(steps)) %>%
  group_by(date) %>%
  summarize(steps = sum(steps)) %>%
  print

```
2. Constructing the histogram of steps taken per day

```{r echo = TRUE} 
ggplot(steps, aes(x = steps)) + geom_histogram(fill = "blue", binwidth = 1000) + labs(title = "Histogram of Steps per day", x = "Steps per day", y = "Frequency")
```

3. Mean and Median of the Total Number of Steps per Day

```{r echo = TRUE}
mean_steps <- mean(steps$steps, na.rm = TRUE)
median_steps <- median(steps$steps, na.rm = TRUE)

mean_steps
median_steps

```

*Average Daily Activity Pattern* 

1a. Calculate the average steps 

```{r echo = TRUE}

interval <- activity %>%
  filter(!is.na(steps)) %>%
  group_by(interval) %>%
  summarize(steps = mean(steps))

```

1b. Plot the graph
```{r echo = TRUE}

ggplot(interval, aes(x=interval, y=steps)) +
  geom_line(color = "blue") + labs(title = "Graph of Interval Against Average Steps Taken")

```

2. Interval with maximum number of steps

```{r echo = TRUE}

interval[which.max(interval$steps),]

```

*Imputing missing values*

```{r echo = TRUE}

activity_full <- activity
nas <- is.na(activity_full$steps)
avg_interval <- tapply(activity_full$steps, activity_full$interval, mean, na.rm=TRUE, simplify=TRUE)
activity_full$steps[nas] <- avg_interval[as.character(activity_full$interval[nas])]

```
**Calculating the steps after inputting missing values**

```{r echo = TRUE}
steps_full <- activity_full %>%
  filter(!is.na(steps)) %>%
  group_by(date) %>%
  summarize(steps = sum(steps)) %>%
  print

```

**Drawing the histogram**

```{r echo = TRUE} 
ggplot(steps_full, aes(x = steps)) + geom_histogram(fill = "blue", binwidth = 1000) + labs(title = "Histogram of Steps including missing values per day", x = "Steps per day", y = "Frequency")
```
**Mean and Median of Full Data**

```{r echo = TRUE}

mean_steps_full <- mean(steps_full$steps)
median_steps_full <- median(steps_full$steps)

mean_steps_full
median_steps_full

```

After inputting the missing the values, the mean remained the same, hwover the median increased slightly and became equal to the mean.

*DIFFERENCES BETWEEN ACTIVITY PATTERNS OF WEEKDAYS AND WEEKENDS*

1.Inputting weekdays or weekends

```{r echo = TRUE}

activity_full <- mutate(activity_full, weektype = ifelse(weekdays(activity_full$date) == "Saturday" | weekdays(activity_full$date) == "Sunday", "weekend", "weekday"))
activity_full$weektype <- as.factor(activity_full$weektype)

```

2. Calculating the average steps and drawing the plot

```{r}

interval_full <- activity_full %>%
  group_by(interval, weektype) %>%
  summarise(steps = mean(steps))

ggplot(interval_full, aes(x=interval, y=steps, color = weektype)) +
  geom_line() +
  facet_wrap(~weektype, ncol = 1, nrow=2)

```

