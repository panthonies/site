---
layout: post
title: "Linear Model Selection and Regularization"
description: "An alternative fitting method to least squares, such as subset selection, shrinkage (ridge regression, lasso), and dimension reduction techniques (principle components analysis, partial least squares) can help prediction accuracy and model interpretability."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents 

  <ul>
    <li><a href = "#i-overview">I. Overview</a></li>
    <li><a href = "#ii-subset-selection">II. Subset Selection</a>
      <ul class="hi"><li>Best subset selection</li>
      <li>Forward stepwise selection</li>
      <li>Backward stepwise selection</li>
      <li>Statistics for model selection</li></ul></li>
    <li><a href = "#iii-shrinkage">III. Shrinkage</a>
      <ul class="hi"><li>Ridge Regression</li>
      <li>Lasso</li></ul></li>
    <li><a href = "#iv-dimension-reduction-techniques">IV. Dimension Reduction Techniques</a>
      <ul class="hi"><li>Principle components analysis</li>
      <li>Partial least squares</li></ul></li>
    <li><a href = "#v-applications-in-r">V. Applications in R</a></li>
  </ul>
</div>


### I. Overview

Why use an alternative fitting model to least squares?

- Prediction Accuracy
  - If $$n$$ is not much larger than $$p$$, then least squares can have a lot of variability
  - If $$p < n$$, then there is no longer a unique least squares coefficient estimate, and we must constrain or shrink the coefficients

- Model Interpretability
  - Remove irrelevant variables with feature/variable selection

<span class="boxheader">Alternatives to Least Squares</span>

1. Subset selection: select a subset of predictors before fitting least squares
2. Shrinkage: fit model with all predictors, with coefficients shrunk toward 0 relative to the least squares estimate
3. Dimension reduction: projecting predictors onto an $$M$$-dimensional sbspace where $$M < p$$ by computing projections/linear combinations of the variables.  These $$M$$ projections are then used as predictors for least squares.


---
---
---
### II. Subset Selection

<span class="boxheader">Best Subset Selection</span>

1. Let $$M_0$$ denote the null model.
2. For $$ k = 1, 2, ..., p$$, fit all models with $$k$$ predictors, and choose the best one (smallest RSS/largest $$R^2$$), $$M_k$$.
3. Select the best model from $$M_1, ..., M_k$$ using a model selection metric such as adjusted $$R^2$$, AIC, BIC, or cross-validated prediction error.

Note: this is very computationally expensive, and infeasible for $$p > 40$$, since there are $$2^p$$ total models to check. In addition this method has high variance which can lead to overfitting.


<span class="boxheader">Forward Stepwise Selection</span>

1. Let $$M_0$$ denote the null model.
2. For $$ k = 0, 2, ..., p - 1$$, consider all $$p - k$$ models that add one additional predictor to $$M_k$$, then choose the best one, $$M_{k+1}$$.
3. Select the best model from $$M_0, ..., M_p$$ using a model selection metric.

Note: this is much more computationally feasible than best subset selection with $$1 + \frac{p(p+1)}{2}$$ total models to check, but it's not guaranteed to yield the "best" model for the training data. When $$p>n$$, this is the only feasible subset selection method.

<span class="boxheader">Backward Stepwise Selection</span>

1. Let $$M_0$$ denote the null model.
2. For $$ k = p, p-1, ..., 1$$, consider all $$k$$ models that contain all but 1 of the predictors in $$M_k$$, a total of $$k-1$$ predictors, then choose the best one, $$M_{k+1}$$.
3. Select the best model from $$M_0, ..., M_p$$ using a model selection metric.

Note: Like forward selection, this only searches through $$1 + \frac{p(p+1)}{2}$$ models, and is not guaranteed to yield the best model. It cannot be used with $$p>n$$. Hybrid approaches (forward-backward) may be used as well.

<span class="boxheader">Model selection statistics</span>

