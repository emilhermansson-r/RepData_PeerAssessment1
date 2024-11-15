---
# Course project 1 Reproducable Research

## Loading and preprocessing the data
```{r echo = TRUE}
#Setting WD
setwd("/Users/emilhermansson/Desktop/RkursJH/Project")

#Loading packages
library(ggplot2)

#Downloading csv
download.file('https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip', destfile = 'DATA.zip', method = "curl")

#Unzip data
unzip("DATA.zip", exdir = "/Users/emilhermansson/Desktop/RkursJH/Project/Datacp1")

#Reading data
data <- read.csv("/Users/emilhermansson/Desktop/RkursJH/Project/Datacp1/activity.csv")

#Summarizing data
summary(data)
```      

## Calculating the mean number of steps per day
```{r echo = TRUE}
#aggregating data
steps_day  <- aggregate(steps ~ date, data = data, sum)

ggplot(steps_day,aes(steps)) +
  geom_histogram(binwidth = 2000, fill = "white", color = "black") +
  ylab("Frequency") +
  xlab("Number of steps per day")
  
#calculating mean and median steps per day
mean_steps <- mean(steps_day$steps)
median_steps <- median(steps_day$steps)

print(paste("mean value of steps per day is",mean_steps, "while the median value is", median_steps,"steps per day."))
```      

## What is the average daily activity pattern?

```{r echo = TRUE}
average_interval <- aggregate(steps ~ interval, data = data, mean)

#plotting
plot(x = average_interval$interval, y = average_interval$steps, type = "l", xlab = "5 minute interval", ylab = "avarage number of steps", main = "Avarage steps across a day") 

#maximum 
max_steps <- which.max(average_interval$steps)
interval <- average_interval$interval[max_steps]

#printing results
print(paste0("The maximum number of steps is taken at the 5 minute interval at ", substring(interval,1,1),":",substring(interval,2,3)))
```    

## Imputing missing values
```{r echo = TRUE}

#calculating missing values
nofmissing <- nrow(data[is.na(data$steps) == TRUE,])

#filling missing values with avarage for the interval
data$steps <- ifelse(is.na(data$steps), average_interval$steps, data$steps)

#calculate daily avarage
day_aggr <- aggregate(steps ~ date, data = data, sum)

#median and mean
mean_day <- mean(day_aggr$steps)
median_day <- median(day_aggr$steps)
print("Since the missing value was filled with means per interval it does not affect the mean value overall. However, the median was affected and is now equal to the mean. ")

```

## Are there differences in activity patterns between weekdays and weekends?

```{r echo = TRUE}
#Converting string to date format
data$date <- date <- as.Date(data$date, format = "%Y-%m-%d")

#Creating variable that tells weekday 
data$day <- weekdays(data$date)

#Creating variable that tells if it is weekend or weekday
data$daytype <- ifelse(data$day == "Saturday" | data$day == "Sunday", "Weekend", "Weekday" )

#Aggregating 
steps_interval_daytype  <- aggregate(steps ~ interval + daytype, data = data, mean)

ggplot(steps_interval_daytype,aes(interval, steps )) +
  geom_line() +
  ylab("Frequency") +
  xlab("Number of steps per day")
  
  
#Creating subsets for weekday and weekend
steps_weekday <- steps_interval_daytype[steps_interval_daytype$daytype == "Weekday",]
steps_weekend <- steps_interval_daytype[steps_interval_daytype$daytype == "Weekend",]

#creating space for two rows of graphs
par(mfrow = c(2,1), mar = c(4,4,2,2))

#creating two plots depending on if it is weekday or weekend
plot(steps_weekday$interval, steps_weekday$steps, col = "blue", main = "Weekday", type = "l", xlab = " ", ylab = "Steps")
plot(steps_weekend$interval, steps_weekend$steps, col = "red", main = "Weekend", type = "l", xlab = "Interval", ylab = "Steps")
```
