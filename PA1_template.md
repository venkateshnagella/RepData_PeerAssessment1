## Reproducible Research : Peer Assessment 1
Venkatesh

### Loading and preprocessing the data

```r
library(lattice)
unzip("activity.zip")
data <- read.csv("activity.csv", header = T, colClasses = c("numeric", "Date", "numeric"), stringsAsFactors = FALSE)
```



### What is mean total number of steps taken per day?

To find the mean total number of steps we have to sum the number of steps of each day excluding NAs

```r
## Takes out the NA values
nonNAdata <- data[complete.cases(data[, c("steps")]), ] 

## find total number of steps by day
stepsbyday <- tapply (nonNAdata$steps, nonNAdata$date, FUN = function ( x ) sum ( x ), simplify = TRUE ) 

## make a histogram
histogram ( ~ stepsbyday , main = "Distribution of total numbers of steps \n taken each day" , xlab = "Steps by Day", ylab = "Frequency") 
```

<img src="figure/unnamed-chunk-2.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" style="display: block; margin: auto;" />

```r
## find the mean
mean(stepsbyday) 
```

```
## [1] 10766
```

```r
## find the median
median(stepsbyday) 
```

```
## [1] 10765
```



### What is the average daily activity pattern

To compute the mean number of steps taken at each interval, the number of steps are averaged for a given interval for all days

```r
# the 5-min interval axis
five.min.interval <- unique ( nonNAdata$interval )

# applys the mean for each interval (over all days)
average.daily <- tapply ( nonNAdata$steps, nonNAdata$interval, FUN = function ( x ) mean ( x ) , simplify = TRUE )
max.step <- max ( average.daily )

## Plotting the graph
xyplot ( average.daily ~ five.min.interval, main = "average number of steps \n over all days for each interval" , xlab = "5-min interval", ylab = "average number of steps taken, \n averaged across all days", type="l")
```

<img src="figure/unnamed-chunk-3.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" style="display: block; margin: auto;" />

```r
## finding the interval containing the most steps taken
five.min.interval [ average.daily == max.step ]
```

```
## [1] 835
```



### Imputing missing values

The total number of missing values is 2304. The missing values are imputed here with the average number of steps of a given 5-minute interval over all days. The new mean is 10766. The new median is 10766. 


```r
## finding the total number of missing values
length(data[is.na(data$steps), "steps"]) 
```

```
## [1] 2304
```

```r
## repalces NA's with mean steps across all days for each interval
stepswithNAfilled <- replace(data$steps, is.na(data$steps), average.daily) 

## constructs a new data set with NA's replaced with interval mean steps
newdata <- data
newdata$steps <- stepswithNAfilled

## calculate steps per day
totalstepsperdaywithna <- tapply (newdata$steps , newdata$date , FUN = function ( x ) sum ( x ), simplify = TRUE ) 

## make the histogram
histogram ( ~ totalstepsperdaywithna, main = "steps per day with filled na", xlab = "steps per day with filled na") 
```

<img src="figure/unnamed-chunk-4.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" style="display: block; margin: auto;" />

```r
## mean of steps per day with Na imputed
mean(totalstepsperdaywithna) 
```

```
## [1] 10766
```

```r
## median of steps per day with NA imputed
median(totalstepsperdaywithna) 
```

```
## [1] 10766
```



### Are there differences in activity patterns between weekdays and weekends?

```r
# get column of weekday and weekend
dates <- weekdays (  as.POSIXlt( newdata$date ) )
dates [ dates == "Saturday" | dates == "Sunday" ] <- "weekend"
dates [ dates != "weekend" ] <- "weekday"
newdata$day <- dates

# Calculation of mean for all days in weekday and weekend
mean_days <- tapply ( newdata$steps, list ( newdata$day, newdata$interval ), FUN = function ( x ) mean ( x ), simplify = FALSE )

# formating data for plotting
mean_days <- t ( as.data.frame ( mean_days ) )
temp <- list ( interval = as.numeric ( row.names ( mean_days ) ), weekday = as.numeric ( mean_days [ , "weekday" ] )  , weekend = as.numeric ( mean_days [ , "weekend"] ) )

library(reshape2)
data2 <- melt ( data.frame ( temp ), id.vars = "interval", value.name = "Number.of.steps" )

# generating the plot
xyplot ( Number.of.steps  ~ interval | variable, data2, layout = c(1,2), main = "Average number of steps \n over all days for each interval", xlab = "Interval", ylab = "Nnumber of steps", type="l")
```

<img src="figure/unnamed-chunk-5.png" title="plot of chunk unnamed-chunk-5" alt="plot of chunk unnamed-chunk-5" style="display: block; margin: auto;" />
