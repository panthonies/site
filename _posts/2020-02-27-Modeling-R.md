---
layout: post
title: "Model Building in R"
description: "In which we explore the basics of modeling as an exploratory tool through recording and graphing predictions and residuals, variable interactions, and transformations."
thumb_image: 
tags: [academic, r]
---

This post provides an introduction to modeling in R without going into statistical details.[^1] We'll go over some examples of fitting models to data, and then examine the list-column data structure. The following libraries are used:

[^1]: This post is meant for a person who is looking for a refresher on basic modeling in R. The content in this post is based on chapter twenty-two through twenty-five of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund.

{% highlight R %}
library("tidyverse")
library("modelr")              # tools for modeling
library("splines")             # tools for modeling with polynomials
library("broom")               # tools for turning models into tidy data frames
library("gapminder")           # dataset with country descriptive data over time
options(na.action = na.warn) # warns you if models drop data
{% endhighlight %}

## Model Basics

Let's go through an example exploratory workflow of fitting a model to a continuous variable using simulated dataset {% ihighlight R %}sim1{% endihighlight %}:

##### 1. **Graph the initial data.**
{% highlight R %}
ggplot(sim1, aes(x, y)) +
  geom_point()
{% endhighlight %}
{% include image.html path="documentation/02-28-cont-graph.png"
                      path-detail="documentation/02-28-cont-graph.png"
                      alt="Graph Continuous Variable" %}

{:start="2"}
##### 2. **Fit a simple linear regression to the data.**
{% highlight R %}
sim1_mod <- lm(y ~ x, data = sim1)
{% endhighlight %}

**Note:** To model an interaction between x-variables, use {% ihighlight R %}*{% endihighlight %}

{:start="3"}
##### 3. **Create a new data frame with model prediction data.**
{% highlight R %}
grid <- sim1 %>% 
  data_grid(x) %>%
  add_predictions(sim1_mod) 
{% endhighlight %}

**Note:** {% ihighlight R %}add_predictions(){% endihighlight %} adds a single new column with model predictions, {% ihighlight R %}spread_predictions(){% endihighlight %} adds one column for each model, and {% ihighlight R %}gather_predictions(){% endihighlight %} two columns, model and prediction, and repeats the input rows for each model.

{:start="4"}
##### 4. **Graph the initial data with the fitted model.**
{% highlight R %}
ggplot(sim1, aes(x)) +
  geom_point(aes(y=y)) +
  geom_line(data = grid, aes(y = pred), color = "red", size = 1)
{% endhighlight %}
{% include image.html path="documentation/02-28-cont-graph-2.png"
                      path-detail="documentation/02-28-cont-graph-2.png"
                      alt="Graph Continuous Variable with Predictions" %}

{:start="5"}
##### 5. **Add the residuals to the initial data.**
{% highlight R %}
sim1 <- sim1 %>%
  add_residuals(sim1_mod)
{% endhighlight %}

{:start="6"}
##### 6. **Graph the initial x-values against their residuals.**
{% highlight R %}
ggplot(sim1, aes(x, resid)) + 
  geom_ref_line(h = 0) + 
  geom_point()
{% endhighlight %}
{% include image.html path="documentation/02-28-cont-graph-3.png"
                      path-detail="documentation/02-28-cont-graph-3.png"
                      alt="Graph Continuous Variable Resids" %}

## Transformations

Transformations can be perfromed inside of the model formula, but if it contains an operation of {% ihighlight R %}+, -, *, ^,{% endihighlight %} wrap it in {% ihighlight R %}I(){% endihighlight %} so it doesn't become part of the model specs. Let's go through an example workflow again, this time with a polynomial transformation.

##### 1. **Create and graph the initial data.**
{% highlight R %}
sim2 <- tibble(
  x = seq(0, 3.5 * pi, length = 50),
  y = 4 * sin(x) + rnorm(length(x))
)

ggplot(sim2, aes(x, y)) +
  geom_point()
{% endhighlight %}
{% include image.html path="documentation/02-28-trans-graph.png"
                      path-detail="documentation/02-28-trans-graph.png"
                      alt="Graph Data" %}