We can choose the optimal model either by estimate the test error *indirectly* by adjusting the training error to account for overfitting bias, or *directly* by using a validation set or cross-validation.

**Adjusting the training error:**
- $$C_p = \frac{1}{n} (\text{RSS} + 2 d \hat \sigma^2) $$ ; 
  - Penalizes the model based on the number of predictors, $$d$$. Lower is better.
  - If $$\hat \sigma^2$$ is an unbiased estimator of $$\sigma^2$$, then $$C_p$$ is an unbiased estimator of the test MSE.

- $$\text{AIC} = \frac{1}{n \hat \sigma^2} (\text{RSS} + 2 d \hat \sigma^2) $$ ;
  - Use this for models fit by maximum likelihood.
  - With gaussian errors, maximum likelihood = least squares.

- $$\text{BIC} = \frac{1}{n \hat \sigma^2} (\text{RSS} + \log {n} d \hat \sigma^2) $$ ;
  - Use this for least squares model with $$d$$ predictors.
  - Has a heavier penalty for models with many variables.

- $$ \text{Adj. } R^2 = 1 - \frac{\text{RSS} / (n - d - 1)} {TSS / (n-1)} $$ ;
  - Larger is better.

Note: These formulas are all for linear least squares, and can be generalized for additional model types.
m
**Cross Validation or validation set error:**
- Makes fewer assumptions about the underlying model, and can be used when the number of predictors is unknown, or $$\hat \sigma^2$$ cannot be estimated.
- One Standard Error Rule: choose the smallest model for which the estimated test MSE is within one standard error of the lowest point on the curve.

---
---
---
### III. Shrinkage

Shrinking coefficient estimates can significantly reduce model variance.

<span class="boxheader">Ridge Regression</span>

While least squares minimizes RSS, ridge regression picks coefficient estimates $$\hat \beta^R$$ to minimize:

$$ \text{RSS} + \lambda \sum_{j = 1}^p \beta_j^2 $$

Where:
- $$\text{RSS} = \sum_{i = 1}^n \Bigl( _i - \beta_0 - \sum_{j = 1}^p \beta_j x_{ij} \Bigr) ^ 2 $$ ;
- $$\lambda \geq 0$$ is a tuning parameter
- $$\lambda \sum_j \beta_j^2$$ is the shrinkage penalty, and grows as $$\lambda \to \infty$$

It is best to perform ridge regression after standardizing the predictors (note the denomenator is the standard deviation of the $$j$$th predictor):

$$ \tilde x_{ij} = \frac{x_{ij}} {\sqrt {\frac {1} {n} \sum_{i = 1}^n (x_{ij} - \bar x_j)^2}} $$ 

**Improvements over least squares:**
- Bias-variance tradeoff: as $$\lambda$$ increases, variance decreases and bias increases.
- In cases where relationships between predictors and response is approximate linear, then least squares will have low bias but may have high variance, in particular when the number of variables is almost as big, or bigger, than the sample size.
- Ridge regression works best when the least squares estimate has high variance.
- Computationally superior to best subset selection, since solving for all $$\lambda$$ is almost computationally equivalent to fitting least squares. 

**Drawback:** 
- Ridge regression will include all $$p$$ predictors in the final model, which presents challenges for interpretability.


<span class="boxheader">Lasso</span>

The lasso coefficients $$\hat \beta_{\lambda}^L$$ minimize the equation:

$$ \text{RSS} + \lambda \sum_{j = 1}^p \vert \beta_j \vert $$

Where $$ \text{RSS } = \sum_{i = 1}^n \Bigl( _i - \beta_0 - \sum_{j = 1}^p \beta_j x_{ij} \Bigr) ^ 2$$ 

**Notes:**
- Bias-variance tradeoff: as $$\lambda$$ increases, variance decreases and bias increases.
- Lasso uses an $$l_1$$ penalty instead of  the $$l_2$$ penalty of ridge regression.
- The $$l_1$$ norm of coefficient vector $$\beta$$ is $$\lVert \beta \rVert _1 = \sum \vert \beta_j \vert $$.
- Performs variable selection by forcing some coefficients to 0, and yields sparse models.

