---
layout: post
title: "Data Wrangling in R: Factors"
description: "In which we review some basic information about factors, a type of data used work with categorical variables."
thumb_image: 
tags: [academic, r]
---

### What are factors, and why should we use them?

Factors are a data type designed to work with categorical variables, which have a fixed and known set of possible values.[^1] They are more convenient to work with than strings for two main reasons: they help with sorting data, and help with identifying valid categories (i.e. prevent typos).

[^1]: This post is meant for a person who is looking for a refresher on factors in R, and the content in this post is based on chapter fifteen of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund.

##### 1. Factors help with sorting data

Say we are investigating an issue that affects people differently at stages of their life, and we have a variable for the stage of life that people are in -- infants, toddlers, children, adolescents, adults, and seniors. If we try to sort them as strings, they will be sorted alphabetically, but if we turn them into factors, they can be sorted in the proper age order.

{% highlight R %}
# a list of stages of life will be sorted in alphabetical order
people_sample <- c("adults", "children", "toddlers")
sort(people_sample)
#> [1] "adults"   "children" "toddlers"

# when we treat them as factors, they become sortable in order of age group
age_levels <- c("infants", "toddlers", "children", # create factor levels
  "adolescents", "adults", "seniors") 
people_sample_factors <- factor(people_sample, levels = age_levels) # convert to factor
sort(people_sample_factors)
#> [1] toddlers children adults  
#> Levels: infants toddlers children adolescents adults seniors
{% endhighlight %}

##### 2. Factors help with identifying valid categories
Considering the same example, let's take a slightly different initial list of stages of life with one category that doesn't belong and one typo in the data. However, if we convert these strings into factors, they are replaced with missing values (NA).

{% highlight R %}
people_sample_2 <- c("adults", "strangers", "todders")

# convert using factors(), values turn into NAs
(people_sample_2_factors <- factor(people_sample_2, levels = age_levels))
#> [1] adults <NA>   <NA>  
#> Levels: infants toddlers children adolescents adults seniors

# convert using parse_factor(), values turn into NAs and gives a warning
people_sample_2_factors <- parse_factor(people_sample_2, levels = age_levels)
#> Warning: 2 parsing failures.
#> row col           expected    actual
#>  2  -- value in level set strangers
#>  3  -- value in level set todders  
{% endhighlight %}

##### Additional notes:
* If you do not specify factor levels, then factors are created from the data in alphabetical order. 
* If you would like to match the order of levels with the order of appearance in the data, then set levels as follows: {% ihighlight R %}factor(data, levels = unique(data)){% endihighlight %}

## Working with Factors
---
We can reorder factors with these functions (often useful for visualizations):

* {% ihighlight R %}fct_inorder(){% endihighlight %} orders factors in order of their appearance in the data
* {% ihighlight R %}fct_reorder(){% endihighlight %} orders factors based on other variables
* {% ihighlight R %}fct_relevel(){% endihighlight %} brings specified factors to the beginning of the list of factors
* {% ihighlight R %}fct_infreq(){% endihighlight %} orders factors based on its frequency
* {% ihighlight R %}fct_rev{% endihighlight %} reverses the order of factor levels

And we can modify factors with these functions:

* {% ihighlight R %}fct_recode(){% endihighlight %} changes the values of each factor level
* {% ihighlight R %}fct_collapse(){% endihighlight %} collapses many factor levels into fewer levels
* {% ihighlight R %}fct_lump(){% endihighlight %} lumps together the least or most common factor levels into an "other" category

We'll explore examples of ordering and modifying factors using the {% ihighlight R %}gss_cat{% endihighlight %} dataset, a sample of categorical values from the General Social survey that comes from the {% ihighlight R %}forcats{% endihighlight %} package in the tidyverse. Here is a preview of the data:

{% highlight R %}
head(gss_cat, 3)
#> A tibble: 3 x 9
#>    year marital         age race  rincome        partyid            relig      denom            tvhours
#>   <int> <fct>         <int> <fct> <fct>          <fct>              <fct>      <fct>              <int>
#> 1  2000 Never married    26 White $8000 to 9999  Ind,near rep       Protestant Southern baptist      12
#> 2  2000 Divorced         48 White $8000 to 9999  Not str republican Protestant Baptist-dk which      NA
#> 3  2000 Widowed          67 White Not applicable Independent        Protestant No denomination        2
{% endhighlight %}

### Examples: Ordering Factors

**Order factors based on another variable when the factor is mapped to position** with {% ihighlight R %}fct_reorder(){% endihighlight %}:

{% highlight R %}
# summarize the data by each religion and how many hours of tv they watch
relig_summary <- gss_cat %>%
  group_by(relig) %>%
  summarise(
    age = mean(age, na.rm = TRUE),
    tvhours = mean(tvhours, na.rm = TRUE),
    n = n())