{:start="2"}
##### 2. **Fit linear regressions with polynomials up to degree five to the data.**
{% highlight R %}
mod1 <- lm(y ~ ns(x, 1), data = sim2)
mod2 <- lm(y ~ ns(x, 2), data = sim2)
mod3 <- lm(y ~ ns(x, 3), data = sim2)
mod4 <- lm(y ~ ns(x, 4), data = sim2)
mod5 <- lm(y ~ ns(x, 5), data = sim2)
{% endhighlight %}

{:start="3"}
##### 3. **Put predictions into a data frame.**
{% highlight R %}
grid <- sim5 %>% 
  data_grid(x = seq_range(x, n = 50, expand = 0.1)) %>% 
  gather_predictions(mod1, mod2, mod3, mod4, mod5, .pred = "y")
{% endhighlight %}

**Note:** {% ihighlight R %}seq_range(){% endihighlight %} provides a specified number of values between the minimum and maximum of a variable, which can be useful for graphing.

{:start="4"}
##### 4. **Graph the initial data with the fitted models.**
{% highlight R %}
ggplot(sim2, aes(x, y)) + 
  geom_point() +
  geom_line(data = grid, colour = "red") +
  facet_wrap(~ model, nrow = 1)
{% endhighlight %}
{% include image.html path="documentation/02-28-trans-graph-2.png"
                      path-detail="documentation/02-28-trans-graph-2.png"
                      alt="Graph Data with Predictions" %}

{:start="5"}
##### 5. **Add the residuals to the initial data.**
{% highlight R %}
sim5 <- sim5 %>%
  gather_residuals(mod1, mod2, mod3, mod4, mod5)
{% endhighlight %}

{:start="6"}
##### 6. **Graph the initial x-values against their residuals.**
{% highlight R %}
ggplot(sim5, aes(x, resid)) +
  geom_hline(yintercept = 0, size = 2, color = "white") +
  geom_point() +
  facet_wrap(~model, nrow = 1)
{% endhighlight %}
{% include image.html path="documentation/02-28-trans-graph-3.png"
                      path-detail="documentation/02-28-trans-graph-3.png"
                      alt="Graph Data Residuals" %}

## Many Models

If you have a complex dataset, it may be possible to unpack the data using many simple models. For example, the *gapminder* dataset contains data on the life expectancy, among other variables, in many countries over the course of 50 years. With this data, let's explore how life expectancy changes over time for each country, and dig into which countries deviate significantly from the rest of the world.

##### 1. **Graph the initial data.**
{% highlight R %}
gapminder %>%
  ggplot(aes(year, lifeExp, group = country)) + 
  geom_line(alpha = 1/3)
{% endhighlight %}
{% include image.html path="documentation/02-28-many-graph.png"
                      path-detail="documentation/02-28-many-graph.png"
                      alt="Graph Data Many Models" %}

{:start="2"}
##### 2. **Nest the data frame to create one row for each country and continent, with a new column of data frames containing the rest of the country information.**
{% highlight R %}
by_country <- gapminder %>%
  group_by(country, continent) %>%
  nest()
{% endhighlight %}

{:start="3"}
##### 3. **Create a linear model and run it over each country, storing the results in a new column.**
{% highlight R %}
# create a linear model function 
country_model <- function(df) {
  lm(lifeExp ~ year, data = df)
}

# run models and store them in a new list-column; helpful for filtering and arranging
by_country <- by_country %>%
  mutate(model = map(data, country_model))
{% endhighlight %}

{:start="4"}
##### 4. **Add residuals to the data.**
{% highlight R %}
by_country <- by_country %>%
  mutate(resids = map2(data, model, add_residuals))
{% endhighlight %}

{:start="5"}
##### 5. **Unnest and plot the residuals by continent. Notice that the model performs the worst in Africa.**
{% highlight R %}
resids <- unnest(by_country, resids)
resids %>% 
  ggplot(aes(year, resid, group = country)) +
  geom_line(alpha = 1 / 3) + 
  facet_wrap(~continent, nrow = 1)
{% endhighlight %}
{% include image.html path="documentation/02-28-many-resids.png"
                      path-detail="documentation/02-28-many-resids.png"
                      alt="Graph Data Many Residuals" %}



