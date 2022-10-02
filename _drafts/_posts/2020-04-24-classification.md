---
layout: post
title: "Classification"
description: "Classification models are used to predict a categorical response. "
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents 

- [I. K-Nearest Neighbors](#i-k-nearest-neighbors)
- [II. Logistic Regression](#ii-logistic-regression)
- [III. Linear Discriminant Analysis](#iii-linear-discriminant-analysis)
- [IV. Quadratic Discriminant Analysis](#iv-quadratic-discriminant-analysis)
- [V. Comparison of Classification Methods](#v-comparison-of-classification-methods)
- [VI. Applications in R](#vi-applications-in-r)

</div>


### I. K-Nearest Neighbors

Given a positive integer $$K$$ and a test observation $$x_0$$, the K-nearest neighbors (KNN) classifier identifies the $$K$$ points in the training data closest to $$x_0$$, represented by $$\eta_0$$, and then estimates the conditional probability for class $$j$$ as the fraction of points in $$\eta_0$$ whose response values equal $$j$$:

$$ Pr(Y = j \vert X = x_i) = \frac{1}{K} \sum_{i \in \eta_0} I(y_i = j) $$

KNN has high bias and low variance when $$K$$ is large, and vice versa when $$K$$ is small. While the training error rate will always decrease as $$K$$ decreases, the test error will take on a characteristic U-shape due to overfitting.

---
---
---
### II. Logistic Regression

<span class="boxheader">Logistic Regression with One Predictor</span> 

Logistic regression models the probability that $$Y$$ belongs to a particular category.

$$ p(X) = Pr(Y = 1 \vert X) = \frac {e^{\beta_0 + \beta_1 X}} {1 + e^{\beta_0 + \beta_1 X}} $$

We can re-write this equation to represent the odds $$\Bigl[ \frac {p} {1 - p} \Bigr]$$, and log-odds/logit $$ \Bigl[ \log \bigl( \frac{p(X)} {1 - p(X)} \bigr) \Bigr]$$ of $$(Y=1 \vert X)$$:

$$ \begin{align}
\frac{p(X)} {1 - p(X)} & = e^{\beta_0 + \beta_1 X} \\
\Rightarrow \log \Biggl( \frac{p(X)} {1 - p(X)} \Biggr) & = \beta_0 + \beta_1 X 
\end{align} $$

* Note that because logistic regression takes an $$S$$ shape, the effect of one-unit increase in $$X$$ on $$p(X)$$ depends on the current value of $$X$$.

We estimate the coefficients $$\beta_0$$ and $$\beta_1$$ using estimates that maximize the likehood that the logistic model produced the observed data; in other words, plugging the coefficients into the logistic model should result in a number close to 1 for all observations with $$Y=1$$, and close to 0 for all observations with $$Y=0$$. This is accomplished by maximizing the likelihood function, $$l(\beta_0, \beta_1)$$:

$$ l(\beta_0, \beta_1) = \Pi_{i : y_i = 1} p(x_i) \Pi_{i' : y'_i = 1} (1 - p(x_i))$$

We can conduct a hypothesis test to test if the probability of $$Y$$ does not depend on $$X$$ with null hypothesis $$ H_0: \beta_1 = 0 $$, by using a $$z$$-statistic, $$ \hat \beta_1 / SE(\hat \beta_1) $$

<span class="boxheader">Multiple Logistic Regression</span> 

We can extend the basic logistic regession model to include multiple predictors:

$$ p(X) = \frac {e^{\beta_0 + \beta_1 X + \beta_2 X_2 + ... + \beta_p X_p}} {1 + e^{\beta_0 + \beta_1 X + \beta_2 X_2 + ... + \beta_p X_p}} $$

$$ log \Biggl( \frac{p(X)} {1 - p(X)} \Biggr) = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + ... + \beta_p X_p $$


Be careful of confounding variables! The results obtained using one predictor may be different than those obtained with multiple predictors, especially if predictors are correlated with each other.

* Note: Logistic regression is mostly used when the response variable has two classes. It is possible to use logistic regression for a response variable with more than two classes; however, linear discriminant analysis is much more often used in that scenario.

---
---
---
### III. Linear Discriminant Analysis

Linear discriminant analysis (LDA) focuses on maximizing the separability among categories by modeling the distributions of the predictors $$X$$ separately in each response class, $$Pr(X \vert Y)$$, then using Bayes' theorem to flip them into estimates of $$Pr(Y = k \vert X = x)$$.  

<span class="boxheader">Motivation</span> 

Why use LDA instead of logistic regression?
1. When the classes are well-separated, the parameter estimates for LDA are more stable than for logistic regression.
2. If $$n$$ is small and the distribution of predictors $$X$$ is approximately normal in each class, then LDA is more stable than logistic regression.

Let us define the following conditions:
- $$\pi_k$$ is the prior probability that a random observation comes from the $$k$$th class
- $$f_k(x)$$ is the density function of $$X$$ for an observation from the $$k$$th class
- $$p_k(x)$$ is the posterior probability that an observation $$X=x$$ belongs to the $$k$$th class

Then Bayes' theorem states that:

$$ p_k(X) = Pr(Y = k \vert X = x) = \frac{\pi_k f_k(x)} {\sum_{l = 1}^K \pi_l f_l(x)} $$

The goal is to estimate $$f_k (x)$$ to develop a classifier that estimates the Bayes classifier.

<span class="boxheader">LDA for one predictor</span> 

Assumptions:
1. $$f_k(x)$$ is normal or gaussian, and its density takes the form $$ \frac{1} {\sqrt {2\pi} \sigma_k} \exp \Bigl( \frac{-1}{2 \sigma_k^2} (x -\mu_k)^2 \Bigr) $$
2. Variance is constant within each class; $$ \sigma_1^2 = ... = \sigma_k^2 $$

We assign each observation to the class that maximizes the discriminant function, which we obtain by plugging the $$f_k(x)$$ into Bayes' theorem and simplifying:

$$ \delta_k(x) = x \frac{\mu_k}{\sigma^2} - \frac{\mu_k^2}{2 \sigma^2} + \log (\pi_k) $$

The parameters $$\mu_k, \pi_k,$$ and $$\sigma_k$$ are estimated as follows:

$$\begin{align}
\hat \mu_k & = \frac{1} {n_k} \sum_{i: y_i = k} x_i \\
\hat \sigma_k^2 & = \frac{1}{n-k} \sum_{k = 1}^K \sum_{i: y_k = k} (x_i - \hat \mu_k)^2 \\
\hat \pi_k & = n_k / n
\end{align}$$

Thus, the LDA classifier results are calculated by plugging estimates for each parameter with the observation value into the Bayes classifier.

- Note that for the two-class scenario with $$\pi_1 = \pi_2$$, then the Bayes decision boundary correponds to the point where $$ x = \frac{u_1 + u_2}{2} $$.


<span class="boxheader">LDA for multiple predictors</span> 

Assumption:
1. $$ X = (X_1 ... X_p) $$ is drawn from a multivariate gaussian distribution with a class-specific mean vector and common covariance matrix, $$ X \sim N(\mu, \Sigma) $$ with density:

$$ f(x) = \frac{1}{(2 \pi)^{p / 2} \vert \Sigma \vert ^{1/2}} \exp { \Biggl( -\frac{1}{2}(x - \mu)^T \Sigma^{-1} (x - \mu) \Biggr) } $$

If observations in the $$k$$th class are drawn from $$N(\mu_k, \Sigma)$$, then we can plug in the density function to find that the Bayes classifier assigns an observatino $$X = x$$ to maximize the discriminant function:

$$ \delta_k (x) = x^T \Sigma^{-1} \mu_k - \frac{1}{2} \mu_k^T \Sigma^{-1} \mu_k + \log \pi_k $$

The unknown parameters $$\mu_k, \pi_k, and \sigma_k$$ are estimated with formulas similar to those in the one-dimensional case. To assign a new observation, LDA plugs the parameter estimates into the discriminant function and classifies it to the class for which $$\hat \delta_k(x)$$ is largest. 

<span class="boxheader">Considerations when using LDA</span> 

1. The higher the ratio of parameters $$p$$ to sample size $$n$$, the more likely LDA will overfit the data.
2. LDA has low sensitivity (power), because it is based off of the Bayes classifier which **minimizes total error** by assigning observations to categories if $$P(\text{Category} = \text{Yes} \vert X = x) > .5 $$

A confusion matrix is a table that displays predicted vs. actual values, and they can be used to calculate the following important measure for classification and diagnostic testing:
- **Sensitivity** (aka power) = $$P(\text{True Positive})$$
- **Specificity** = $$P(\text{True Negative}) $$
- **Type I Error** is the false positive rate, also 1 - Specificity 
- **Type II Error** is the false negative rate, also 1 - Sensitivity
- Note: check out [Wikipedia](https://en.wikipedia.org/wiki/Confusion_matrix) for a nice visualization of a 2x2 confusion matrix and all of its related statistics.

We can address problem #2 by **setting a new threshold** for assigning an observation to a certain class, based on domain knowledge. Setting a new threshold can be visualized with an **ROC curve** (receiver operating characteristics). An example is shown below.[^1]

<center>{% include image.html path="documentation/04-24-roc.png"
                      path-detail="documentation/04-24-roc.png"
                      alt="ROC" %}</center>

[^1]: Image source: James et. al, *Introduction to Statistical Learning, 7th Ed.*, pg. 148

The ROC curve graphs False Positive Rate (Type I Error) against the True Positive Rate (Power/Sensitivity). Ideally, it should hug the top left corner of the plot. Increasing the threshold will move results to the lower left along the ROC curve, and decreasing the threshold will move results to the upper right. 


**AUC** is the area under the ROC curve, and it tells us how much the model is able to distinguish between classes. The higher the AUC value, the better the model, with a max value = 1.
- High AUC: model classifies observations well
- Close to .5: model has no separation ability
- Low AUC: model classifies the opposite way

---
---
---
### IV. Quadratic Discriminant Analysis

Quadratic distriminant analysis (QDA) assumes that each class has its own covariance matrix, so an observation in the $$k$$th class is given by $$ X \sim N(\mu_k, \Sigma_k) $$.

We assign an observation $$X = x$$ to the class where $$\delta_k(x)$$ is largest. Note that $$\delta_k(x)$$ is a quadratic function of $$x$$, as opposed to a linear function.

$$ \delta_k (x) = - \frac{1}{2} x^T \Sigma^{-1}_k x + x^T \Sigma^{-1} \mu_k - \frac{1}{2} \mu_k^T \Sigma^{-1} \mu_k - \frac{1}{2} \log \vert \Sigma_k \vert + \log \pi_k $$

Comparing LDA and QDA:
- when there are p predictors, assuming a separate covariance matrices with QDA means estimating $$K * \frac{p(p+1)}{2}$$ parameters. However, LDA only requires estimates $$Kp$$ linear coefficients by assuming a common covariance matrix.
    - LDA has higher bias and lower variance
    - QDA has lower bias and higher variance
- LDA: useful when there are fewer training observations, when reducing variance is important
- QDA: useful when the training set is ver large, or if you can't assume a common covariance matrix

---
---
---
### V. Comparison of Classification Methods

Logistic regression and LDA produce linear decision boundaries. While logistic regression estimates parameters with maximum likelihood, LDA estimates parameters with the estimated $$\mu$$ and $$\sigma^2$$ from a normal distribution. Therefore, we should use LDA when we can assume that observations are drawn from a gaussian distribution with common covariance matrix, and use logistic regression when those assumptions are not met.

QDA can model a wider range of data than logistic regression and LDA, but it is not as flexible as KNN. However, QDA performs better than KNN with limited training observations.

KNN dominates when the true decision boundary is non-linear, but it suffers a major drawback that it doesn't show which predictors are important in obtaining the result.

<span class="boxheader">Summary of Each Method</span> 

**Logistic Regression**
- Linear decision boundary
- Provides interpretability with odds ratios
- High bias, low variance

**Linear Discriminant Analysis**
- Linear decision boundary
- Provides interpretability with predictor ability to separate variation
- Assumes all observations are drawn from normal distributions
- Assumes observations in all classes share a covariance matrix
- Stable when classes are "well-separated"
- Stable when sample size is small
- Commonly used over logistic regression when response is >2 classes
- Can adjust assignment-probability threshold for a better specificity rate with the ROC curve
- High bias, low variance

**Quadratic Discriminant Analysis**
- Quadratic decision boundary
- Assumes observations are drawn from a multivariate normal distribution
- Assumes different covariance matrices for each class
- Performs well with many training observations compared to LDA
- Medium bias, medium variance

**K-Nearest Neighbors**
- Non-parametric decision boundary
- No interpretability
- Requires smart selection of the smoothness $$K$$
- Bad when the number of predictors is large, due to the curse of dimensionality
- Low bias, high variance


---
---
---
### VI. Applications in R

{% highlight R %}
library("MASS")     # glm + boston dataset
library("class")    # knn 
library("caret")    # ML
library("tidyverse")
{% endhighlight R %}

{% highlight R %}

# Boston data: predict whether a given suburb has a crime rate above or below the median
# 1. transform data
boston_data <- Boston %>%
  mutate(crime_above_median = (crim > median(crim)),
         train = (1:nrow(Boston) %% 4 != 0)) # 1/4 testing, 3/4 training

# 2. explore data
cor(boston_data) # seems like rad, tax, dis, age, indus may play a role

# 3. split train and test
train <- as_tibble(subset(boston_data, train))
test  <- as_tibble(subset(boston_data, !train))

###
### logistic regression ------------------------------
logit_model <- glm(crime_above_median ~ rad + tax + dis + age + indus, 
                          family = binomial, 
                          train)

# predictions
logit_probs_train <- predict(logit_model, train, type = "response") 
logit_pred_train  <- rep("FALSE", length(train$crime_above_median))
logit_pred_train[logit_probs_train > .5] <- "TRUE"

logit_probs_test <- predict(logit_model, test, type = "response") 
logit_pred_test <- rep("FALSE", length(test$crime_above_median))
logit_pred_test[logit_probs_test > .5] <- "TRUE"

# confusion matrices; train and test
(matrix_logit <- confusionMatrix(table(logit_pred_train, train$crime_above_median))) 
(matrix_logit <- confusionMatrix(table(logit_pred_test, test$crime_above_median)))

###
### linear discrimination analysis ------------------------------
lda_model <- lda(crime_above_median ~ rad + tax + dis + age + indus, train)

# predictions
lda_pred_train <- predict(lda_model, train)
lda_pred_test <- predict(lda_model, test)

# confusion matrices; train and test
(matrix_lda <- confusionMatrix(table(lda_pred_train$class, train$crime_above_median)))
(matrix_lda <- confusionMatrix(table(lda_pred_test$class, test$crime_above_median))) 

###
### quadratic discrimination analysis ------------------------------
qda_model <- qda(crime_above_median ~ rad + tax + dis + age + indus, train)

# predictions
qda_pred_train <- predict(qda_model, train)
qda_pred_test <- predict(qda_model, test)

# confusion matrices; train and test
(matrix_qda <- confusionMatrix(table(qda_pred_train$class, train$crime_above_median))) 
(matrix_qda <- confusionMatrix(table(qda_pred_test$class, test$crime_above_median)))

###
### k-nearest neighbors ------------------------------
knn_train <- scale(select(train, rad, tax, dis, age, indus))
knn_test  <- scale(select(test, rad, tax, dis, age, indus))
knn_train_class <- train$crime_above_median

set.seed(1)
knn_pred <- knn(knn_train, knn_test, knn_train_class, k = 1)

# confusion matrix
(matrix_knn <- confusionMatrix(table(knn_pred, test$crime_above_median)))
{% endhighlight R %}
