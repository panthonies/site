---
layout: post
title: "Creating Simple Documents with R Markdown"
description: "In which we review the basics of R Markdown files, including the YAML header, code chunk options, and formatting options."
thumb_image: 
tags: [academic, r]
---

This post provides a basic introduction to R Markdown.[^1] An R Markdown file contains three essential ingredients -- a YAML header to define document-wide settings, chunks of R code, and markdown text. They are designed to be used:

1. For communicating to those who want to focus on the conclusions, not the code behind the analysis.
2. For collaborating with those who are interested in both the conclusions and how they were reached.
3. For capturing the work you do and what you were thinking at the time.

[^1]: This post is meant for a person who is looking for a refresher on R Markdown. The content in this post is based on chapters twenty-seven through thirty of [R for Data Science](https://r4ds.had.co.nz/index.html) by Hadley Wickham & Garrett Grolemund.


#### Resources:

- [R Markdown Cheat Sheet]({{ site.baseurl }}/pdf/r-markdown-cheat-sheet.pdf){:target="blank"}: a quick reference
- [R Markdown Reference]({{ site.baseurl }}/pdf/r-markdown-reference.pdf){:target="blank"}: overview of markdown syntx, knitr chunk options, and Pandoc options
- [R Markdown: The Definitive Guide](https://bookdown.org/yihui/rmarkdown/){:target="blank"}: a comprehensive overview of R Markdown
- [Git and Github chapter of R Packages](http://r-pkgs.had.co.nz/git.html): learn about git with R

### YAML Header

{% highlight x %}
---
title: "Title"
author: "Anthony Pan"
date: "2020-03-01"
output: 
  html_document: 
    toc: TRUE
    toc_float: TRUE
    code_folding: "hide"
---
{% endhighlight %}

* `toc: TRUE` adds a table of contents.
* `toc_float: TRUE` makes the table of contents float as you scross through the document.
* `code_folding` makes code chunks hidden by default but visible with a click.
* For a list of options, see `?rmarkdown::html_document`.
* Set `parameters` to set values when rendering the report; useful with `pwalk()` to re-render with different variables.
* Set [`bibliography`](https://rmarkdown.rstudio.com/authoring_bibliographies_and_citations.html){:target="blank"} to a bibliography file to automatically generate citations.

### Formatting code chunks

* `eval = FALSE` prevents code from being evaluated.
* `include = FALSE` runs the code, but doesn't show the code or results in the final document.
* `echo = FALSE` prevents code, but not results from appearing in the finished file.
* `message = FALSE` or `warning = FALSE` prevents messages or warnings from appearing in the finished file.
* `results = 'hide'` hides printed output, and `fig.show = 'hide'` hides plots.
* `fig.show = 'hold'` holds multiple plots from a code chunk, and `out.width = '50%'` specifies the width.
* `error = TRUE` causes the render to continue even if code returns an error (for debugging).

**Tip:** To print fancier-looking tables, surround your code with `knitr::kable(..., caption = "Table Title")` or use other packages such as `xtable, stargazer, apnder, tables, ascii`.

**Tip:** Cache code chunks for better performance with the option `cache = TRUE`. Use the `dependson` option if the chunk needs to be re-run if a dependency is changed, specifying a character vector of every chunk that the cached cunk depends on. Clean the cache with `clean_cache()`

**Tip:** Change the code chunk defaults at the beginning of the document -- for example, if you are presenting a doc for decision makers, you can turn off the default display of code globally:

{% highlight R %}
```{r example_setup_chunk, include = FALSE}
knitr::opts_chunk$set(
  echo = FALSE
)
```
{% endhighlight %}


### Troubleshooting R Markdown

1. Restart R, then run all chunks *( Ctrl + Alt + R )*
2. Check the working directory with `r getwd()` in a chunk.
3. Set `r error = TRUE` on the chunk causing the problem, then use `r print()` and `r str()` to check settings.

### Misc Info - R Markdown Formats

The output format can be set permanently by modifying the YAML header or set once by calling `rmarkdown::render()`.

* Basic document types: `pdf_document`, `word_document`, `github_document`, and `html_notebook`.
* Use `html_notebook` to collaborate; they contain full source code, are viewable in a browser and editable in RStudio.
* Use [`beamer_presentation`](https://bookdown.org/yihui/rmarkdown/beamer-presentation.html){:target="blank"} to create a PDF presentation with LaTeX Beamer
* Use [`flexdashboard::flex_dashboard`](https://rmarkdown.rstudio.com/flexdashboard/){:target="blank"} for dashboards.
* To produce interactivity, [`htmlwidgets`](http://www.htmlwidgets.org/){:target="blank"} produces interactive HTML visualizations in markdowon on the client-side, while [`shiny`](https://shiny.rstudio.com/){:target="blank"} creates interactivity using R code, but needs a Shiny server to be run online.
* Other formats include [`bookdown`](https://bookdown.org/yihui/bookdown/){:target="blank"} to write books, or [`prettydoc`](https://github.com/yixuan/prettydoc/){:target="blank"} to create nice-looking documents.

{% highlight r %}
# setup for collaboration
output:
  html_notebook: default   # gives a local preview and file that you can share via email
  github_document: default # creates a markdown file to use with git
{% endhighlight %}