{:start="7"}
##### 7. **Unpack the data by using glance and unnest to skim and add model data to the data frame.**
{% highlight R %}
glance <- by_country %>%
  mutate(glance = map(model, glance)) %>%
  unnest(glance, .drop = TRUE)
{% endhighlight %}

{:start="6"}
##### 8. **Graph the R-Squared coefficient to investigate where the model breaks down.**
{% highlight R %}
glance %>%
  arrange(r.squared) %>%
  ggplot(aes(continent, r.squared)) +
  geom_jitter(width = 0.5)
{% endhighlight %}
{% include image.html path="documentation/02-28-many-rsq.png"
                      path-detail="documentation/02-28-many-rsq.png"
                      alt="Graph Data Many R-Squared" %}


{:start="6"}
##### 9. **Pull out the problem countries into a separate table. Then, plot the life expectancies of those countries over time by joining the table with the original data.**
{% highlight R %}
bad_fit <- glance %>%
  filter(r.squared < .25)
gapminder %>%
  semi_join(bad_fit, by = "country") %>%
  ggplot(aes(year, lifeExp, color = country)) +
    geom_line()
{% endhighlight %}

**Note:** History provides an explanation where the data breaks down: this graph reveals the devastating effects of the Rwandan genocide and HIV/AIDS in African countries in the 1990s.

{% include image.html path="documentation/02-28-many-drilldown.png"
                      path-detail="documentation/02-28-many-drilldown.png"
                      alt="Graph Data Many Drilldown" %}

## Data Structures: List-Columns

The life expectancy example above made use of list-column data structures. In general, an effective list-column pipeline will take the following form:
1. Create the list-column.
2. Create other intermediate list-columns by transforming existing list columns.
3. Simplify the list-column back down to a data frame or atomic vector. 

### Creating List-Columns

* {% ihighlight R %}nest(){% endihighlight %} converts a grouped data frame into a nested data frame with a list-column of data frames.
* {% ihighlight R %}mutate(){% endihighlight %} applied with vectorized functions that return a list will create list-columns.
* {% ihighlight R %}summarize(){% endihighlight %} applied with summary functions that return multiple results will create list-columns.

{% highlight R %}
# summarize the quantiles of miles per gallon by the number of cylinders in the car
probs <- c(0.01, 0.25, 0.5, 0.75, 0.99)
mtcars %>%
  group_by(cyl) %>%
  summarize(p = list(probs), q = list(quantile(mpg, probs))) %>%
  unnest()
#>  A tibble: 15 x 3
#>     cyl     p     q
#>   <dbl> <dbl> <dbl>
#> 1     4  0.01  21.4
#> 2     4  0.25  22.8
#> 3     4  0.5   26  
#> 4     4  0.75  30.4
#> 5     4  0.99  33.8
#> 6     6  0.01  17.8
#> 7     6  0.25  18.6
#> ...
{% endhighlight %}

### Simplifying List-Columns
In order to manipulate and visualize the data, you will need to simplify list-columns.

* If you want a single value from the list-column, use {% ihighlight R %}mutate(){% endihighlight %} with {% ihighlight R %}map_lgl(), map_int(), map_dbl(), map_chr(){% endihighlight %} to create an atomic vector.
* If you want many values from the list-column, use {% ihighlight R %}unnest(){% endihighlight %} to convert list columns back to regular columns, repeating the rows as many times as necessary.

{% highlight R %}
# extract a single value by referring to its element name
df <- tribble(
  ~x,
  list(a = 1, b = 2),
  list(a = 2, c = 4))

df %>% mutate(
  a = map_dbl(x, "a"),
  b = map_dbl(x, "b", .null = NA_real_))
#> A tibble: 2 x 3
#>   x                    a     b
#>   <list>           <dbl> <dbl>
#> 1 <named list [2]>     1     2
#> 2 <named list [2]>     2    NA
#> {% endhighlight %}

### Turning Models into Tidy Data 
The following three functions help turn models into tidy data, and often make use of list-columns.

* {% ihighlight R %}glance(){% endihighlight %} returns a row for each model, where each column gives a model summary.
* {% ihighlight R %}tidy(){% endihighlight %} returns a row for each coefficient in the model, where each column has info about estimate/variability.
* {% ihighlight R %}augment(){% endihighlight %} returns a row for each row in data, adding extra values like residuals and influence stats.