<span class="boxheader">Comparison of Ridge Regression and Lasso</span>

**Ridge Regression:**
- $$ \min_{\beta} \text{\{RSS\}} \text{ subject to } \sum_{j = 1}^p \beta_j^2 \leq s $$ ;
- uses $$l_2$$ penalty
- computationally feasible alternative to best subset selection
- outperforms lasso when all variables are related to the response, or when there are many predictors of relatively equal size.

**Lasso:**
- $$ \min_{\beta} \text{\{RSS\}} \text{ subject to } \sum_{j = 1}^p \vert \beta_j \vert \leq s $$ ;
- uses $$l_1$$ penalty
- computationally feasible alternative to best subset selection
- outperforms ridge when only a few predictors have substantial coefficients, and the rest are very small or 0. It also has advantages in that it performs variable selection and is easier to interpret.

The tuning parameter $$\lambda$$ should be selected with cross validation by selecting a grid of $$\lambda$$ values, computing the cross-validation error for each $$\lambda$$, and selecting $$\lambda$$ for which cv error is smallest.

The classic image below[^1]  illustrates the difference between ridge regression and lasso in two dimensions. Points along each red ellipse have equal RSS, and the blue areas represent the $$l_1$$ and $$l_2$$ constraints. The ridge and lasso solutions are where the red ellipses touch the blue areas.

<center>{% include image.html path="documentation/04-27-lasso-ridge.png"
                      path-detail="documentation/04-27-lasso-ridge.png"
                      alt="Lasso vs Ridge" %}</center>


Further intuition for a simple case is given below[^2]  -- we see that ridge regression shrinks every dimension of data by the same proportion, while lasso shrinks all coefficients toward 0 by a similar amount, and sufficiently small coefficients are shrunk to 0. 


<center>{% include image.html path="documentation/04-27-lasso-ridge-2.png"
                      path-detail="documentation/04-27-lasso-ridge-2.png"
                      alt="Lasso vs Ridge 2" %}</center>



<span class="boxheader">Extra Theory: a bayesian interpretation for ridge regression and lasso</span>


Suppose that:
- a coefficient vector $$\beta$$ has a prior distribution $$p(\beta)$$, where $$\beta = (\beta_0, ..., \beta_p) $$
- the likelihood of the data can be written as $$f(Y \vert X, \beta)$$, where $$X = (X_1, ..., X_p)$$

Then, multipling the prior distribution by the likelihood gives the posterior distribution up to a proportionality constant:

$$P(\beta \vert X, Y) \propto f(Y \vert X, \beta) p(\beta \vert X) = f(Y \vert X, \beta) p(\beta)$$

Assume:
- $$ Y = \beta_0 + X_1 \beta_1 + ... + X_p \beta_p + \epsilon $$ ; where $$ \epsilon \sim \text{iid} N(\mu, \sigma) $$
- $$p(\beta) = \Pi_{j = 1}^P g(\beta_j)$$ for some density function $$g$$

Then,
- If $$g$$ is a **gaussian distribution** with mean 0 and standard deviation that is a function of $$\lambda$$, then the posterior mode (most likely value for $$\beta$$ given the data) for $$\beta$$ is given by the **ridge regression solution**.
- If $$g$$ is a **double-exponential (Laplace) distribution** with mean 0 and scale that is a function of $$\lambda$$, then the posterior mode for $$\beta$$ is given by the **lasso solution**.
  - However, note that the lasso solution is not the posterior mean, and the posterior mean doesn't yield a sparse coefficient vector.

A visualization of the gaussian and double-exponential distributions:[^3]

<center>{% include image.html path="documentation/04-27-lasso-ridge-3.png"
                      path-detail="documentation/04-27-lasso-ridge-3.png"
                      alt="Lasso vs Ridge 3" %}</center>



