---
layout: post
title: "Basic Data Transformation in R"
description: "In which we review the fundamentals of transforming data in R, using six key functions in the dplyr package: filter, arrange, select, mutate, summarize, and group by."
thumb_image: 
tags: [academic, r]
---

In this post we'll be taking a look at basic data transformation in R -- namely, six important R functions included in the [dplyr](http://dplyr.tidyverse.org){:target="blank"} package:[^1]

[^1]: This post is meant for a person who is looking for a refresher on the very basics of the dplyr package (i.e., me when I inevitably forget how to summarize data). The content in this post is based on chapter five of [R for Data Science](https://r4ds.had.co.nz/index.html){:target="blank"} by Hadley Wickham & Garrett Grolemund, which I would recommend reading for a more thorough explanation.

* `filter()` - filter rows
* `arrange()` - order rows
* `select()` - select columns
* `mutate()` - transform and create variables
* `group_by()` - group data
* `summarize()` - summarize data

These functions will help you scratch the surface of manipulating datasets.

**First things first:** Here is the [RStudio Data Transformation Cheat Sheet]({{ site.baseurl }}/pdf/r-cheat-sheet-data-transformation.pdf){target="blank"}, if that's what you're looking for.

### Prologue: The Pipe
Before we get into the functions themselves, let's review an important part of R syntax: the pipe, `%>%`. The pipe symbol can be read as "then."

{% highlight R %}
x %>% f(y) == f(x, y)
x %>% f(y) %>% g(z) == g(f(x, y), z)
...
{% endhighlight %}

See the R code at the bottom for examples of the pipe in action, and my [post on programming in R](programming-r) for a deeper dive into pipes.

## Filter
`filter()` allows for subsetting observations based on their values. In other words, you can filter the rows in your dataset.
* Filter with comparison operators: `== , != , > , >= , < , <=`
* Filter with logical operators -- and, or not, and exclusive or: `&, |, !, xor()`
* Filter on rows where x is one of the values in vector y: `x %in y`
* Determine if a value is missing: `is.na()`
* Determine if numbers fall within an inclusive range: `between()`

Note: Don't forget De Morgan's law, `!(x & y) == !x | !y , !(x | y) = !x & !y`

## Arrange
`arrange()` allows you to re-order the rows of your dataset.
* By default, arrange sorts your data in ascending order
* Sort data in descending order with `desc()`
* Missing values are placed at the end unless explicitly told otherwise

## Select
`select()` lets you zoom in on a subset of the data based on the names of columns. This is useful for narrowing data with many variables down to only the ones you care about. 
* Select based on column name: `starts_with(), ends_with(), contains(), matches()`
* Select all columns between two columns, inclusive: `(x:y)`
* Exclude columns from being selected: `-`
* Rename a variable without dropping the others: `rename()` 
* Reorder ceertain columns to the very beginning: `select()` with `everything()`

## Mutate
`mutate()` lets you add new columns that are functions of existing columns to the end of the table. 
* Create columns with arithmatic operators, `+, -, *, /, ^`, with aggregate functions, `sum, mean`
* Create columns with integer division, `%/%`, and remainder ,`%%`
* Create columns with logs: `log(x), log2(x), log10(x)`
* Create columns with offsets, such as leading and lagging values, i.e. `x - lead(x), x != lag(x)`
* Create columns with cumulative and rolling aggregates: `cumsum, cumprod, cummin, cummax, cummean`
* Create columns with logical comparisons: `<, >, ==, !=`
* Create columns with ranking functions: `minrank, row_number, percent_rank, ntile`
* Select the top or bottom entries in a group `top_n()`
* Only keep the new variables that you define with `transmute()`

## Group By
`group_by()` changes the scope of analysis from the complete dataset to individually defined groups.

It is most useful when used with `summarize()`, but it can also be used with `mutate()` and `filter()` -- for example, if you wanted to fine the worst members of each group, find all groups bigger than a threshold, or create group summary metrics.

Use `ungroup()` to remove groupings.

## Summarize
`summarize()` collapses a data frame into a single row. If the data is grouped, then it summarizes by group.
* Summarize by location: `mean, median`
* Summarize by spread: `sd, IQR, mad`, (std. deviation, interquartile range, median absolute deviation)
* Summarize by rank: `min, max, quantile`
* Summarize by position: `first, nth, last`
* Summarize by count: `n(), sum(is.na(x)), n_distinct(x)`
* Use the `na.rm` parameter for functions like mean to specify whether to remove missing values before computation. 
* Use a count, `n()`, when doing aggregation to prevent drawing conclusions from small amounts of data
* Use the `count()` function (with an optional weight variable) if you simply want a count
* Use the `count()` function to also find counts and proportions of logical values

#### ...The End!

Keep reading for examples of these functions in action.

## Examples and R Code
---
We'll be using two libraries: [tidyverse](https://www.tidyverse.org){:target="blank"}, which contains the dplyr package, and nycflights13, a data frame that contains all flights that departed from New York City in 2013.

{% highlight R %}
### load packages
library(tidyverse)
library(nycflights13)

# tip: use the pipe %>% to write more efficient code
# investigate the relationship between the distance and average delay for each location
delays <- flights %>%
  group_by(dest) %>% # group by flight destination
  summarise(avg_distance = mean(distance), # and then summarize by the average distance,
            avg_delay = mean(arr_delay, na.rm = TRUE), # the average delay,
            count = n()) %>% # and the number of times it happened 
  filter(count > 20) # filter the summary only if the destination was flown to >20 times
# then we plot the average flight delay by average distance flown
ggplot(data = delays, mapping = aes(x = avg_distance, y = avg_delay)) +
  geom_point(mapping = aes(size = count), alpha = 1/3) + 
  geom_smooth(se = FALSE)

# tip: include a count, n(), when doing aggregation 
# this way you don't draw conclusions from small amounts of data
# example: create a data set of not cancelled flights
not_cancelled <- flights %>% 
  filter(!is.na(dep_delay), !is.na(arr_delay))
# create a summarized data set of average delays
delays <- not_cancelled %>% 
  group_by(tailnum) %>% 
  summarise(
    delay = mean(arr_delay, na.rm = TRUE),
    n = n()
  )
# graph only average delays where the total delays happened over 25 times
# we see that the delays have less variance
delays %>% 
  filter(n > 20) %>% 
  ggplot(mapping = aes(x = n, y = delay)) + 
  geom_point(alpha = 1/10)

# tip: use the count() function with an optional weight variable
# count the total # miles a plane flew
not_cancelled %>%
  count(tailnum, wt = distance)

# tip: count() can be used to find counts and proportions of logical values
# how many flighs left before 5am?
not_cancelled %>%
  group_by(year, month, day) %>%
  count(early_flights = sum(dep_time < 500))
# what proportion of flights are delayed more than an hour?
not_cancelled %>%
  group_by(year, month, day) %>%
  count(big_delays = mean(arr_delay > 60))

# tip: groups of multiple variables can be summarized individually
# can be used for sums, but be careful of weighting means and variances
daily <- group_by(flights, year, month, day)
(per_day <- summarize(daily, flights = n()))
(per_month <- summarize(per_day, flights = sum(flights)))
(per_year <- summarize(per_month, flights = sum(flights)))

# tip: use ungroup() to remove groupings
# ungroup the grouped flights and display the total number of flights
daily %>% 
  ungroup() %>%
  summarize(flights = n())

# exercise: is the prop of cancelled flights per day related to the average delay?
flights %>%
  group_by(day) %>%
  summarise(cancelled = mean(is.na(dep_delay)),
            mean_dep = mean(dep_delay, na.rm = T),
            mean_arr = mean(arr_delay, na.rm = T)) %>%
  ggplot(aes(y = cancelled)) +
  geom_point(aes(x = mean_dep), colour = "red") +
  geom_point(aes(x = mean_arr), colour = "blue") +
  labs(x = "Avg delay per day", y = "Cancelled flights p day")

# tip: grouping can also be used with filter() and mutate()
# find the worst members of each group, 
flights %>%
  group_by(year, month, day) %>%
  filter(rank(desc(arr_delay)) < 10)
# or find all groups bigger than a threshold
popular_dests <- flights %>% 
  group_by(dest) %>% 
  filter(n() > 365)
# and then standardize to compute per group metrics (usually you will avoid)
# fun fact: functions that work in grouped mutates and filters are window functions
# vignette("window-functions") for more info
popular_dests %>% 
  filter(arr_delay > 0) %>% 
  mutate(prop_delay = arr_delay / sum(arr_delay)) %>% 
  select(year:day, dest, arr_delay, prop_delay)

# which plane (tailnum) has the worst on-time record?
not_cancelled %>%
  group_by(tailnum) %>%
  summarize(count = n(),
            on_time_prop = sum(arr_delay > 0, na.rm = T)/count) %>%
  arrange(on_time_prop) %>%
  filter(count > 25)

# what time of the day should you fly to avoid delays?
flights %>%
  filter(is.na(dep_time) != TRUE) %>%
  group_by(hour) %>%
  summarize(avg_dep_delay = mean(dep_delay, na.rm = T),
            avg_arr_delay = mean(arr_delay, na.rm = T),
            count = n()) %>%
  ggplot() +
    geom_point(mapping = aes(x = hour, y = avg_dep_delay, size = count), color = "red") +
    geom_point(mapping = aes(x = hour, y = avg_arr_delay, size = count), color = "blue")
  
# compute total minutes of delay for each destination
# compute proportion of total delay for the destination from each flight
not_cancelled %>%
  group_by(dest) %>%
  summarize(total_delay = sum(arr_delay[arr_delay > 0]))%>%
  arrange(desc(total_delay))

not_cancelled %>%
  filter(arr_delay > 0) %>%
  group_by(tailnum) %>%
  mutate(total_delay_dest = sum(arr_delay)) %>%
  group_by(tailnum, dest) %>%
  summarize(flight_delay = sum(arr_delay),
            prop_of_delay_by_dest = flight_delay / mean(total_delay_dest))

# explore how flight delay is related to the delay of the immediately preceding flight
not_cancelled %>%
  filter(month == 1 & day == 1 & origin == "LGA") %>% # only looking at 1/1 in LGA
  arrange(dep_time) %>%
  mutate(prev_flight_delay = lag(dep_delay)) %>%
  ggplot(data = delay_lag, mapping = aes(x = dep_delay, y = prev_flight_delay)) + 
  geom_point() +
  geom_smooth(se = FALSE)

# find flights that are suspiciously fast
# compute air time of a flight relative to the shortest flight to that dest
# which flights were most delayed in the air?
not_cancelled %>%
  select(year:day, air_time, distance, origin, dest) %>%
  ggplot(aes(x = distance, y = air_time)) +
  geom_point() +
  geom_smooth()

f <- not_cancelled %>% 
  mutate(speed = air_time / distance) %>% # computed air time relative to distance
  arrange(desc(speed)) %>%
  filter(min_rank(desc(speed)) < 10)  # top 10 most in air delay

# find all destinations flown by at least two carriers, then rank the carriers 
not_cancelled %>%
  group_by(dest) %>%
  summarize(num_carriers = n_distinct(carrier)) %>%
  filter(num_carriers > 2) %>%
  arrange(desc(num_carriers))

flights %>%
  group_by(dest) %>%
  filter(n_distinct(carrier) > 2) %>%
  group_by(carrier) %>%
  summarise(n = n_distinct(dest)) %>%
  arrange(-n)

# for each plane, count the number of flights before the first delay of >1 hour
not_cancelled %>%
  group_by(origin, tailnum) %>%
  arrange(tailnum, year, month, day) %>%
  summarize(count = n(),
            long_delay_happened = sum(cumsum(dep_delay > 60) < 1))

{% endhighlight %}
