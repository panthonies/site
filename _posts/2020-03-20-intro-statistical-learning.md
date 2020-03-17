---
layout: post
title: "An Introductory Overview of Statistical Learning and Methods"
description: ""
thumb_image: 
tags: [academic]
---

- **Regression:** Predicting or explaining a continuous (quantitative) output [wage data]
	- Linear Regression
	- Stepwise Selection
	- Ridge Regression
	- Principle Components Regression
	- Partial Least quares
	- Lasso
	- Non-Linear Additive Models

- **Classification:** Predicting or explaining a categorical (qualitative) output [stock market data]
	- Logistic Regression
	- Linear Discriminant Analysis
	- K-Nearest Neighbors
	- Support Vector Machines

- **Clustering:** Group individuals according to observed characteristics [gene data]
	- Principle Components Analysis
	- K-means clustering
	- Hierarchical Clustering

- **Tree-based Methods:**
	- Bagging
	- Boosting
	- Random Forests

- **Resampling Methods:**
	- Cross valiation
	- Bootstrap

### Statistical Learning

Why estimate *f* that connects the input and output?
1. Prediction:
	- estimate output with a black box function to minimize the reducible error
2. Inference: 
	- which predictors are associated with the response?
	- what is the relationship between te response and each predictor?
	- can the relationship between Y and each predictor be adequately summaried using linear equation?

How do we estimate *f*?
1. Parametric Methods make an assumption about functional form (i.e. linear model), then use training data to fit or train the model.
	- Pro: Simplifies the problem down to estimating a set of parameters, and results are easily interpretable
	- Con: THe chosen model will likely not match the unknown form of *f*
2. Non-parametric Methods do not make explicit assumptions about the functional form of *f*.
	- Pro: Potential to accurately fit a wider range of possible shapes
	- Con: Need a very large number of observations to obtain an accurate estimate.

What are the two types of statistical learning?
1. Supervised learning has predictor measurements and associated response measurements.
2. Unsupervised learning as observed measurements, but no associated response; often seek to understand relationships between variables or between observations.
	- Note: semi-supervised learning is when there are a limited number of response observations, and these methods are outside the scope of this book)

Assessing Model Accuracy
- Bias-Variance Tradeoff (find a balance between inflexible methods with large bias/small variance and flexible methods with small bias/large variance)
- Regression
	- Mean squared error (find a balance between overly simple models and overfitting models)
- Classification
	- Error rate 
	- Bayes classifier, determined by the Bayes decision boundary, produces the Bayes error rate
	- K-Nearest Neighbors: will classify a point based on the values of the K closest neighbors; this often gets very close to the optimal Bayes classifier. A lower value of K corresponds to less flexibility in Bayes decision boundary.

