---
layout: post
title: "Programming in R: Pipes, Functions, and Vectors"
description: "In which we explore the basics of programming in R by examining common programming tools, data structures, and strategies to help effectively analyze data."
thumb_image: 
tags: [academic, r]
---

I'll start with some general tips & tricks that I find useful in RStudio before jumping into the main topics:[^1] 
* **cmd + shift + r** creates a commented header in the code for readability
* **cmd + shift + p** resends a code chunk from the editor into console
* **cmd + shift + f10** restarts RStudio
* **cmd + shift + s** reruns the current script

[^1]: This post is meant for a person who is looking for a refresher on basic programming in R, and the content in this post is based on chapters seventeen through twenty of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund.


## Pipes
---
Pipes, which can be denoted by {% ihighlight R %}%>%{% endihighlight %} and pronounced while reading code as "then," help clearly express a sequence of multiple operations by eliminating the need to create intermediate objects and bringing focus to the primary object and its transformations.  

{% highlight R %}
x %>% f(y) == f(x, y)
x %>% f(y) %>% g(z) == g(f(x, y), z)
...
{% endhighlight %}

However, because pipes reassemble code in a form that overwrites an intermediate object, they **will not work** for:

1. Functions that use the current environment must explicity define the environment if being used in a pipeline; for example, {% ihighlight R %}assign(), get(), load(){% endihighlight %}
2. Functions that use lazy evaluation cannot be used with pipes; for example, {% ihighlight R %} try(), tryCatch(), suppresMessages(), suppressWarnings(){% endihighlight %}

Furthermore, **do not** use pipes in these scenarios:

* If there are multiple inputs or outputs (pipes should only transform one primary object)
* If there is a directed graph with a complex dependency structure (pipes are linear)
* If there are many steps in the pipeline, consider creating meaningfully named intermediate objects to help debugging

The pipe is loaded automatically in the tidyverse, but originally comes from the *magrittr* package. The *magrittr* package also has some additional piping tools that may be useful:

* {% ihighlight R %}%T>%{% endihighlight %} returns the left-hand side instead of the right-hand side. This is useful for printing, plotting, or saving objects in a way that doesn't terminate the pipeline.
* {% ihighlight R %}%$%{% endihighlight %} explodes out the variables in a data frame so you can refer to them explicitly. This is useful if you want to pass individual vectors, not data frames, into functions.

{% highlight R %}
library("magrittr")
# Use %T>% to plot a distribution and continue to use that object in a pipeline
rnorm(100) %>%
  matrix(ncol = 2) %T>%
  plot() %>%
  str()
# Use %$% to find correlation between two columns by referring to the columns explicitly
mtcars %$%
  cor(disp, mpg)
{% endhighlight %}


## Functions
---
You should consider creating a function when you've copied and pasted a block of code more than twice. The aspects of functions that we'll dive into below are conditional execution, function arguments, and return values.[^2]