[^1]: Source: James et. al, *Intro to Statistical Learning 7th edition*, pg. 222
[^2]: Source: James et. al, *Intro to Statistical Learning 7th edition*, pg. 226
[^3]: Source: James et. al, *Intro to Statistical Learning 7th edition*, pg. 227


---
---
---
### IV. Dimension Reduction Techniques

These techniques transform the predictors, before using least squares to fit a model with the transformed predictors. When the number of predictors is large relative to the sample size, then significantly reducing the number of dimensions can significantly reduce variance. In more formal terms...

Let $$Z_1, ..., Z_m$$ represent $$M < p$$ linear combinations of the original $$p$$ predictors:

$$ Z_m = \sum_{j = 1}^p \phi{jm} X_j \text {  for some constants  } \theta_{1m}, ..., \theta_{pm}, m = 1, ..., M$$

Then using least squares, fit the linear regression model: $$ y_i = \theta_0 + \sum_{m = 1}^M \theta_m z_{im} + \epsilon_i$$
- The regression coefficients are $$\theta_0, ..., \theta_m$$
- Dimensions are reduced from $$p+1$$ to $$M+1$$
- Coefficients are restrained by $$\beta_j = \sum_{i = 1}^M \theta \phi_{jm}$$



<span class="boxheader">Principle Components Analysis</span>

The first principle components vector is chosen from all combinations of predictors such that:
- $$\phi_{11}^2 + ... + \phi_{pm}^2 = 1$$ ;
- captures the highest variance of predictors
- defines the line that is as close as possible to the data
- the first principle components score for the $$i$$th observation is the distance in the $$x$$-direction of the $$i$$th projection from 0, the mean point of all predictors.
- all principle components vectors are orthogonal to each other

The second principle component $$Z_2$$ is a linear combination of the variables that is uncorrelated with $$Z_1$$ and has the largest variance, which means it must be orthogonal to the first principal component direction. Other principle components can be calculated in the same way.

**Principle components regression**
- assume that the directions in which $$X_1, ..., X_p$$ show the most variation are the directions that are associated with $$Y$$
- using more principle components means lower bias, higher variance
- this is NOT a feature selection method, since it combines all $$p$$ original features
- closely related to ridge regression

Notes:
- choose the number of principle components $$M$$ using cross validation
- standardize each predictor before generating principle components, so that high variance variables don't dominate


<span class="boxheader">Partial Least Squares</span>

Partial Least Squares (PLS) is a supervised alternative to PCA that identifies new features $$Z_1, ..., Z_m$$ that are combinations of the original predictors and **related to the response**. The biggest drawback of PCA is that its components are not guaranteed to be the ones that best explain the response, since it's unsupervised.

- **compute the first PLS direction**, $$Z_1 = \sum_{j = 1}^p \phi_{j1} X_j$$, by setting each $$\phi_{j1}$$ to the simple linear regression coefficients of $$Y \sim X_j$$.
  - this places the highest weights on variables most strongly correlated with the response when calculating $$Z_1$$
  - does not fit the predictors as well as PCA, but does a better job explaining the response
- **compute the second PLS direction**, $$Z_2$$, by regressing each variable on $$Z_1$$ and taking the residuals, then compute $$Z_2$$ by using the orthogonalized data in the same way as $$Z_1$$.
- Choose the number of predictors $$M$$ with cross-validation.

This method is popular in chemometrics, but in practice often performs no better than PCR or ridge regression. Relative to PCR, PLS has lower bias and higher variance.

<span class="boxheader">Considerations in High Dimensions</span>

What goes wrong when $$p \geq n$$?
- be careful of overfitting -- always evaluate perfromance on an independent test set
- $$C_p$$, AIC, BIC cannot be used because estimating $$\hat \sigma^2$$ is problematic. Adjusted $$R^2$$ also can't be used.

