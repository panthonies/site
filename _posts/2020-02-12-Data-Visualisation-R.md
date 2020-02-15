---
layout: post
title: "Basic Data Visualization in R"
description: "In which we review the fundamentals of creating graphs in R with ggplot2."
thumb_image: 
tags: [academic, R]
---

In an effort to reduce the amount of time that I spend searching the Internet for basic ggplot2 questions, I'm writing a brief overview of the very basics on the [grammar of graphics](http://vita.had.co.nz/papers/layered-grammar.pdf) as a reference that I can come back to for a quick refresher.[^1]

[^1]: This post is meant for a person who has used ggplot2 in the past and is looking for a brief summary of the basics. The content in this post is based on chapter three of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund, which I would highly recommend reading in full if you have never used ggplot2 before.

**First things first:** The [RStudio ggplot2 Cheat Sheet](https://rstudio.com/wp-content/uploads/2015/03/ggplot2-cheatsheet.pdf) most likely has everything you need to know and more. To be honest, you might just be here for this link.

One last thing before we begin -- make sure that you have installed, updated, and loaded the [tidyverse](https://www.tidyverse.org) package. 

{% highlight R %}
### install packages
install.packages("tidyverse")

### update packages
tidyverse_update()

### load packages
library("tidyverse")
{% endhighlight %}

Now, let's get started.

# The General Structure of Graphs

{% highlight R %}
ggplot(data = <DATA>) + 
	<GEOM_FUNCTION>(
		mapping = aes(<MAPPINGS>)
		stat = <STAT>,
		position = <POSITION>)
	) +
	<COORDINATE_FUNCTION> +
	<FACET_FUNCTION>
{% endhighlight %}

### Data
Put your data in this parameter to creates a coordinate system and define the dataset.

### Geom_Function
This adds a layer of geometric shapes (points, bars, lines) that represent the dataset. Some examples are:

* GEOM_POINT creates a scatterplot
* GEOM_BAR creates a bar graph
* GEOM_SMOOTH creates a smooth line

#### Mappings
Mappings define how your variables are mapped to visual properties on the graph. They can be defined locally (inside of your geom_function) or globally (inside of the parent ggplot function). They are always defined by aesthetic properties, such as:

* x: the variable to map on the x-axis
* y: the variable to map on the y-axis
* color/fill: the [color](http://sape.inf.usi.ch/quick-reference/ggplot2/colour) of the data on the graph
* alpha: the transparency of data on the graph (from 0, transparent to 1, opaque)
* shape: the shape (numbers #1-20) of points on the graph
* size: the size of points on the graphs (in mm)
* stroke: the size of shape borders (in mm) 
* linetype: type of line to display on the graph

Note: you can map additional variables to color, alpha, etc. in addition to x and y, although whether this is actually a good idea depends on your data.

Each type of geometric function has a different set of available mappings, which can be found in the help documentation (i.e. by typing "?geom_point"). See the end of this post for quick mapping references.

#### Stat
Stat, or statistical transformation, are used to transform the data before graphing it. Each geometric function has a default statistical transformation -- the most common example is bar graphs computing and displaying a count of a variable in the data.

You may need to define a stat in these cases:
* to override the default stat of a geometric function. For example, using stat = "identity" for geom_bar if you already have a frequency variable in the data.
* to override the default mapping from transformed variables to aesthetics. For example, using geom_bar to display a proportion. 
* as an alternative to geom_function to build a layer for your graph (see the ggplot2 cheat sheet)


#### Position 
Position is used mainly for bar charts to help with displaying data. When you use color or fill to map a third variable in your data to different colors, there are a number of ways to position the additional information on your graph. The options include:

* by default, the bar chart will stack the bars 
* identity: creates overlapping bars (not that useful, but if you're doing it then use fill = NA)
* dodge: places bars next to one another (the most useful, in my opinion)
* fill: makes all of the bars the same height (if you don't care about the y-variable)

Note: geom_jitter() is a useful position adjustment for scatter plots to solve the problem of overplotting (where you have a lot of overlapping dots that aren't visible).

### Coordinate Function
Most likely, you won't be using this argument because the default Cartesian coordinate system will satisfy your needs. However, here are some common uses:

* coord_flip() switches the x and y axes
* coord_fixed() lets you define the ratio between your x and y axes (default: 1)


### Facet Function
Facets are subplots that are useful for visually separating your data by discrete variables. You can create facets in two main ways:

* facet_wrap(): splits the plot by a single discrete variable
* facet_grid(): splits the plot by a combination of two variables separated by ~

### Titles, Labels, and Axes
Even though these aspects are not a part of the basic structure of graphs, they are one of the most important. Nobody cares how great your graph looks if they don't know what it's meant to show.

The basics are best shown through example:

{% highlight R %}
# Example: Titles, Labels, and Axes
ggplot(data = mpg) +
  geom_point(mapping = aes(x = trans, y = hwy)) + 
  scale_x_discrete(name = "X-Axis Name") +
  scale_y_continuous(name = "Y-Axis Name", breaks = c(20, 30, 40), labels = c("Twenty", "Thirty", "Forty"), limits = c(10,50)) +
  labs(title = "Graph Title", subtitle = "Graph subtitle", caption = "Graph Caption")
{% endhighlight %}

# ...and we're finished! Not too bad, right?
I've included some useful references and example code below that illustrates the concepts of this post in practice.

## Quick References
A reference for ggplot2 point shapes: 
{% include image.html path="documentation/ggplot2_shapes.png" path-detail="documentation/ggplot2_shapes.png" alt="ggplot2 shapes" %}
A reference for ggplot2 line types:
{% include image.html path="documentation/ggplot2_linetypes.png" path-detail="documentation/ggplot2_linetypes.png" alt="ggplot2 line types" %}

## Basic Examples in R code
I recommend copying and pasting this code into RStudio for ease of use since the formatting on this page is not ideal. 
{% highlight R %}
### load package
library("tidyverse")

# using mpg data from tidyverse, we can create basic scatterplots
ggplot(data = mpg) + 
  geom_point(mapping = aes(x = displ, y = hwy, color = class))

ggplot(data = mpg) + 
  geom_point(mapping = aes(x = displ, y = hwy, color = class, alpha = class))

ggplot(data = mpg) + 
  geom_point(mapping = aes(x = displ, y = hwy, color = displ < 5), shape = 2)

# experimenting with the line geom
ggplot(data = mpg) + 
  geom_smooth(mapping = aes(x = displ, y = hwy))

ggplot(data = mpg) + 
  geom_smooth(mapping = aes(x = displ, y = hwy, linetype = drv))

# multiple geoms in one plot - local mapping
ggplot(data = mpg) + 
  geom_point(mapping = aes(x = displ, y = hwy)) +
  geom_smooth(mapping = aes(x = displ, y = hwy))

# multiple geoms in one plot - global mapping
ggplot(data = mpg, mapping = aes(x = displ, y = hwy)) + 
  geom_point() +
  geom_smooth()

# multiple geoms in one plot - both global and local mappings
ggplot(data = mpg, mapping = aes(x = displ, y = hwy)) + 
  geom_point(mapping = aes(color = class)) + 
  geom_smooth(se = FALSE)

# basic transformations - automatically done by bar graphs, histograms, and others
ggplot(data = diamonds) + 
  geom_bar(mapping = aes(x = cut))

# basic transformations - proportion graph
ggplot(data = diamonds) + 
  geom_bar(mapping = aes(x = cut, y = stat(prop), group = 1))

# adding color/fill
ggplot(data = diamonds) + 
  geom_bar(mapping = aes(x = cut, fill = clarity))

# use position =  "identity" for overlapping bars (recommend fill = NA)
ggplot(data = diamonds, mapping = aes(x = cut, color = clarity)) + 
  geom_bar(fill = NA, position = "identity")

# use position = "fill" to make all of the bars the same height
ggplot(data = diamonds) + 
  geom_bar(mapping = aes(x = cut, fill = clarity), position = "fill")

# use position = "dodge" to place bars beside one another
ggplot(data = diamonds) +
  geom_bar(mapping = aes(x = cut, fill = clarity), position = "dodge")

# for scatter plots, use jitter to solve overplotting (overlapping points)
ggplot(data = mpg) + 
  geom_point(mapping = aes(x = displ, y = hwy), position = "jitter")

ggplot(data = mpg) +
  geom_jitter(mapping = aes(x = displ, y = hwy))

ggplot(data = mpg) +
  geom_boxplot(mapping = aes(x = trans, y = hwy, group = trans))

# coordinate systems: coord(flip) switches x and y axis
ggplot(data = mpg, mapping = aes(x = class, y = hwy)) + 
  geom_boxplot() +
  coord_flip()

# using facets - subplots that display subsets of the data
ggplot(data = mpg) + 
  geom_point(mapping = aes(x = displ, y = hwy)) +
  facet_wrap(~class, nrow = 2)

ggplot(data = mpg) + 
  geom_point(mapping = aes(x = displ, y = hwy)) +
  facet_grid(drv ~ cyl)

ggplot(data = mpg) + 
  geom_point(mapping = aes(x = displ, y = hwy)) +
  facet_grid(drv ~ .)

# use labs to add labels to the chart
ggplot(data = diamonds) + 
  geom_bar(mapping = aes(x = cut, fill = clarity)) +
  labs(title = "Number of cuts of diamond and their clarity",
       subtitle = "Subtitle",
       xlab = "Type of Cut",
       xlab = "Count",
       caption = "Sample Caption")

# use scale_x_continuous and scale_y_continuous to set the axis
ggplot(data = mpg, mapping = aes(x = cty, y = hwy)) +
  geom_point() + 
  geom_abline() +
  coord_fixed() +
  scale_x_continuous(limits = c(0,NA)) +
  scale_y_continuous(limits = c(0,NA))
{% endhighlight %}

