---
layout: post
title: "Data Wrangling in R: Strings"
description: "In which we dive into string manipulation, with a focus on regular expressions."
thumb_image: 
tags: [academic, r]
---

In this post we'll be taking a look at basic string functions, regular expression syntax, and several applications of regular expressions in string manipulation.[^1]

[^1]: This post is meant for a person who is looking for a refresher on string manipulation and regular expressions in R. The content in this post is based on chapter fourteen of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund, which I would recommend reading for in-depth examples.

The [RStudio String Manipulation Cheat Sheet]({{ site.baseurl }}/pdf/r-cheat-sheet-strings.pdf){:target="blank"} has great reference material on this topic in a condensed format. 

### String Basics

* `str_length()` returns the number of characters in a string 
* `str_replace_na()` turns missing values (NA) into "NA" 
* `str_c(..., sep = ",")` combines two or more strings with a specified separator
* `str_c(..., collapse = ", "` collapses a vector of strings into a single string with a specified separator
* `str_sub()` extracts or modifies subsets of a string by character position
* `str_sort()` sorts strings; specify locale if necessary
* `str_to_lower(), str_to_upper(), str_to_title()` changes cases; specify locale if necessary

## Regular Expressions - Basic Syntax
---
#### Anchors
* `^` matches the start of a string
* `$` matches the end of a string

#### Character Matching
* `.` matches any character
* `\d` matches any digit
* `\s` matches any whitespace
* `[abc]` matches a, b, or c
* `[^abc]` matches anything except a, b, or c

**Note:** In order to explicitly search for a metacharacter in a regular expression, `^, $, ., \, etc...`, you must add a backslash in front of it.

#### Repetition
* `?` is 0 or 1
* `+` is 1 or more
* `*` is 0 or more
* `{n}` is exactly n
* `{n,}` is n or more
* `{,m}` is at most m
* `{n,m}` is between n and m, inclusive
* `\1, \2` backreference previous text in parentheses and search for the same pattern

**Note:** By default, these are greedy and match the longest string possible; make them lazy by putting a `?` after them.

## Appling Regular Expressions in R

It's important to note that in order for your regular expressions to work in R, you must add an additional backslash, ` \ `, to all existing backslashes in your expression. This is because backslashes have their own meaning in R strings.

You can also search for patterns using OR logic using the pipe: `|`.

Let's go through some simple applications of regular expressions using the two libraries below. The two datasets that I will use as examples are `words`, a character vector of 980 common words, and `sentences`, a character vector of 720 sentences.

{% highlight R %}
library("tidyverse") # contains stringr package
library("htmlwidgets") # contains str_view() function
{% endhighlight %}

### 1. Detecting Matches: **string_detect()**, **string_subset()**, and **string_count()**

`string_detect()` searches for a pattern in a string and returns TRUE or FALSE.

{% highlight R %}
# how many common words start with t?
sum(str_detect(words, "^t"))
#> [1] 65

# filter on words that end with x in a tibble
df <- tibble(
  word = words, 
  i = seq_along(word))
df %>%
  filter(str_detect(word, "x$"))
#> A tibble: 4 x 2
#>  word      i
#>  <chr> <int>
#>1 box     108
#>2 sex     747
#>3 six     772
#>4 tax     841
{% endhighlight %}

`string_subset()` keeps strings matching a pattern.

{% highlight R %}
# filter on words that end with x in a vector
str_subset(words, "x$")
#>[1] "box" "sex" "six" "tax"
{% endhighlight %}

`string_count()` counts the number of matches there are in a string.

{% highlight R %}
# how many consonants and vowels are there in each word?
df %>%
  mutate(
    vowels = str_count(word, "[aeiou]"),
    consonants = str_count(word, "[^aeiou]"))
#> A tibble: 980 x 4
#>   word         i vowels consonants
#>   <chr>    <int>  <int>      <int>
#> 1 a            1      1          0
#> 2 able         2      2          2
#> 3 about        3      3          2
#> 4 absolute     4      4          4
#> ...
{% endhighlight %}

### 2. Extracting Matches: **string_extract()** and **string_extract_all()**

`str_extract(), str_extract_all()` extracts the actual text of a match.

{% highlight R %}
## find colors from a vector listed in "sentences" 
colors <- c("red", "orange", "yellow", "green", "blue", "purple") %>% # colors to find
	str_c(collapse = "|") # create regex with colors %>%
more <- sentences[str_count(sentences, colors) > 1] # filter on sentences with colors
str_view_all(more, color_match) # view sentences with colors highlighted

# view all colors in each matched sentence
str_extract_all(more, color_match) 
#>[[1]]
#>[1] "blue" "red" 
#>
#>[[2]]
#>[1] "green" "red"  
#>
#>[[3]]
#>[1] "orange" "red"   