Tips:
- forward stepwise selection, ridge regression, lasso, and PCR are useful
- regularization/shrinkage plays a big role
- tuning parameter selection is very important
- test error will increase as dimensions increase unless additional features are truly predictive (curse of dimensionality)

Interpreting results
- In high dimensions, any variable can be written as a linear combination of the others. For this reason, we can't know which variables are truly predictive of the outcome, or which coefficients are best.
- All models are only one of many possible models for prediction, and must be further validated on independent data sets.
- Never use the sum of squared errors, p-values, $$R^2$$, or others on training data as evidence of good fit. Instead, use either cross-validation results or test on an independent test set.

---
---
---
### V. Applications in R

{% highlight R %}

### exercise!!!! 
### using COLLEGE data - predict Apps (# applications)
library("ISLR")
library("pls")
library("glmnet")
library("purrr")
library("leaps")
library("tidyverse")
library("patchwork")
library("ggplot2")

{% endhighlight R %}

{% highlight R %}

# prepare data
college <- College
set.seed(1)
split <- sample(nrow(College), nrow(College) * .7)
train <- college[split,]
test <- college[-split,]
train_matrix <- model.matrix(Apps ~ ., data = train)
test_matrix  <- model.matrix(Apps ~ ., data = test)

mse <- function(pred, test) {
  return(mean((pred - test) ^ 2))
}

rsq <- function(pred, test) {
  return(1 - ((pred - test) ^ 2) / ((mean(pred) - test) ^ 2))
}

# least squares
model_lsq <- lm(Apps ~ ., data = train)
pred_lsq <- predict(model_lsq, test)

mse(pred_lsq, test$Apps) # MSE: 1261630
rsq(pred_lsq, test$Apps) # R-Squared: .9135

# least squares - best subset
model_lsq_best <- regsubsets(Apps ~ ., data = train, nvmax = 17)
model_lsq_best_summ <- summary(regsubsets(Apps ~ ., data = train, nvmax = 17))
model_lsq_best_selection <- tibble(index = seq(1,length(model_lsq_best_summ$rss)),
                                   rss = model_lsq_best_summ$rss,
                                   adjr2 = model_lsq_best_summ$adjr2,
                                   cp = model_lsq_best_summ$cp,
                                   bic = model_lsq_best_summ$bic)

    # model performance by RSS, adj R2, cp, and bic
  p1 <- ggplot(model_lsq_best_selection, aes(index, rss)) +
    geom_line() 
  
  max_adjr2_loc <- which.max(model_lsq_best_selection$adjr2) 
  p2 <- ggplot(model_lsq_best_selection, aes(index, adjr2)) +
    geom_line() + 
    annotate("point", x = max_adjr2_loc, y = model_lsq_best_selection$adjr2[max_adjr2_loc], color = "blue")
  
  min_cp_loc <- which.min(model_lsq_best_selection$cp) 
  p3 <- ggplot(model_lsq_best_selection, aes(index, cp)) +
    geom_line() + 
    annotate("point", x = min_cp_loc, y = model_lsq_best_selection$cp[min_cp_loc], color = "blue")
  
  min_bic_loc <- which.min(model_lsq_best_selection$bic)
  p4 <- ggplot(model_lsq_best_selection, aes(index, bic)) +
    geom_line() + 
    annotate("point", x = min_bic_loc, y = model_lsq_best_selection$bic[min_bic_loc], color = "blue")

  (p1 + p2) / (p3 + p4)
    
errors <- rep(NA, 17) # test data cross validation error
for (i in 1:17) {
  coefi <- coef(model_lsq_best, id = i)
  pred <- test_matrix[, names(coefi)] %*% coefi
  errors[i] <- mse(pred, test$Apps)
}
which.min(errors) # 12 - MSE: 1256814
plot(errors, xlab = "Number of predictors", ylab = "Test MSE", pch = 19, type = "b",col="red")
    
