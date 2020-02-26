---
layout: post
title: "Basic Data Exploration in R"
description: "In which we examine some common practical examples of data exploration: observing variance and co-variance with histograms, boxplots, scatterplots, and heat maps."
thumb_image: 
tags: [academic, r]
---

In this post we'll be taking a look at several examples of data exploration using the {% ihighlight R %}diamonds{% endihighlight %} dataset in the tidyverse library.[^1]

[^1]: This post is meant for a person who is wondering how to apply basic knowledge of R to explore datasets. Its contents are based on chapter seven of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund.

{% highlight R %}
### load package
library(tidyverse)
{% endhighlight %}

## When investigating variance within a single variable, ask:
* Which values are most common, most rare, and why? 
* Does this match your expectations?
* Are there any unusual patterns, and what are the possible explanations?
* If there are clusters of observations, how are those observations similar or different?

**Example 1:** Use a histogram to visualize the distribution of a variable. Here we observe the distribution of the carat (weight) of diamonds.
{% highlight R %}
ggplot(data = diamonds) +
  geom_histogram(mapping = aes(x = carat), binwidth = .5)
{% endhighlight %}
{% include image.html path="documentation/02-16-variance-histogram.png"
                      path-detail="documentation/02-16-variance-histogram.png"
                      alt="Variance Histogram" %}

**Example 2:** Play with the binwidth of histograms to reveal different patterns in the data. In this example, we get a more accurate view of the distribution of carat.
{% highlight R %}
smaller <- diamonds %>%
  filter(carat < 3)
ggplot(data = smaller) +
  geom_histogram(mapping = aes(x = carat), binwidth = .1)
{% endhighlight %}
{% include image.html path="documentation/02-16-variance-histogram-binwidth.png"
                      path-detail="documentation/02-16-variance-histogram-binwidth.png"
                      alt="Variance Histogram Binwidth" %}

**Example 3:** Play with the x-axis and y-axis limits to identify outliers. In this example, there are some very high values of the "x" variable, a measure of a diamond's dimensions, which could indicate data entry errors. 

Note that in ggplot2, {% ihighlight R %}coord_cartesian(){% endihighlight %} keeps truncated values while {% ihighlight R %}xlim(), ylim(){% endihighlight %} discards them.
{% highlight R %}
ggplot(diamonds) + 
  geom_histogram(mapping = aes(x = y), binwidth = 0.5) +
  coord_cartesian(ylim = c(0, 50))

# Note - a simple way to deal with outliers is to replace them with NA's
#   using mutate(), ifelse(), case_when(). 
# For example:
diamonds2 <- diamonds %>%
  mutate(y = ifelse(y < 3 | y > 20, NA, y))
{% endhighlight %}
{% include image.html path="documentation/02-16-variance-outliers.png"
                      path-detail="documentation/02-16-variance-outliers.png"
                      alt="Variance Histogram Outliers" %}

## When investigating covariance between two variables, ask:
* Could this pattern be due to random chance? 
* What is the relationship implied by the pattern, and how strong is it?
* What other variables might affect the relationship?
* Does the relationship change if you look at subgroups of data?

### Case 1: A Categorical and a Continuous Variable
**Example 1:** Use {% ihighlight R %}geom_freqpoly(){% endihighlight %} to overlay multiple histograms by density. Here we can see multiple histograms of carat, split by cut.
{% highlight R %}
ggplot(data = smaller, mapping = aes(x = carat, color = cut)) +
  geom_freqpoly(binwidth = 0.1)
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-freqpoly-density.png"
                      path-detail="documentation/02-16-covariance-freqpoly-density.png"
                      alt="Covariance Freqpoly Density" %}

**Example 2:** Another way to visualize the relationship is to create a boxplot, order the variables, and flip the axes. This is illustrated with the relationship between car class and highway miles per gallon via the mpg dataset. 
{% highlight R %}
ggplot(data = mpg) +
  geom_boxplot(mapping = aes(x = reorder(class, hwy, FUN = median), y = hwy)) +
  coord_flip()
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-boxplot.png"
                      path-detail="documentation/02-16-covariance-boxplot.png"
                      alt="Covariance Boxplot" %}

