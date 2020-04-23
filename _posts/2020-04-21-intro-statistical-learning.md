---
layout: post
title: "An Introduction to Statistical Learning and Methods"
description: ""
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents 

- [I. Motivation](#i-motivation)
- [II. Methods](#ii-methods)

</div>

The purpose of this series of posts is to create a concise overview of important modelling concepts, and the intended audience is someone who has learned these concepts before, but would like a refresher on the most important bits (e.g. myself).

### I. Motivation

Statistical learning involves building models to understand data.

<span class="boxheader">Why estimate the function, *f*, that connects the input and output?</span>
1. **Prediction:**
	- estimate output with a black box function to minimize the reducible error
2. **Inference:**
	- which predictors are associated with the response?
	- what is the relationship between te response and each predictor?
	- can the relationship between Y and each predictor be adequately summaried using linear equation?

<span class="boxheader">How do we estimate *f*?</span>
1. **Parametric Methods** make an assumption about functional form (i.e. linear), then use training data to fit or train the model.
	- Pro: Simplifies the problem down to estimating a set of parameters, and results are easily interpretable
	- Con: The chosen model will likely not match the unknown form of *f*
2. **Non-parametric Methods** do not make explicit assumptions about the functional form of *f*.
	- Pro: Potential to accurately fit a wider range of possible shapes
	- Con: Need a very large number of observations to obtain an accurate estimate.

<span class="boxheader">What are the two types of statistical learning?</span>
1. **Supervised learning** has predictor measurements and associated response measurements.
2. **Unsupervised learning** as observed measurements, but no associated response; often seek to understand relationships between variables or between observations.
	- Note: semi-supervised learning is when there are a limited number of response observations, and these methods are outside the scope of this book)

<span class="boxheader">How do we assess model accuracy?</span>

- **Bias-Variance Tradeoff** 
	- The goal is to develop a model that balances inflexible methods with large bias/small variance and flexible methods with small bias/large variance:

$$ E(y_0 - \hat{f}(x_0))^2 = Var(\hat{f}(x_0)) + [Bias (\hat{f}(x_0))]^2 + Var(\epsilon)$$

<br>

- **Regression: Mean squared error**

$$ \frac{1}{n} \sum_{i = 0}^n (y_i - \hat{f}(x_i))^2 $$

- **Classification: Error rate**

$$ \frac{1}{n} \sum_{i=1}^n I(y_i \neq \hat{y}_{i} ) $$

where $$I(y_i \neq \hat y_i)$$ is an indicator variable that equals 1 if the prediction is incorrect and 0 if correct.

- Notes:
	- The Bayes classifier on average minimizes the test error rate by assigning each observation to the most likely class, given its predictor values: $$ Pr(Y = j \vert X = x_0) $$. 
	- The Bayes decision boundary is the separating boundary between classes (note: K-nearest neighbors often gets very close to the optimal Bayes classifier).
	- The Bayes error rate is the lowest possible test error rate: $$ 1 - E( \max_{j} Pr(Y = j \vert X)) $$

<br>

---
### II. Methods

Details will be covered in later posts.

- **Regression:** Predicting or explaining a continuous (quantitative) output
	- Linear Regression with stepwise selection
	- Ridge Regression
	- Lasso
	- Principle Components Regression
	- Partial Least Squares
	- Non-Linear Additive Models

- **Classification:** Predicting or explaining a categorical (qualitative) output
	- Logistic Regression
	- Linear Discriminant Analysis
	- K-Nearest Neighbors
	- Support Vector Machines

- **Resampling Methods:** Techniques that produce more accurate models
	- Cross validation
	- Bootstrap

- **Tree-based Methods:** Stratifying or segmenting the predcitor space into regions
	- Bagging
	- Boosting
	- Random Forests

- **Clustering:** Grouping individuals according to observed characteristics
	- Principle Components Analysis
	- K-means Clustering
	- Hierarchical Clustering

