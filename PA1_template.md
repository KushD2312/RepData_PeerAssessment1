# Peer graded assignment 1 for Reproducible Research

## First we have to load tha data in the file activity.zip


```{r}
library(plyr)
library(lattice)
unzip('activity.zip')
data <- read.csv('activity.csv')
head(data)
```

## For finding the mean total number of steps

The sum of steps for each day is found and the days where the steps are missing is ignored:
```{r}
steps_per_day <- ddply(data[!is.na(data$steps),], .(date), summarize, steps = sum(steps))
head(steps_per_day)
```
The corresponding histogram:
```{r}
histogram(steps_per_day$steps, xlab = 'Steps', main = 'Total steps per day', breaks = 10)
```


## What is the average daily activity pattern?

The mean number of steps for each interval across all days is found and the days when steps are missing are ignored:
```{r}
intervalAvg <- ddply(data[!is.na(data$steps),], .(interval), summarize, steps = mean(steps))
head(intervalAvg)
```
The corresponding time series plot:
```{r}
xyplot(steps ~ interval, data = intervalAvg, type='l', 
     xlab = '5-minute Interval', ylab = 'Steps', main = 'Average steps')
```


## Imputing missing values

The number of rows in the dataset with number of steps missing is:
```{r}
sum(is.na(data$steps))
```
```{r}
imputed <- data
imputed[is.na(imputed$steps),]$steps <- intervalAvg$steps
head(imputed)
```
The histogram for the total number of steps per day using the imputed data:
```{r}
stepsPerDay <- ddply(imputed, .(date), summarize, steps = sum(steps))
histogram(stepsPerDay$steps, xlab = 'Steps', main = 'Total steps (No missing values)', breaks = 10)
```
The new mean and median:
```{r}
mean(stepsPerDay$steps)
median(stepsPerDay$steps)
```
The mean remains the same as earlier, but the median increases slightly.  
Replacing several values in the dataset with interval means results in the median becoming equal to the mean.  
Likewise, the histogram also shows a noticable increase in frequency of days with total number of steps around the median.

## Are there differences in activity patterns between weekdays and weekends?

A new column is added to the imputed dataset, factoring it into weekdays and weekends. Saturday and Sunday are taken to be weekends.
```{r}
imputed$day <- factor(weekdays(as.Date(imputed$date)))
levels(imputed$day)
levels(imputed$day) <- c('weekday','weekday','weekend','weekend','weekday','weekday','weekday')
```
The mean number of steps for each interval interval across all weekdays and all weekends separately is obtained:
```{r}
weekSteps <- ddply(imputed, .(interval, day), summarize, steps = mean(steps))
head(weekSteps)
```
The corresponding time series plot:
```{r}
xyplot(steps ~ interval | day, data = weekSteps, type='l', layout = c(1, 2), 
       xlab = 'Interval', ylab='Number of steps', main = 'Average steps by day')
```
The plot shows comparitively higher activity in mornings during weekdays, and higher activity at midday and evenings during weekends.  

**END**