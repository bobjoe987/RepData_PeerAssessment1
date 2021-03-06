---
title: "PA1_template"
author: "Dave Walters"
date: "June 14, 2015"
output:
  html_document:
    self_contained: no
---

Reproducible Research Project 1 PA1_template
==============================================
###Prerequisites
```{r, echo=TRUE}
library(dplyr)
library(ggplot2)
setClass("myDate")
setAs("character","myDate",function(from) format(strptime(from,format="%Y-%m-%d"),"%Y-%m-%d"))
cls <- c("numeric","myDate","numeric")
```

#Loading and preprocessing the data
1. Load the data (i.e. read.csv())
```{r, echo=TRUE}
data <- read.csv('./activity.csv',header=TRUE,colClasses=cls)
```
2. Process/transform the data (if necessary) into a format suitable for your analysis
```{r, echo=TRUE}
for (i in 1:5) { print("see prerequisites section....")}
```


#What is mean total number of steps taken per day?
1. Calculate the total number of steps taken per day
```{r, echo=TRUE}
sum_steps_by_date <- data %>% group_by(date) %>% summarize(sum=sum(steps,na.rm=TRUE))
```
2. Make a histogram of the total number of steps taken each day 
```{r, echo=TRUE, tidy=FALSE}
#base plotting
par(mfrow=c(1,1),las=1)
hist(sum_steps_by_date$sum,xlab="Sum of Steps Taken per Day",main="Summary of Steps per Day")
#ggplot
ggplot(sum_steps_by_date,aes(x=sum)) + geom_histogram(fill="blue",binwidth=750) + labs(y=expression("Freq")) + labs(x=expression("Sum Steps per Day")) + labs(title=expression("Steps per Day"))
```
3. Calculate and report the mean and median of the total number of steps taken per day
```{r, echo=TRUE}
mean_steps_by_day <- data %>% group_by(date) %>% summarize(mean=mean(steps)) %>% select(mean)
median_steps_by_day <- data %>% group_by(date) %>% summarize(median=median(steps)) %>% select(median)
report_table <- data %>% group_by(date) %>% count(date) %>% select(date)
report_table <- cbind(report_table,mean_steps_by_day)
report_table <- cbind(report_table,median_steps_by_day)
```


#What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
```{r, echo=TRUE}
avg_steps_by_interval <- data %>% group_by(interval) %>% summarise(average=mean(steps,na.rm=TRUE))
plot(avg_steps_by_interval$average,type="l")
```
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r, echo=TRUE}
avg_steps_by_interval[which.max(avg_steps_by_interval$average),]
```


#Imputing missing values
1. Calculate and report the total number of missing values in the dataset
```{r, echo=TRUE}
length(which(is.na(data) == TRUE))
```
2. Devise a strategy for filling in all of the missing values in the dataset
```{r, echo=TRUE}
#create a new data table where the averages of each interval are calulated across all days
mean_steps_by_interval <- data %>% group_by(interval) %>% summarize(mean=mean(steps,na.rm=TRUE)) %>% select(interval,mean)
#join the existing table and the mean table by the interval creating a new column that has the average steps for that interval
new <- inner_join(data,mean_steps_by_interval,by="interval")
#loop through the new table and if any value in the 1st column is NA, updated it to be the value of the 4th, the average interval
for (i in 1:dim(new)[1]) { 
  if (is.na(new[i,1]) == TRUE) { 
    new[i,1] <- round(new[i,4])
  } 
}
```
3. Create a new dataset that is equal to the original dataset but with the missing data filled in
```{r, echo=TRUE}
new <- new %>% select(steps,date,interval)
```
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day
```{r, echo=TRUE}
#new summary table
new_sum_steps_by_date <- new %>% group_by(date) %>% summarize(sum=sum(steps,na.rm=TRUE))
#plot package
hist(new_sum_steps_by_date$sum,xlab="Sum of Steps Taken per Day")
#ggplot package
ggplot(new_sum_steps_by_date,aes(x=sum)) + geom_histogram(fill="blue",binwidth=750) + labs(y=expression("Freq")) + labs(x=expression("Sum Steps per Day")) + labs(title=expression("Steps per Day"))
#calculate and report mean and median of new dataset
new_mean_steps_by_day <- new %>% group_by(date) %>% summarize(mean=mean(steps)) %>% select(mean)
new_median_steps_by_day <- new %>% group_by(date) %>% summarize(median=median(steps)) %>% select(median)
new_report_table <- new %>% group_by(date) %>% count(date) %>% select(date)
new_report_table <- cbind(new_report_table,new_mean_steps_by_day)
new_report_table <- cbind(new_report_table,new_median_steps_by_day)
```


#Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels ??? ???weekday??? and ???weekend??? indicating whether a given date is a weekday or weekend day
```{r, echo=TRUE}
new$wday <- as.factor(ifelse(weekdays(as.Date(new$date)) %in% c("Saturday","Sunday"), "Weekend", "Weekday"))
```
2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)
```{r, echo=TRUE}
ggplot(new,aes(x=interval,y=steps)) + geom_line() + facet_wrap(~wday,nrow=2,ncol=1)
```

##write knitr file
knitr2html("PA1_template.Rmd","PA1_template.html")
