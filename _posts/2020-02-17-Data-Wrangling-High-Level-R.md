---
layout: post
title: "High-Level Data Wrangling in R: Imports, Pivots, and Joins"
description: "In which we go over importing data into R, working with 'tidy data,' manipulating single tables, and joining related tables."
thumb_image: 
tags: [academic, r]
---

**The scenario:** you have a dataset that you'd like to analyze, but before you can do so, you have to import it into R and get it into a workable format. This post dives into the necessary precursors to any kind of data analysis: importing and tidying data.[^1] For reference, here is the [RStudio Data Import Cheat Sheet]({{ site.baseurl }}/pdf/r-cheat-sheet-data-import.pdf){:target="blank"}. 

First, let's explore the concept of tidy data.

In a tidy dataset, each variable must have its own column, each observation must have its own row, and each value must have its own cell -- in practice, you can make a dataset tidy by formatting it as a **tibble** and putting each variable into a column. This allows you to make use of R's vectorized nature and store the data in a consistent way.

##### A quick detour to introduce **tibbles**, an improved type of data frame:
* Tibbles only print the first 10 rows of data by default, and has stricter subsetting rules than data frames
* Create a new tibble from individual vectors: `tibble()`
* Create a new tibble by doing data entry in code: `tribble()`
* Turn a data frame into a tibble: `as_tibble()`
* Turn a tibble into a data frame: `as.data.frame()`
* Use backticks to refer to non-syntactic column names for tibbles
* For more details, see `vignette("tibble")`


[^1]: The content in this post is based on chapters nine through thirteen of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund, which I would recommend reading for a more thorough explanation.

## Importing Data
---
There are a number of different functions to import different types of flat files into data frames, and they all work similarly -- we'll be using `read_csv()` from the *readr* package, which is part of the tidyverse.

`read_csv()` scans the first 1000 rows in the file and guesses what type of data each column is (logical, integer, double, number, time, date, and date-time). Most csv files can be read with variations of the following code:
{% highlight R %}
# skip the first two lines of the file, first processed row is not headings
read_csv("filepath", skip = 2, col_names = FALSE)

# drop all lines that start with "#", and name the three columns "x", "y", and "z"
read_csv("filepath", comment = "#", col_names = c("x", "y", "z"), na = ".")
{% endhighlight %}

However, if the first 1000 rows of a column is not representative of the whole dataset or if there are other parsing problems, identify the problem rows with `problem()` and try the following:

1. Specify the column types manually with the parameter `col_types`
2. Increase the number of lines to parse from 1000 with parameter `guess_max`
3. Import all columns as characters using `col_types = cols(.default = col_character())`, then resolve the errors, possibly with `type_convert()`
4. Read the file into a character vector of lines, `read_lines()` or a character vector of length one, `read_file()`, and then use more advanced string parsing skills.

{% highlight R %}
# Solution 1: Specify the column types manually with col_types
challenge <- read_csv(readr_example("challenge.csv"))
challenge <- read_csv(
  readr_example("challenge.csv"), 
  col_types = cols(
    x = col_double(),
    y = col_date()))
{% endhighlight %}

##### But wait -- how does R parse data, and what can go wrong?
Parsing functions take a character vector and return a more specialized vector like a logical, integer, or date. They make up the foundation of importing data, and are useful to understand for imports.

