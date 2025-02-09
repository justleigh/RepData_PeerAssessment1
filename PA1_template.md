Activity Monitoring Project
================
Leigh Pearson
2023-10-20

This assignment makes use of data from a personal activity monitoring
device.

The variables included in this dataset are:

**steps:** Number of steps taking in a 5-minute interval (missing values
are coded as NA)

**date:** The date on which the measurement was taken in YYYY-MM-DD
format

**interval:** Identifier for the 5-minute interval in which measurement
was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset.

## Data Loading

The first step is to read the date and assign it to a variable:

``` r
setwd("/Users/leighpearson/RStudio Practice/Activity Monitoring Project/RepData_PeerAssessment1")
activity_data <- read.csv("activity.csv")
```

Then we check to see the structure

``` r
head(activity_data)
```

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

## Total Number of Steps Taken Per Day

Now, we need to complete the first analysis task: Calculate the total
number of steps taken per day, ignoring the NAs

``` r
daily_steps <- aggregate(steps ~ date, activity_data, sum, na.rm=TRUE)
```

Next, we need to make a histogram of the total number of steps taken
each day. For this I will use ggplot…

``` r
library(ggplot2)
```

    ## Warning: package 'ggplot2' was built under R version 4.3.1

``` r
ggplot(daily_steps, aes(x=steps)) +
  geom_histogram(fill="blue", color="black", binwidth=1000) +
  labs(title="Histogram of Total Steps Taken Each Day",
       x="Number of Steps",
       y="Frequency") +
  theme_minimal()
```

![](PA1_template_figures/histogram%20steps%20per%20day.png)<!-- -->

Next, we need to calculate and report the mean and median of the total
number of steps taken per day

``` r
mean_steps <- mean(daily_steps$steps)
median_steps <- median(daily_steps$steps)
mean_steps
```

    ## [1] 10766.19

``` r
median_steps
```

    ## [1] 10765

The calculated mean number of steps taken per day is 1.0766^{4} and the
median is 1.0765^{4}.

I thought it might also be useful to add these to the previous
histogram, but the result is unclear as the mean and median are
essentially the same

``` r
ggplot(daily_steps, aes(x=steps)) +
  geom_histogram(aes(y=after_stat(density)), fill="blue", color="black", binwidth=1000) +
  geom_vline(aes(xintercept=mean_steps, color="Mean"), linetype="dashed", linewidth=1) +
  geom_vline(aes(xintercept=median_steps, color="Median"), linetype="dashed", linewidth=1) +
  geom_text(aes(x=mean_steps, label=paste("Mean = ", round(mean_steps))), y=0.02, vjust=-1, color="red", size=4) +
  geom_text(aes(x=median_steps, label=paste("Median = ", round(median_steps))), y=0.02, vjust=-1, color="green", size=4) +
  labs(title="Histogram of Total Steps Taken Each Day",
       x="Number of Steps",
       y="Density") +
  theme_minimal() +
  scale_color_manual(name="Statistics", values=c("Mean"="red", "Median"="green")) +
  guides(color=guide_legend(title=NULL))
```

![](PA1_template_figures/histogram%20steps%20per%20day%20+%20mean%20and%20median-1.png)<!-- -->

## Average Daily Activity Pattern

Next we have to calculate the average daily activity pattern by making a
time series plot (i.e.  type = “l”) of the 5-minute interval (x-axis)
and the average number of steps taken, averaged across all days
(y-axis). To do this we need to aggregate the data to calculate the mean
number of steps for each 5-minute interval across all days and then plot
in ggplot. However, I also realised that some intervals may contain only
NAs, in which case 0 is used.

``` r
# Aggregate data to calculate mean steps for each 5-minute interval
# Calculate mean steps for each interval, ignoring NAs
interval_means <- aggregate(steps ~ interval, activity_data, mean, na.rm=TRUE)

# Replace NA means (intervals with only NA values) with 0
interval_means$steps[is.na(interval_means$steps)] <- 0

# Create the time series plot
ggplot(interval_means, aes(x=interval, y=steps)) +
  geom_line() +
  labs(title="Average Number of Steps Taken in 5-minute Intervals",
       x="5-minute Interval",
       y="Average Number of Steps") +
  theme_minimal()
```

![](PA1_template_figures/histogram%20steps%20per%20day.png)<!-- -->

Next we need to calculate which 5-minute interval, on average across all
the days in the dataset, contains the maximum number of steps.

