---
layout: post
title: "Programming in R: Iteration"
description: "In which we explore the basics of iteration through the lenses of functional and imperative programming by examining for loops, map functions, and more."
thumb_image: 
tags: [academic, r]
---

Iteration, like functions, serve to reduce the amount of duplication in code. In R, this comes in two flavors: imperative programming, such as for and while loops, and functional programming, which avoids changing-state data.[^1]

[^1]: This post is meant for a person who is looking for a refresher on basic programming in R, and the content in this post is based on chapter twenty-one of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund.

## Imperative Programming: For Loops
When creating for loops in R, it's important to allocate sufficient space for new objects using the `vector()` function. For example, we create a new object that computes the median of each column of a data frame:

{% highlight R %}
output <- vector("double", ncol(df)) # 1. output: make sure to allocate sufficient space
for (i in seq_along(df)) {          # 2. sequence: how should the function iterate?
  output[[i]] <- median(df[[i]])   # 3. body: what should the code do for each case?
}
{% endhighlight %}

There are a couple of looping patterns worth noting:
1. `for (i in seq_along(xs))` loops over indices; this is the most common way to construct for loops.
2. `for (x in xs)` loops over elements; this is useful if you only care about side effects like plots.
3. `for (nm in names(xs))` loops over names; this is useful if you only need names 

{% highlight R %}
# using seq_along(), we can extract names and values of all indices
for (i in seq_along(x)) {
  name <- names(x)[[i]] # extract names
  value <- x[[i]] # extract values
}
{% endhighlight %}

Most of the time, you will know what form the function output will take. However, if the length of the output is unknown, combine the data into a single object after the loop is complete:
* When generating an unknown number of results, save vectors to a list and combine them into a single vector with `unlist()`
* When generating a long string, save outputs in a character vector and combine them into a single string with `paste(output, collapse = "")`
* When generating a big data frame, save output to a list and combine them into a single data frame with `bind_rows(output)`

{% highlight R %}
# sample a random number (1-100) of observations from a normal dist with different means
means <- c(0, 1, 2)
out <- vector("list", length(means)) # create a list of length 3
for (i in seq_along(means)) {
  n <- sample(100, 1)
  out[[i]] <- rnorm(n, means[[i]]) # store data in list values
}
str(unlist(out)) # flatten list of vectors into a single vector
{% endhighlight %}

As an example exercise to demonstrate the syntax of for loops, let's write a function that takes a data frame, and prints the mean of each numeric column next to the name of that column.

{% highlight R %}
show_means <- function(x) {
  numeric_cols <- select_if(x, is.numeric) # filter on numeric columns
  numeric_means <- vector("list", 2)
  for (i in seq_along(numeric_cols)) { # populate the list with names and means
    numeric_means[[1]][[i]] <- names(numeric_cols[i])
    numeric_means[[2]][[i]] <- round(mean(numeric_cols[[i]]), 2)
  }

  a <- str_count(numeric_means[[1]])
  b <- str_count(round(numeric_means[[2]], 0))
  padding <- max(a + b) - a - b
  for (i in seq_along(numeric_means[[1]])) # print names and means with padding
    cat(numeric_means[[1]][[i]], ": ", str_dup(" ", padding[i]), 
      format(numeric_means[[2]][[i]], nsmall = 2), fill = TRUE, sep = "")
  }
}

show_means(iris)
#> Sepal.Length: 5.84
#> Sepal.Width:  3.06
#> Petal.Length: 3.76
#> Petal.Width:  1.20
{% endhighlight %}

## Functional Programming: Map Functions
Map functions automate the pattern of looping over a vector, applying a function to each piece, and returning a new vector that's the same length as the input. These functions are a part of the *purrr* package, which are implemented in C for speed. There is one map function for each type of output:

* `map()` makes a list
* `map_lgl()` makes a logical vector
* `map_int()` makes an integer vector
* `map_dbl()` makes a double vector
* `map_chr()` makes a character vector

**Example 1:** The most basic case -- calculate the means of all columns of the *mtcars* dataset.

{% highlight R %}
library("purrr")
map_dbl(mtcars, mean)
{% endhighlight %}

**Example 2:** Pass in a one-sided formula for anonymous functions. For each value of the number of cylinders on a car, what is the the linear relationship between miles per gallon and car weight?

{% highlight R %}
models <- mtcars %>% 
  split(.$cyl) %>% 
  map(~lm(mpg ~ wt, data = .))
{% endhighlight %}

**Example 3:** Extract named components from a function using a string. For each linear relationship between miles per gallon and car weight by the number of cylinders, what is the correlation?

{% highlight R %}
models %>% 
  map(summary) %>% 
  map_dbl("r.squared")
{% endhighlight %}

**Example 4:** Select elements by position by passing in an integer. Select the elements in the second position of a list.

{% highlight R %}
x <- list(list(1, 2, 3), list(4, 5, 6), list(7, 8, 9))
x %>% map_dbl(2)
{% endhighlight %}

#### Dealing with mapping failures: **safely()**, **possibly()**, and **quietly()**
If any one of the mapping operations fails, then an error message will be returned with no output. There are three ways to deal with failures:

1. `safely()` returns a modified version of a function that returns a list of two elements -- result and error.

{% highlight R %}
x <- list(1, 10, "a")
x %>% map(safely(log))
  %>% str()