# graph religion by avg tv hours, and order religion based on tv hours
relig_summary %>%
  mutate(relig = fct_reorder(relig, tvhours)) %>% # reorder relig based on tvhours
  ggplot(aes(tvhours, relig)) +
  geom_point()
{% endhighlight %}

{% include image.html path="documentation/02-21-factors-fct-reorder.png"
                      path-detail="documentation/02-21-factors-fct-reorder.png"
                      alt="Factor Reorder" %}

**Order factors based on another variable when the factor is mapped to a non-position aesthetic** with {% ihighlight R %}fct_reorder2(){% endihighlight %}:

{% highlight R %}
# summarize the data by age, marital status, and proportion of age with marital status
by_age <- gss_cat %>%
  filter(!is.na(age)) %>%
  count(age, marital) %>%
  group_by(age) %>%
  mutate(prop = n / sum(n))

# graph age by proportion of marital status 
# order marital status (color) by age and proportion; makes legend easier to read
ggplot(by_age, aes(age, prop, colour = fct_reorder2(marital, age, prop))) + 
  geom_line() +
  labs(colour = "marital")
{% endhighlight %}

{% include image.html path="documentation/02-21-factors-fct-reorder2.png"
                      path-detail="documentation/02-21-factors-fct-reorder2.png"
                      alt="Factor Reorder 2" %}


**Bring specified factors to the beginning of the list of factors** with {% ihighlight R %}fct_relevel(){% endihighlight %}:


{% highlight R %}
# summarize the data by age and income
rincome_summary <- gss_cat %>%
  group_by(rincome) %>%
  summarise(
    age = mean(age, na.rm = TRUE),
    n = n()
  )

# graph average income levels by age, with N/A at the front of the list
ggplot(rincome_summary, 
       aes(age, fct_relevel(rincome, "Not applicable"))) + # move N/A to the front 
  geom_point()
{% endhighlight %}

{% include image.html path="documentation/02-21-factors-fct-relevel.png"
                      path-detail="documentation/02-21-factors-fct-relevel.png"
                      alt="Factor Relevel" %}

**Order factors based on its frequency** with {% ihighlight R %}fct_infreq(){% endihighlight %}:

{% highlight R %}
# graph the different marital statuses by increasing frequency
gss_cat %>%
  mutate(marital = marital %>% fct_infreq() %>% fct_rev()) %>%
  ggplot(aes(marital)) + geom_bar()
{% endhighlight %}

{% include image.html path="documentation/02-21-factors-fct-infreq.png"
                      path-detail="documentation/02-21-factors-fct-infreq.png"
                      alt="Factor Order by Freq" %}

### Examples: Modifying Factor Levels


**Change the values of each level** with {% ihighlight R %}fct_recode(){% endihighlight %}:

{% highlight R %}
# fct_recode() - change the values of each level
gss_cat %>%
  mutate(partyid = fct_recode(partyid,
                              "Republican, strong"    = "Strong republican",
                              "Republican, weak"      = "Not str republican",
                              "Independent, near rep" = "Ind,near rep",
                              "Independent, near dem" = "Ind,near dem",
                              "Democrat, weak"        = "Not str democrat",
                              "Democrat, strong"      = "Strong democrat"
  )) %>%
  count(partyid)
#> A tibble: 10 x 2
#>    partyid                   n
#>    <fct>                 <int>
#>  1 No answer               154
#>  2 Don't know                1
#>  3 Other party             393
#>  4 Republican, strong     2314
#>  5 Republican, weak       3032
#>  6 Independent, near rep  1791
#>  7 Independent            4119
#>  8 Independent, near dem  2499
#>  9 Democrat, weak         3690
#> 10 Democrat, strong       3490
{% endhighlight %}

**Collapse many specific levels into fewer levels** with {% ihighlight R %}fct_collapse(){% endihighlight %}:

{% highlight R %}gss_cat %>%
# fct_collapse() - collapse levels
  mutate(partyid = fct_collapse(partyid,
                                other = c("No answer", "Don't know", "Other party"),
                                rep = c("Strong republican", "Not str republican"),
                                ind = c("Ind,near rep", "Independent", "Ind,near dem"),
                                dem = c("Not str democrat", "Strong democrat")
  )) %>%
  count(partyid)
#> A tibble: 4 x 2
#>   partyid     n
#>   <fct>   <int>
#> 1 other     548
#> 2 rep      5346
#> 3 ind      8409
#> 4 dem      7180
{% endhighlight %}

**Lump together least or most common factor levels** with {% ihighlight R %}fct_lump(){% endihighlight %}:

{% highlight R %}
# fct_lump() - lumps groups togetherlumps together groups to make a plot or table simpler
gss_cat %>%
  mutate(relig = fct_lump(relig, n = 5)) %>%
  count(relig, sort = TRUE)
#> A tibble: 6 x 2
#>   relig          n
#>   <fct>      <int>
#> 1 Protestant 10846
#> 2 Catholic    5124
#> 3 None        3523
#> 4 Other        913
#> 5 Christian    689
#> 6 Jewish       388
{% endhighlight %}