``` r
max_interval <- interval_means$interval[which.max(interval_means$steps)]
max_interval
```

    ## [1] 835

## Imputing Missing Values

There are a number of days/intervals where there are missing values
(coded as NA). The presence of missing days may introduce bias into some
calculations or summaries of the data. This section of the assignment
consists of 4 tasks:

1.  Calculate and report the total number of missing values in the
    dataset (i.e. the total number of rows with NAs)

2.  Devise a strategy for filling in all of the missing values in the
    dataset. The strategy does not need to be sophisticated. For
    example, you could use the mean/median for that day, or the mean for
    that 5-minute interval, etc.

3.  Create a new dataset that is equal to the original dataset but with
    the missing data filled in.

4.  Make a histogram of the total number of steps taken each day and
    Calculate and report the mean and median total number of steps taken
    per day. Do these values differ from the estimates from the first
    part of the assignment? What is the impact of imputing missing data
    on the estimates of the total daily number of steps?

### Missing Values Solution 1

``` r
missing_values <- sum(is.na(activity_data$steps))
print(paste("Total number of missing values:", missing_values))
```

    ## [1] "Total number of missing values: 2304"

### Missing Values Solution 2

For solution 2, I chose to use the mean for that 5-minute interval
across all days to fill in missing values, but I also allowed for the
fact that some intervals may contain only NAs, in which case 0 would be
used.

### Missing Values Solution 3

``` r
# Calculate mean for each interval
interval_means <- aggregate(steps ~ interval, activity_data, mean, na.rm=TRUE)

# If any interval mean is NA, replace it with 0
interval_means$steps[is.na(interval_means$steps)] <- 0

# Merge the original data with the interval means
activity_data_filled <- merge(activity_data, interval_means, by="interval", suffixes=c("", ".mean"))

# Replace NA values in the original data with the calculated interval means
activity_data_filled$steps[is.na(activity_data_filled$steps)] <- activity_data_filled$steps.mean[is.na(activity_data_filled$steps)]

# Clean up the data frame
activity_data_filled <- activity_data_filled[, c("interval", "date", "steps")]
```

### Missing Values Solution 4 (New Histogram with Imputed Missing Values)

``` r
# Calculate total steps taken per day
daily_steps <- aggregate(steps ~ date, activity_data_filled, sum)

# Plot histogram
ggplot(daily_steps, aes(x=steps)) +
  geom_histogram(fill="blue", color="black", binwidth=1000) +
  labs(title="Histogram of Total Steps Taken Each Day (Imputed Data)",
       x="Number of Steps",
       y="Frequency") +
  theme_minimal()
```

![](PA1_template_figures/Histogram%20of%20Daily%20Steps%20with%20Imputed%20Data.png)<!-- -->

``` r
# Calculate and display mean and median
mean_steps_imputed <- mean(daily_steps$steps)
median_steps_imputed <- median(daily_steps$steps)

print(paste("Mean number of steps taken per day (with imputed data):", round(mean_steps_imputed)))
```

    ## [1] "Mean number of steps taken per day (with imputed data): 10766"

``` r
print(paste("Median number of steps taken per day (with imputed data):", round(median_steps_imputed)))
```

    ## [1] "Median number of steps taken per day (with imputed data): 10766"

## Weekday and Weekend Differences in Activity Patterns

### New Weekday and Weekend Factor Variables

``` r
# Adding a new factor variable 'day_type'
activity_data_filled$day_type <- ifelse(weekdays(as.Date(activity_data_filled$date)) %in% c("Saturday", "Sunday"), "weekend", "weekday")

# Convert 'day_type' to factor
activity_data_filled$day_type <- as.factor(activity_data_filled$day_type)
```

## Panel Plot for Weekday vs Weekend

``` r
# Calculate the average steps per interval for both weekdays and weekends
avg_steps_interval <- aggregate(steps ~ interval + day_type, activity_data_filled, mean)

# Panel plot
p <- ggplot(avg_steps_interval, aes(x=interval, y=steps, group=day_type)) +
  geom_line(aes(color=day_type), linewidth=1) +
  facet_grid(day_type ~ .) +
  labs(title="Average Number of Steps Taken per 5-minute Interval",
       x="5-minute Interval",
       y="Average Number of Steps") +
  theme_minimal()

print(p)
```

![](PA1_template_figures/Weekday%20Weekend%20Average%20Steps.png)<!-- -->