# use simplify = TRUE: view all colors in each matched sentence in a matrix
str_extract_all(more, color_match, simplify = TRUE)
#>     [,1]     [,2] 
#>[1,] "blue"   "red"
#>[2,] "green"  "red"
#>[3,] "orange" "red"
{% endhighlight %}

### 3. Grouped Matches: **str_match()** and **str_match_all()**

`str_match() and str_match_all()` are very similar to the previous string extracting functions -- they extract a matching pattern from a vector, but also returns each individual component by returning a matric with one column for the complete match followed by one column for each group.

{% highlight R %}
# find "articles" and "nouns" in a vector of sentences
noun <- "(a|the) ([^ ]+) "
has_noun <- sentences %>%
  str_subset(noun) %>%
  head(10)
has_noun %>% 
  str_match(noun)
#>      [,1]          [,2]  [,3]     
#> [1,] "the smooth " "the" "smooth" 
#> [2,] "the sheet "  "the" "sheet"  
#> [3,] "the depth "  "the" "depth"  
#> ...
{% endhighlight %} 

`extract()` does the same thing, but is especially useful for tibbles (as opposed to vectors). It will add additional columns to the tibble for each grouped match.

{% highlight R %}
# find "articles" and "nouns" in a tibble with a column containing sentences
noun <- "(a|the) ([^ ]+) "
tibble(sentence = sentences) %>% 
  tidyr::extract(
    sentence, c("article", "noun"), "(a|the) ([^ ]+)", 
    remove = FALSE
  )
#> A tibble: 720 x 3
#>   sentence                                    article noun   
#>   <chr>                                       <chr>   <chr>  
#> 1 The birch canoe slid on the smooth planks.  the     smooth 
#> 2 Glue the sheet to the dark blue background. the     sheet  
#> 3 It's easy to tell the depth of a well.      the     depth  
#> ...
{% endhighlight %}


### 4. Replacing Matches: **str_replace()** and **str_replace_all()**

`str_replace()` will replace the first occurence of a match, while `str_replace_all()` will replace all occurences.

{% highlight R %}
# replacing the first match
x <- c("apple", "pear", "banana")
str_replace(x, "[aeiou]", "-")
#> [1] "-pple"  "p-ar"   "b-nana"

# replacing all matches
str_replace_all(x, "[aeiou]", "-")
#>[1] "-ppl-"  "p--r"   "b-n-n-"
{% endhighlight %}

### 5. Splitting Matches: **str_split()**

`str_split()` will split strings based on a pattern.

{% highlight R %}
## splitting a string by words
sentences %>%
  head(3) %>% 
  str_split(boundary("word"), simplify = TRUE)
#>     [,1]    [,2]    [,3]    [,4]      [,5]  [,6]    [,7]     [,8]          [,9]   
#>[1,] "The"   "birch" "canoe" "slid"    "on"  "the"   "smooth" "planks."     ""     
#>[2,] "Glue"  "the"   "sheet" "to"      "the" "dark"  "blue"   "background." ""     
#>[3,] "It's"  "easy"  "to"    "tell"    "the" "depth" "of"     "a"           "well."
{% endhighlight %}

### 6. Finding the Positions of Matches: **str_locate()**, **str_locate_all()**

`str_locate(), str_locate_all()` return the starting and ending positions of each match. When none of the other functions do what you want, you may want to locate the positions of the matching patterns, then use `str_sub()` to extract/modify them.

### ...A final note about regular expressions:

In the examples above, the pattern matching string is automatically wrapped into a call to `regex()`:

{% highlight R %}
# The regular call:
str_detect(fruit, "banana")
# Is shorthand for
str_detect(fruit, regex("banana"))
{% endhighlight %}

We can explicitly call the `regex()` function to **change case matching**, **search over multiple lines**, or **add comments for readability.**

{% highlight R %}
# create a regular expression that finds all of these bananas
bananas <- c("\nbanana", "Banana", "BANANA")
str_view(bananas, regex("^banana # search for all bananas", 
                        ignore_case = TRUE,
                        multiline = TRUE,
                        comments = TRUE))
{% endhighlight %}

## Applying Pattern Matching Without Regular Expressions
---
All of the functions that we've looked at to apply pattern matching via regular expressions by default. However, it is possible to override the pattern matching type by explicity specifying one of three functions in place of `regex()`:

1. `fixed()` matches the exact specified sequence of bytes, ignoring all special regular expressions. It is much faster than regular expressions, but be careful with non-English data.
2. `coll()` compares strings using standard collation rules. This is useful for doing case insensitive matching, but is slower than the other functions.
3. `boundary()` can match boundaries, such as characters, words, or sentences.