#> List of 3
#>  $ :List of 2
#>   ..$ result: num 0
#>   ..$ error : NULL
#>  $ :List of 2
#>   ..$ result: num 2.3
#>   ..$ error : NULL
#>  $ :List of 2
#>   ..$ result: NULL
#>   ..$ error :List of 2
#>   .. ..$ message: chr "non-numeric argument to mathematical function"
#>   .. ..$ call   : language .Primitive("log")(x, base)
#>   .. ..- attr(*, "class")= chr [1:3] "simpleError" "error" "condition"
{% endhighlight %}

{:start="2"}
2. `possibly()` always always succeeds, but gives a default value when there is an error.

{% highlight R %}
x <- list(1, 10, "a")
x %>% map_dbl(possibly(log, NA_real_))
  %>% str()
#> num [1:3] 0 2.3 NA
{% endhighlight %}

{:start="3"}
3. `quietly()` captures printed output, messages, and warnings instead of errors.

{% highlight R %}
x <- list(1, -1)
x %>% map(quietly(log)) 
  %>% str() 
#> List of 2
#>  $ :List of 4
#>   ..$ result  : num 0
#>   ..$ output  : chr ""
#>   ..$ warnings: chr(0) 
#>   ..$ messages: chr(0) 
#>  $ :List of 4
#>   ..$ result  : num NaN
#>   ..$ output  : chr ""
#>   ..$ warnings: chr "NaNs produced"
#>   ..$ messages: chr(0) 
{% endhighlight %}

#### Map a function over multiple arguments: **map2()** and **pmap()**
1. `map2()` allows you to iterate along two related inputs in parallel. As an example, we can draw a random sample from the normal distribution with three different mean and standard deviation pairs:

{% highlight R %}
mu <- list(1, 10, 100)
sigma <- list(1, 5, 10)
map2(mu, sigma, rnorm, n = 5) %>% str()
{% endhighlight %}

{:start="2"}
2. `pmap()`, on a similar note, iterates along a list of related inputs in parallel. If your arguments are the same length, it is best practice to store them in a data frame. For example, we can vary the mean, standard deviation, and the number of samples from a normal distribution:

{% highlight R %}
params <- tribble(
  ~mean, ~sd, ~n,
    5,     1,  1,
   10,     5,  3,
   -3,    10,  5
)
params %>% 
  pmap(rnorm)
{% endhighlight %}


#### Map multiple functions with multiple sets of arguments: **invoke_map()**

`invoke_map()` can invoke different functions with multiple parameters. Use a data frame to make matching pairs easier. For example, we can generate samples from three different types of distributions with their own inputs:

{% highlight R %}
sim <- tribble( # use a data frame to make matching pairs easier
  ~f,      ~params,
  "runif", list(min = -1, max = 1),
  "rnorm", list(sd = 5),
  "rpois", list(lambda = 10)
)
sim %>% 
  mutate(sim = invoke_map(f, params, n = 10))
{% endhighlight %}

#### Walk: an alternative to map

When you want to call a function for its side effects, such as printing output to the screen or saving files to disk, you can use the walk functions: `walk(), walk2(), pwalk()`.

For example, if you had a list of plots and a vector of file names, you can use the `pwalk()` function to save each file to teh corresponding location on disk.

{% highlight R %}
library(ggplot2)
plots <- mtcars %>% 
  split(.$cyl) %>% 
  map(~ggplot(., aes(mpg, wt)) + geom_point())
paths <- stringr::str_c(names(plots), ".pdf")

pwalk(list(paths, plots), ggsave, path = tempdir())
{% endhighlight %}

Note: these functions all invisibly return the first argument, which makes them suitable for use in the middle of pipelines.

## Functional Programming: Other Patterns

There are a number of other functions in the *purrr* package that abstract over other types of for loops, including predicate functions, reduce, and accumulate.

#### Predicate functions check a condition, returning either TRUE or FALSE

* `keep()` keeps element where the predicate is true
* `discard()` discards elements where the predicate is false
* `some()` and `every()` determines if the predicate is true for any or all of the elements
* `detect()` finds the first element where true, detect_index returns its position
* `head_while()` and `tail_while()` take elements from the start r end of a vector while a predicate is true

{% highlight R %}
# keep columns that are factors
iris %>% 
  keep(is.factor) %>% 
  str()

# discard columns that are factors
iris %>% 
  discard(is.factor) %>% 
  str()

# check if some or all elements are characters
x <- list(1:5, letters, list(10))
x %>% 
  some(is_character) # are any elements characters?
x %>% 
  every(is_vector) # are all elements vectors?

# detect the first element, and the index of the first element, that is > 5
x <- sample(10)
x %>% 
  detect(~ . > 5)
x %>% 
  detect_index(~ . > 5)

# show elements from the head or tail of the list until a value of > 5
x %>% 
  head_while(~ . > 5)
x %>% 
  tail_while(~ . > 5)
{% endhighlight %}

#### Reduce and Accumulate use a binary function to simplify complex lists

`reduce()` takes a binary function -- a function with two primary inputs -- and applies it to a list until only one element is left.

{% highlight R %}
# find the intersection of all vectors in a list
vs <- list(
  c(1, 3, 5, 6, 10),
  c(1, 2, 3, 7, 8, 10),
  c(1, 2, 3, 4, 8, 9, 10)
)
vs %>% reduce(intersect)
#> [1]  1  3 10
{% endhighlight %}

`accumulate()` is the same as reduce except it keeps all of the interim results.

{% highlight R %}
# accumulate the values of a random sampling from 1-10
x <- sample(10)
x %>% accumulate(`+`)
#> [1]  8 18 27 34 36 37 43 46 50 55
{% endhighlight %}