coef(model_lsq_best, which.min(errors))


# least squares - forward selection
model_lsq_forward <- regsubsets(Apps ~ ., data = train, nvmax = 18, method = "forward")

errors <- rep(NA, 17) # test data cross validation error
model_rsq <- rep(NA, 17)
for (i in 1:17) {
  coefi <- coef(model_lsq_forward, id = i)
  pred <- test_matrix[, names(coefi)] %*% coefi
  errors[i] <- mse(pred, test$Apps)
  model_rsq[i] <- rsq(pred, test$Apps)
}
which.min(errors) # 12 - MSE: 1256814
model_rsq[12] # R-Squared: .9138
plot(errors, xlab = "Number of predictors", ylab = "Test MSE", pch = 19, type = "b",col="red")

coef(model_lsq_forward, which.min(errors))

  ## ALTERNATE WAY - select automatically by AIC
model_null <- lm(Apps ~ 1, data = train)
model_full <- lm(Apps ~ ., data = train)

model_lsq_forward_2 <- step(model_null, scope = list(lower = model_null, upper = model_full), direction = "forward")
pred_lsq_forward_2 <- predict(model_lsq_forward_2, test)

mse(pred_lsq_forward_2, test$Apps) # MSE: 1259145
rsq(pred_lsq_forward_2, test$Apps) # R-squared: .9136

# least squares - backward selection
model_lsq_backward <- regsubsets(Apps ~ ., data = train, nvmax = 18, method = "backward")

errors <- rep(NA, 17) # test data cross validation error
for (i in 1:17) {
  coefi <- coef(model_lsq_backward, id = i)
  pred <- test_matrix[, names(coefi)] %*% coefi
  errors[i] <- mse(pred, test$Apps)
}
which.min(errors) # 12 - MSE: 1256814
plot(errors, xlab = "Number of predictors", ylab = "Test MSE", pch = 19, type = "b",col="red")

coef(model_lsq_bbackward, which.min(errors))

# ridge regression
grid <- 10 ^ seq(4, -2, length = 100)

cv_ridge <- cv.glmnet(train_matrix, train$Apps, alpha = 0, lambda = grid, thres = 1e-12)
best_lambda <- cv_ridge$lambda.min

model_ridge <- glmnet(train_matrix, train$Apps, alpha = 0, lambda = best_lambda, thres = 1e-12)
pred_ridge  <- predict(model_ridge, s = best_lambda, newx = test_matrix)

mse(pred_ridge, test$Apps) # MSE: 1261599
rsq(pred_ridge, test$Apps) # R-Squared: .9135

# lasso
cv_lasso <- cv.glmnet(train_matrix, train$Apps, alpha = 1, lambda = grid, thres = 1e-12)
best_lambda_lasso <- cv_lasso$lambda.min

model_lasso <- glmnet(train_matrix, train$Apps, alpha = 1, lambda = best_lambda_lasso, thres = 1e-12)
pred_lasso  <- predict(model_lasso, s = best_lambda_lasso, newx = test_matrix)
coef(model_lasso)

mse(pred_lasso, test$Apps) # MSE: 1261591
rsq(pred_lasso, test$Apps) # R-Squared: .9135

# pcr
model_pcr <- pcr(Apps ~ ., data = train, scale = TRUE, validation = "CV")
validationplot(model_pcr, val.type = "MSEP")

pred_pcr <- predict(model_pcr, test, ncomp = 17)

mse(pred_pcr, test$Apps) # MSE: 1261630
rsq(pred_pcr, test$Apps) # R-Squared: .9135

# pls
model_pls <- plsr(Apps ~ ., data = train, scale = TRUE, validation = "CV")
validationplot(model_pls, val.type = "MSEP")

pred_pls <- predict(model_pls, test, ncomp = 9)

mse(pred_pls, test$Apps) # MSE: 1286077
rsq(pred_pls, test$Apps) # R-Squared: .9118

{% endhighlight R %}
