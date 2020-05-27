---
layout: post
title: "Introduction and Overview of Content"
description: ""
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents

- [I. Introduction to Modeling](#i-introduction-to-modeling)
- [II. Overview of Topics](#ii-overview-of-topics)

</div>

### I. Introduction to Modeling

The foundation of an effective predictive model is laid with intuition and knowledge of the problem context, relevant data, and computational toolbox with techniques for data pre-processing, visualization, and modeling. There is often a tradeoff between predictive accuracy and interpretability.

Why predictive models fail:
1. Inadequate pre-processing of data
2. Inadequate model validation
3. Unjustified extrapolation
4. Overfitting model to existing data

It is important to understand the characteristics of the data set in order to properly construct a model:
 
- Recognizing the **distribution of the response variable** is necessary for splitting data into training and testing sets
  - For continuous response data, is the distribution of the response symmetric or skewed?
  - For categorical response data, is the distribution balanced or unbalanced?
- Recognizing the **characteristics of the predictors** is necessary for data pre-processing and model selection
  - Missing values?
  - Different scales of measurements?
  - High correlation with other predictors?
  - Are predictors sparse (only a few contain unique information)?
  - Do they follow symmetric/skewed, or balanced/unbalanced distribution?
  - Do they even have an underlying relationship with the response?
- The **relationship between the number of samples and predictors** is important for model selection
  - Take computational time into account
  - Dimension reduction techniques can be useful
  - Techniques such as recursive partitioning and K-nearest-neighbors can be used


---
---
---
### II. Overview of Topics

###### **General Strategies**
- Data Pre-processing
- Resampling
- Overfitting and Model Tuning

###### **Regression**
- Linear Regression
- Partial Leaste Squares
- L1 regularization (Lasso)
- Neural Networks
- Multivariate adaptive regression splines (MARS)
- Support vector machines (SVMs)
- KNNs
- Tree-based models
  - Regression trees
  - Bagged trees
  - Random forests
  - Boosting
  - Cubist

###### **Classification**
- Discriminant analysis (linear, quadratic, regualrized, partial least squares)
- Penalized methods for classification
- Flexible discriminant analysis
- Neural networks
- SVMs
- KNNs
- Naive Bayes
- Nearest Shrunken Centroids
- Tree-based models

###### **Other Considerations**
- Measuring predictor importance
- Feature selection techniques
- Factors that can affect model performance

