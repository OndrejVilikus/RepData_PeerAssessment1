---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document: 
    keep_md: true
---


## 1 Reading in the dataset

```r
data<-read.csv("activity.csv")
```
## 2 Histogram of the total number of steps taken each day

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
perDay <- data %>%
  group_by(date) %>%
  summarise(totSteps = sum(steps))

hist(perDay$totSteps, density=30, breaks=20,col="black",  
     xlab="Steps per day", ylab="",  
     main="Total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

## 3 Mean and median number of steps taken each day
plot

```r
# Mean number of steps taken each day
mean(perDay$totSteps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
# Median number of steps taken each day
median(perDay$totSteps, na.rm = TRUE)
```

```
## [1] 10765
```

## 4 Time series plot of the average number of steps taken

```r
perInterval <- data %>%
  group_by(interval) %>%
  summarise(meanSteps = mean(steps,na.rm = TRUE))
perInterval$hm<-paste(as.character((perInterval$interval-perInterval$interval%%100)/100),":",as.character(perInterval$interval%%100),sep="")
perInterval$time<-as.POSIXlt(perInterval$hm,format="%H:%M")

plot(perInterval$time,perInterval$meanSteps,type="l",
     main="Average number of steps taken by interval",
     xlab="",ylab="Average number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

## 5 The 5-minute interval that, on average, contains the maximum number of steps



```r
perInterval[perInterval$meanSteps==max(perInterval$meanSteps),c("hm","meanSteps")]
```

```
## # A tibble: 1 × 2
##   hm    meanSteps
##   <chr>     <dbl>
## 1 8:35       206.
```

## 6 Code to describe and show a strategy for imputing missing data


```r
#total number of missing values in the dataset:
sum(is.na(data$steps)) 
```

```
## [1] 2304
```

As the days are relatively similar, but we see pattern across each day, we can replace missing values with mean for the interval.


```r
library(ggplot2)

# Heatmap 
ggplot(data, aes(as.POSIXct(date), interval, fill= steps)) + 
  geom_tile()
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->
### Create a new dataset with the missing data filled in.

```r
newdata<-data
for (row in 1:nrow(data)){
  if(is.na(data$steps[row])) {
    newdata$steps[row]<-mean(data$steps[data$interval==data$interval[row]],na.rm=TRUE)    
  }
  else {
    newdata$steps[row]<-data$steps[row]
  }
}
```

### Missing numbers check

```r
#total number of missing values in the new dataset:
sum(is.na(newdata$steps)) 
```

```
## [1] 0
```

### Check result with heatmap 

```r
ggplot(newdata, aes(as.POSIXct(date), interval, fill= steps)) + 
  geom_tile()
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

## 7 Histogram of the total number of steps taken each day after missing values are imputed


```r
newperDay <- newdata %>%
  group_by(date) %>%
  summarise(totSteps = sum(steps))

hist(newperDay$totSteps, density=30, breaks=20,col="black",  
     xlab="Steps per day", ylab="",  
     main="Total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->
 

```r
# Mean number of steps taken each day
mean(newperDay$totSteps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
# Median number of steps taken each day
median(newperDay$totSteps, na.rm = TRUE)
```

```
## [1] 10766.19
```

The histogram has new peak where the average day would be. The mean value does not change by adding mean values, but median is now the same as there are lots of days with average number of steps.

## 8 Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

We can see that the morning peak only happens on weekdays, but on weekends, the number of steps tends to be higher over the day.


```r
library(lattice)

isweekend<-weekdays.POSIXt(as.POSIXct(newdata$date))%in%c("Saturday","Sunday")
newdata$dayType<-as.factor(ifelse(isweekend,"weekend","weekday"))

newPerInterval <- newdata %>%
  group_by(interval,dayType) %>%
  summarise(meanSteps = mean(steps,na.rm = TRUE))
```

```
## `summarise()` has grouped output by 'interval'. You can override using the
## `.groups` argument.
```

```r
xyplot(newPerInterval$meanSteps~newPerInterval$interval|newPerInterval$dayType,
       type="l",
       layout=c(1,2),
       xlab="Interval",
       ylab="Number of steps"
       )
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