* `parse_factor()`
* `parse_logical(), parse_integer()`
* `parse_double(), parse_number()` -- potential problems with different grouping marks
* `parse_character()` -- potential problems with non UTF-8 encoding; try `guess_encoding()`
* `parse_date(), parse_time(), parse_datetime()` -- potential problems with formatting; [reference](https://r4ds.had.co.nz/data-import.html#readr-datetimes).

## Managing Data in a Single Table
---
When some of the column names are *values* and not variable names, we can use `pivot_longer()` to pivot the columns into a new pair of variables, for example:
{% highlight R %}
preg <- tribble(
  ~pregnant, ~male, ~female,
  "yes",     NA,    10,
  "no",      20,    12)
preg %>%
  pivot_longer(
  	c("male", "female"), # these column names should be values
  	names_to = "sex",    # new column with names of the old columns: 'sex'
  	values_to = "count") # new column with values of the old columns: 'count'
#> # A tibble: 4 x 3
#>   pregnant sex    count
#>   <chr>    <chr>  <dbl>
#> 1 yes      male      NA
#> 2 yes      female    10
#> 3 no       male      20
#> 4 no       female    12
{% endhighlight %}

When some of the observations are scattered across mutiple rows, we can use `pivot_wider()` to move values into column names, for example:

{% highlight R %}
people <- tribble(
  ~name,              ~names,  ~values,
  #------------------|--------|------
  "Chancelor Bennett", "age",    58,
  "Chancelor Bennett", "height", 26,
  "Saoirse Ronan",     "age",    25,
  "Saoirse Ronan",     "height", 54)
people %>%
  pivot_wider(
  	names_from = names,   # new columns should come from 'age' and 'height'
  	values_from = values) # new values for columns should come from 'values'
#> # A tibble: 2 x 3
#>  name                age height
#>  <chr>             <dbl>  <dbl>
#>1 Chancelor Bennett    58     26
#>2 Saoirse Ronan        25     54
{% endhighlight %}

If needed, we can use `separate(), unite()` to split columns and combine columns:

{% highlight R %}
people %>% 
  separate("name", into = c("first_name", "last_name"), # split name column
   sep = " ", 
   convert = TRUE) %>% 
  unite("name", "first_name", "last_name")	# combine name column
{% endhighlight %}

Missing values can be **missing explicitly** with an NA flag, or **missing implicitly** if they're just not listed. In the following example, the 4th quarter of 2015 is explicity missing and the 1st quarter of 2016 is implicitly missing. 

We can make both missing values explicit with `pivot_wider` or `complete()`, or make both missing values implicit with `pivot_longer(..., values_drop_na = TRUE`

Note: `fill()` can be used to fill in missing values with the most recent non-missing value.

{% highlight R %}
stocks <- tibble(
  year   = c(2015, 2015, 2015, 2015, 2016, 2016, 2016),
  qtr    = c(   1,    2,    3,    4,    2,    3,    4),
  return = c(1.88, 0.59, 0.35,   NA, 0.92, 0.17, 2.66))

stocks %>% # view both missing values explicitly using pivot_wider
  pivot_wider(names_from = year, values_from = return)
#> # A tibble: 4 x 3
#>    qtr `2015` `2016`
#>  <dbl>  <dbl>  <dbl>
#>1     1   1.88  NA   
#>2     2   0.59   0.92
#>3     3   0.35   0.17
#>4     4  NA      2.66

stocks2 %>% # hide both missing values using pivot_longer(..., values_drop_na = TRUE)
  pivot_longer(
  	c(`2015`,`2016`), 
  	names_to = "year", 
  	values_to = "return", 
  	values_drop_na = TRUE)
#> # A tibble: 6 x 3
#>    qtr year  return
#>  <dbl> <chr>  <dbl>
#>1     1 2015    1.88
#>2     2 2015    0.59
#>3     2 2016    0.92
#>4     3 2015    0.35
#>5     3 2016    0.17
#>6     4 2016    2.66

stocks %>% # make missing values explicit using complete() to check combos of year/qtr
  complete(year, qtr)
#># A tibble: 8 x 3
#>   year   qtr return
#>  <dbl> <dbl>  <dbl>
#>1  2015     1   1.88
#>2  2015     2   0.59
#>3  2015     3   0.35
#>4  2015     4  NA   
#>5  2016     1  NA   
#>6  2016     2   0.92
#>7  2016     3   0.17
#>8  2016     4   2.66
{% endhighlight %}

## Managing Relational Data in Multiple Tables
---
Combine data between related data by matching the unique identifier in your main table (primary key) with the unique identifier in another table (foreign key). To ensure that your joins work smoothly:

1. Identify the variables that are the primary key in each table
2. Check that none of the variables in the primary key are missing
3. Check that the primary keys match foreign keys in another table with `anti_join()`

#### **Mutating Joins:** Combine variables from two tables
* `inner join(x, y)` - only matching values are kept
* `left_join(x, y)`  - all x values are kept - MOST COMMON
* `right_join(x, y)` - all y values are kept
* `full_join(x, y)`  - all x and y values are kept

#### **Filtering Joins:** Affects observations, not variables
* `semi_join(x, y)` - keeps all observations in x that have a match in y
* `anti_join(x, y)` - drops all observations in x that have a match in y

#### **Set Operations:** Assumes both tables have the same variables
* `intersect(x, y)` - return only observations in both x and y 
* `union(x, y)`    - return unique observations in x and y
* `setdiff(x, y)`   - return observations in x, but not in y

