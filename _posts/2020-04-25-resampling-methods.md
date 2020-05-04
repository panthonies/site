---
layout: post
title: "Resampling Methods"
description: "Resampling methods provide ways tools for model assessment (evaulate a model's performance) and model selection (optimally adjust model flexibility)."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents 

- [I. Cross-Validation](#i-cross-validation)
- [II. Bootstrap](#ii-bootstrap)
- [III. Applications in R](#iii-applications-in-r)

</div>


### I. Cross Validation

Cross validation involves estimating the test error rate by holding out a subset of the training data. There are three main types of cross validation: using a validation set, leave-one-out cross validation, and k-fold cross validation.

<span class="boxheader">Validation Set</span>

In this simple method, we split the observations into a training and validation set. We build the model based on the training set, and tune its parameters based on their performance in the validation set. 

There are two major drawbacks in this method:
1. Validation estimates of the test error rate are highly variable depending on the choice of validation set
2. only a subset of training data is used to build the model which makes it weaker than it could be, so the validation set often overestimates the test error

<span class="boxheader">Leave-one-out cross validation</span>

Leave-one-out cross validation (LOOCV) uses a single observation for the validation set, fits a model to the rest of the data, then predicts the single validation observation. It then repeats this process for every observation to calculate $$\text{MSE}_1, ... , \text{MSE}_n$$. The average of all of these errors is the LOOCV error: 

$$ CV_{(n)} = \frac{1}{n} \sum_{i = 1}^n \text{MSE}_i $$

LOOCV can be used for all kinds of predictive modeling, has very little bias and no randomness involved, but it can be resource intensive.

For least squares linear or polynomial regression, there is an easy shortcut to solve for the LOOCV error for computational efficiency (same cost as a single model fit):

$$ 
CV_{(n)} = \frac{1}{n} \sum_{i = 1}^n \biggl(\frac {y_i - \hat y_i} {1 - h_i} \biggr)^2 , \\
\text{ where } h_i \text{ is the leverage statistic for observation } i. 
$$


<span class="boxheader">K-fold cross validation</span>

K-Fold cross validation randomly divides the observation into K folds of approximately equal size, has each one serve as the validation set for the rest. The test errors are then averaged to calculate the K-fold cross validation error. 

$$ CV_{(k)} = \frac{1}{K} \sum_{i = 1}^K \text{MSE}_i $$

This is more computationally feasible than LOOCV, and has higher bias and lower variance. This is because the model outputs of LOOCV are highly correlated with each other, while those of K-fold CV are somewhat less correlated with each other, and the mean of highly correlated quantities has higher variance. K-fold cross validation often actually outperforms LOOCV due to the bias/variance tradeoff. 

Note: Use K = 5 or K = 10 in practice for the best results.

<span class="boxheader">Classification vs. Regression</span>

The previous formulas assume that we're dealing with regression and use mean squared error as the measurement of performance. For classifcation, we can use $$Err_i = I(y_i \neq \hat y_i)$$ to represent mischaracterized observations instead of MSE.

For example, LOOCV for logistic regression looks like this:

$$ CV_{(n)} = \frac{1}{n} \sum_{i = 1}^n Err_i $$


---
---
---
### II. Bootstrap

Bootstrap allows us to quantify the uncertainty associated with an estimator or model. It is particularly useful when we can't calculate the standard error of an estimator (i.e. median), or when we can't assume anything about the population distribution (i.e. normality).

- Sample with replacement a large number of times, $$B$$ on a data set $$Z$$ to obtain $$Z^{*1}, Z^{*2}, ... , Z^{*B}$$.
- For each sample, calculate the bootstrap estimates $$ \hat \alpha^{*1}, \hat \alpha^{*2}, ... , \hat \alpha^{*B},$$
- Then calculate the standard error:

$$ SE_B (\hat \alpha) = \sqrt {\frac{1}{B - 1} \sum_{r = 1}^B \biggl( \hat \alpha ^ {*r} - \frac{1}{B} \sum_{r' = 1}^B \hat \alpha ^ {*r'} \biggr) ^ 2} $$

When choosing between models, the "one standard error rule" is used, and we choose the most sparse model who error is no more than one standard error above that of the best performing model.

Note that the probability that a bootstrap sample of size $$n$$ contains the $$j$$th observation is $$p = 1 - (1 - \frac{1}{n})^n$$. Since $$\lim \limits_{n \to \infty} (1 + \frac{x}{n})^n = e^x$$,  $$p$$ converges to $$ 1 - \frac{1}{e} = .632$$ as $$n \to \infty$$

---
---
---
### III. Applications in R

{% highlight R %}

library("ISLR") # sample data set
library("boot") # bootstrap function
library("tidyverse")
{% endhighlight R %}

{% highlight R %}

### Validation set
set.seed(1)
train <- sample(392, 196)

### Leave one out cross validation
model <- glm(mpg ~ horsepower, data = Auto)
loocv_error <- cv.glm(Auto, model)
loocv_error$delta # loocv error

# loocv for five competing models
loocv_error <- rep(0,5)
for (i in 1:5) {
  model <- glm(mpg ~ poly(horsepower, i), data = Auto)
  loocv_error[i] <- cv.glm(Auto, model)$delta[1]
}
loocv_error # 2nd order polynomial is best

### K-fold cross-valudation
kfoldcv_error <- rep(0, 10)
for (i in 1:10) {
  model <- glm(mpg ~ poly(horsepower, i), data = Auto)
  kfoldcv_error[i] <- cv.glm(Auto, model, K = 10)$delta[1]
}
kfoldcv_error

## Bootstrap
bootstrap_stat <- function(data, index) {
  return(coef(lm(mpg ~ horsepower, data = data, subset = index)))
}
set.seed(1)
boot(Auto, bootstrap_stat, 1000)

### Additional exercise: Validation set / bootstrap with "default" data

default <- Default

# validation set
train <- sample(dim(default)[1], dim(default)[1] / 2) # list of indexes for training set
model_logit <- glm(default ~ income + balance, data = default, family = "binomial", subset = train) # fit logistic regression
summary(model_logit) 
  
probs <- predict(model_logit, newdata = default[-train, ], type = "response") # probs that test set is "Yes" based on training model
pred_logit <- rep("No", length(probs)) 
pred_logit[probs > 0.5] <- "Yes" # classify to default category if posterior probability is > 0.5

mean(pred_logit != default[-train, ]$default) # calculate test error rate for prediction results to actual results

# bootstrap
bootstrap_stat <- function(data, index) {
  return(coef(glm(default ~ income + balance, data = data, family = "binomial", subset = index)))
}
bootstrap_est <- boot(Default, bootstrap_stat, 1000)  

{% endhighlight R %}