**Example 3:** In cases where there are a lot of outliers, boxplots may not be as useful. Instead, we can use a letter value plot. Here we have a good view of the density of observations when relating cut and price of diamonds.
{% highlight R %}
ggplot(diamonds) +
  geom_lv(aes(x = cut, y = price))
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-lvplot.png"
                      path-detail="documentation/02-16-covariance-lvplot.png"
                      alt="Covariance LV Plot" %}

**Example 4:** The violin plot is another great way to compare density distributions among different categories. This is a different way to represent the data from the previous graph.
{% highlight R %}
ggplot(diamonds) +
  geom_violin(aes(x = cut, y = price))
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-violin.png"
                      path-detail="documentation/02-16-covariance-violin.png"
                      alt="Covariance Violin" %}

### Case 2: Two Categorical Variables

**Example 1:** We can use {% ihighlight R %}geom_count{% endihighlight %} to map both categorical variables and display their frequency with the size of a point in a grid. In this example, we can see the distribution of observations between diamond cut and color. 
{% highlight R %}
ggplot(data = diamonds) +
  geom_count(mapping = aes(x = cut, y = color))
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-count.png"
                      path-detail="documentation/02-16-covariance-count.png"
                      alt="Covariance Count" %}

**Example 2:** A heat map can also come in handy when comparing density of observations between two categorical variables. Again, we can see the distribution of observations between diamond cut and color.
{% highlight R %}
diamonds %>%
  count(color, cut) %>%
  ggplot(mapping = aes(x = color, y = cut)) +
    geom_tile(mapping = aes(fill = n))
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-tile.png"
                      path-detail="documentation/02-16-covariance-tile.png"
                      alt="Covariance Tile" %}


### Case 3: Two Continuous Variables

**Example 1:** The scatterplot is a classic way to compare continuous variables. For example, here we examine the relationship between diamond carat and price.
{% highlight R %}
ggplot(data = diamonds) + 
  geom_point(mapping = aes(x = carat, y = price), alpha = 1 / 100)
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-scatter.png"
                      path-detail="documentation/02-16-covariance-scatter.png"
                      alt="Covariance Scatter" %}

**Example 2:** We can also bin variables in two dimensions, using the fill color to represent frequency of observations. 
{% highlight R %}
# Note: both geom_bin2d (rectagles) and geom_hex (hexagons) will work here
ggplot(data = smaller) +
  geom_hex(mapping = aes(x = carat, y = price))
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-hex.png"
                      path-detail="documentation/02-16-covariance-hex.png"
                      alt="Covariance Hex" %}

**Example 3:** Another option is to bin only one continuous variables so that it behaves like a categorical variable. Here, we bin carat.
{% highlight R %}
ggplot(data = smaller, mapping = aes(x = carat, y = price)) + 
  geom_boxplot(mapping = aes(group = cut_width(carat, 0.1)))
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-cutwidth.png"
                      path-detail="documentation/02-16-covariance-cutwidth.png"
                      alt="Covariance Cutwidth" %}

**Example 4:** This is the same graph, except the width of the boxplot is proportional to the number of points.
{% highlight R %}
ggplot(data = smaller, mapping = aes(x = carat, y = price)) + 
  geom_boxplot(mapping = aes(group = cut_width(carat, 0.1)), varwidth = TRUE)
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-cutwidth-varwidth.png"
                      path-detail="documentation/02-16-covariance-cutwidth-varwidth.png"
                      alt="Covariance Varwidth" %}

**Example 5:** This is again the same graph, but each box now contains approximately the same number of points.
{% highlight R %}
ggplot(data = smaller, mapping = aes(x = carat, y = price)) + 
  geom_boxplot(mapping = aes(group = cut_number(carat, 20)))
{% endhighlight %}
{% include image.html path="documentation/02-16-covariance-cutnumber.png"
                      path-detail="documentation/02-16-covariance-cutnumber.png"
                      alt="Covariance Cutnumber" %}