[^2]: More complicated functions may warrant unit testing. For information regarding unit testing in R, see [the testthat package](http://r-pkgs.had.co.nz/tests.html).

#### Conditional Execution
{% highlight R %}
if (condition_1) {
  # code executed when condition_1 is TRUE
} else if (condition_2) {
  # code executed when condition_2 is TRUE
} else {
  # code executed when both conditions are FALSE
}
{% endhighlight %}

* conditions can be linked with {% ihighlight R %}||{% endihighlight %} or {% ihighlight R %}&&{% endihighlight %}
* use {% ihighlight R %}near(){% endihighlight %} when comparing floating point numbers 
* use {% ihighlight R %}ifelse(){% endihighlight %} for conditional element selection
* use {% ihighlight html %}switch(){% endihighlight %} to eliminate long chains of if statements
* use {% ihighlight R %}cut(){% endihighlight %} to eliminate long chains of if statements; this is useful because it works with vectors
* be careful when testing for equality because {% ihighlight R %}=={% endihighlight %} is vectorized; either ensure vectors are length 1, collapse vectors with {% ihighlight html %}all(){% endihighlight %} or {% ihighlight html %}any(){% endihighlight %}, or use the non-vectorized function {% ihighlight R %}identical(){% endihighlight %}

Below are two simple examples of {% ihighlight R %}cut(){% endihighlight %} and {% ihighlight R %}switch(){% endihighlight %} in action.

{% highlight R %}
# use cut() to separate temperatures into different descriptive categories
cut(c(-99, -4, 2, 24, 66), breaks = c(-Inf, 0, 10, 20, 30, Inf),
    labels = c("freezing", "cold", "cool", "warm", "hot"))
#> [1] freezing freezing cold     warm     hot     
#> Levels: freezing cold cool warm hot

# use switch() to quickly define alternative arithmatic operations based on an input
arithmatic <- function(x, y, op) {
 switch(op,
   plus = x + y,
   minus = x - y,
   times = x * y,
   divide = x / y,
   stop("Unknown op!")
 )
}
{% endhighlight %}

#### Function Arguments
Generally, the standard naming convention for function arguments are as follows:

* {% ihighlight R %}x, y, z{% endihighlight %}: vectors.
* {% ihighlight R %}i, j{% endihighlight %}: numeric indices (typically rows and columns).
* {% ihighlight R %}df{% endihighlight %}: a data frame.
* {% ihighlight R %}n{% endihighlight %}: length, or number of rows.
* {% ihighlight R %}p{% endihighlight %}: number of columns.
* {% ihighlight R %}w{% endihighlight %}: a vector of weights.

It is common to check for preconditions for your functions, such as making sure the inputs are in the correct format. If these conditions are not met, you may want to exit the function immediately. You can check for preconditions using {% ihighlight R %}stop(){% endihighlight %} to check a condition and throw an error, or {% ihighlight R %}stopifnot(){% endihighlight %} to check if each argument is true, and return a generic error message if any are not true. 

Finally, functions can include a {% ihighlight R %}...{% endihighlight %} argument to send an arbitrary number of inputs on to another function, for example:

{% highlight R %}
# using "...", this function collapses many strings into one comma-delimited string
commas <- function(...) {
  str_c(..., collapse = ", ")
}
{% endhighlight %}

#### Return Values
The return value is usually the last statement that a function evaluates, but you can coose to return early by using {% ihighlight R %}return(){% endihighlight %} -- for example, if the inputs are empty, or if there are simple edge cases.

Functions can either **transform** an object or create **side-effects** like drawing a graph or saving a file. For functions that are side-effects, consider returning the first aragument invisibly so that they can still be used in a pipeline.

{% highlight R %}
# prints the number of missing values in a data frame while preserving pipeline
show_missings <- function(df) {
  n <- sum(is.na(df))
  cat("Missing values: ", n, "\n", sep = "")
  invisible(df)
}
{% endhighlight %}

## Vectors
---
Most functions that you write will work with vectors, a data structure with a sequence of cells that contain data. 
1. **Atomic vectors** are one-dimensional, only contain one type of data, and fall into one of six categories: logical, integer, double, character, complex and raw. **Null** represents the absence of a vector.
2. **Lists**, or recursive vectors, can contain multiple types of data, including other lists. 

Every vector has two required properties -- type, determined with {% ihighlight R %}typeof(){% endihighlight %}, and length, determined with {% ihighlight R %}length(){% endihighlight %}. They can also have aritrary additional attributes, which create augmented vectors such as factors (built on integer vectors), date/times (built on numberic vectors), and data frames/tibbles (built on lists).

### 1. Atomic Vectors
* Logical vectors can take three possible values: TRUE, FALSE, NA
* Integer vectors contain integers; numbers are doubles by default, place an {% ihighlight R %}L{% endihighlight %} after the number to denote an integer. 
* Double vectors contain doubles; null values are represented by {% ihighlight R %}NA, NaN, Inf, -Inf{% endihighlight %}
* Character vectors are made up of string elements
* Raw and complex vectors are rarely used in data analysis

Note: each type of atommic vector has its own missing value: {% ihighlight R %}NA, NA_integer_, NA_real_, NA_character_{% endihighlight %}. They will always be converted to the correct type using implicit coercion rules.

**Coerce vectors** from one type to another either implicitly by using a vector in a specific context that expects a certain type, or explicitly through a function such as {% ihighlight R %}as.logical(), as.double(), as.character(){% endihighlight %}. In addition, R will implicitly coerce the length of vectors by recycling the shorter vector to the same length as the longer vector.

**Check the vector** type with the {% ihighlight R %}is_*{% endihighlight %} family of functions for each type; for example, {% ihighlight R %}is_logical(), is_scalar_logical(){% endihighlight %}.

**Name vectors** during creation with {% ihighlight R %}c(){% endihighlight %} or after creation with {% ihighlight R %}set_names(){% endihighlight %} to provide for easier subsetting.

**Subset vectors** using brackets, {% ihighlight R %}[{% endihighlight %}.Vectors can be subsetted with numeric vectors to denote position, logical vectors to keep all values corresponding to a TRUE value, character vectors to denote names, and empty brackets to select rows or columns of matrices.

### 2. Lists
Lists can be created with {% ihighlight R %}list(){% endihighlight %}, and you can check the structure of a list using {% ihighlight R %}str(){% endihighlight %}. To subset lists,
1. {% ihighlight R %}[{% endihighlight %} extracts a new, smaller list.
2. {% ihighlight R %}[[{% endihighlight %} extracts a single component from a list, drilling down and removing a level of hierarchy.
3. {% ihighlight R %}${% endihighlight %} extracts named elements of a list.

Lists also have arbitrary additional attributes that can be viewed and set individually with {% ihighlight R %}attr(){% endihighlight %}, or collectively with {% ihighlight R %}attributes(){% endihighlight %}. The three fundamental attributes of lists are
1. Names, used to name the elements of a vector
2. Dimensions, which makes a vector behave like a matrix or array
3. Class, used to implement the S3 object oriented system and controls the behavior of generic functions such as {% ihighlight R %}as.Date{% endihighlight %}, based on different classes of input.

---





