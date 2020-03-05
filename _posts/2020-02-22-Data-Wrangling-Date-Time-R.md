---
layout: post
title: "Data Wrangling in R: Dates and Times"
description: "In which we review basic information about dates and times, including how to create them, how they are represented, and what you can do with them."
thumb_image: 
tags: [academic, r]
---

Here we'll review the creation of dates and times, the components that make up dates and times, time spans, and time zones in R.[^1] See the [RStudio Dates and Times Cheat Sheet]({{ site.baseurl }}/pdf/r-cheat-sheet-datetime.pdf){:target="blank"} for a condensed reference. 

There are three types of data that refer to an instant in time: **dates**, **times**, and **date-times**. You can retrieve the current date with `date()` and the current date-time with `now()`. 

We will use the following three packages in this post: 

{% highlight R %}
library("tidyverse")
library("lubridate") # use this when working with date/times
library("nycflights13") # example data set of flights leaving from NYC in 2013
{% endhighlight %}

[^1]: This post is meant for a person who is looking for a refresher on dates and times in R, and the content in this post is based on chapter sixteen of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund.

## Creating Dates and Times
---

##### 1. Create dates and times from strings

Use a function containing `y, m, d` and `h, m, s` to turn a string or series of numbers into a date or date-time. 

{% highlight R %}
ymd("2020-02-22") # numbers in quotes. any combo of ymd will work
mdy(02222020) # unquoted, non-delimited numbers will also work
ymd_hms("2020-02-22 13:05:59") # add combos of _hms to include time
ymd_h(2020022213, tz = "EST") # supply timezone with tz
{% endhighlight %}

##### 2. Create dates and times from other types of data

Use `as_date()` and `as_time()` to convert an object to a date or a date-time.

{% highlight R %}
as_date(now()) # change a date-time to a date
as_datetime(today()) # change a date to a date-time
{% endhighlight %}

##### 3. Create dates and times from components spread across multiple columns in a table

Use `make_datetime()` to create a date-time if your data is spread out across multiple columns in a table. For example, in the nycflights13 dataset, there are separate columns for year, month, day, departure hour, and departure minute. We can create a new date-time column for departure time, and use this new column to visualize the distribution of flights on any day.

{% highlight R %}
flights_dt <- flights %>%
  select(year, month, day, hour, minute) %>%
  mutate(departure = make_datetime(year, month, day, hour, minute)) # create column

flights_dt %>% 
  filter(departure < ymd(20130102)) %>% # filter on January 1st
  ggplot() + # graph departure date-time 
    geom_freqpoly(aes(departure), binwidth = 600) # 600 seconds = 10 minutes
{% endhighlight %}

{% include image.html path="documentation/02-22-date-time-make-datetime.png" 
                      path-detail="documentation/02-22-date-time-make-datetime.png"
                      alt="Make Date-Time" %}

## Retrieve and Edit Date and Time Components
---
Retrieve date and time components from a date or date-time with the following functions:

* `year()` returns the year
* `month(..., label = TRUE)` returns the month; set label = TRUE to return abbreviated month name
* `wday(..., label = TRUE, abbr = FALSE)` returns the day of the week; set abbr = FALSE to return full name
* `mday()` returns the day of the month
* `yday()` returns the day of the year
* `hour()` returns the hour
* `minute()` returns the minute
* `second` returns the second

Edit date and time components with the following functions:

* `update(<DATE>, year = ..., month = ..., mday = ..., hour = ...)` updates dates manually
* `round_date()` rounds to the nearest specified unit (minute, hour, day, week, month, year, etc.)
* `floor_date()` rounds down to the nearest specified unit
* `ceiling_date()` rounds up to the nearest specified unit

**Tip:** Set larger components of a date to a constant to explore patterns in the smaller components. For example, in nycflights13, we can set the flight day to a constant to view the distribution of flights across the course of the day for every day of the year:

{% highlight R %}
# graph the distribution of flights throughout the day for every day of the year
flights_dt %>% 
  mutate(dep_hour = update(dep_time, yday = 1)) %>% 
  ggplot(aes(dep_hour)) +
    geom_freqpoly(binwidth = 300) # 300 seconds = 5 minutes
{% endhighlight %}

{% include image.html path="documentation/02-22-date-time-set.png"
                      path-detail="documentation/02-22-date-time-set.png"
                      alt="Set Components of Date Time" %}

## Time Spans: Durations, Periods and Intervals
---
If you subtract two dates in R regularly, you get a "*difftime*" object that records a time span in seconds, minutes, hours, days, or weeks, which isn't great because the units aren't consistent. Time spans are better measured in these three ways:

1. Durations, which represent an exact number of seconds
2. Periods, which represent human units of time, i.e. weeks, months, and years
3. Intervals, which are durations with a starting and ending point

##### 1. Durations

Durations measure time in seconds. They can be added to each other, multiplied by a constant, and added or subtracted from dates and times.

{% highlight R %}
# convert age to duration
my_age <- today() - ymd(19940101)
my_age <- as.duration(my_age) 

# use d* functions to calculate duration in seconds, minutes, hours, days, weeks, years
dseconds(15)
dminutes(10) * 2 # multiplied by a constant
dhours(c(12, 24))
ddays(0:5)
dweeks(3) + dyears(1) # can be added and multiplied
tomorrow <- today() + ddays(1) # can be added and subtracted from days
{% endhighlight %}

Note: When adding or subtracting durations to dates, be aware of time discrepancies such as daylight savings time.

##### 2. Periods

Periods measure time in human units, such as weeks and months. This takes into account time discrepancies such as daylight savings time with adding or subtracting from dates or times.

{% highlight R %}
# calculate periods in seconds, minutes, hours, days, months, weeks, years
seconds(15)
minutes(10)
hours(c(12, 24))
months(1:6)
days(7) + weeks(3) # can be added and multiplied
ymd("2020-02-22") + years(1) # can be added and subtracted from dates
{% endhighlight %}

##### 3. Intervals

Intervals are durations with starting an ending points, and are represented by `%--%`. They are useful when trying to determine how long a specific period is, sice you want to include all time discrepancies for that period.

For example, you may want to know the number of days between today and the same day next year -- this calculation depends on whether it's a leap year or not, so we should use intervals.

{% highlight R %}
# how many days in the interval between today and next year?
next_year <- today() + years(1)
(today() %--% next_year) / ddays(1) # use regular division for duration in an interval 
(today() %--% next_year) %/% days(1) # use integer division for periods in an interval 
{% endhighlight %}

## Time Zones
---
Time zones are complicated, but here are a couple of basic functions that are useful to know:

* `Sys.timezone()` returns the time zone of your workstation, according to R
* `with_tz(<DATETIME>, tzone = "America/Detroit")` changes time zone while keeping the instant in time
* `force_tz(<DATETIME>, tzone = "America/Detroit")` changes time zone and changes the instant in time





