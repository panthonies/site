---
layout: post
title: "Support Vector Machines"
description: "Support vector machines (SVMs) are often considered one of the best 'out of the box' classifiers. The simple maximal margin classifier can be generalized to the support vector classifier, which can be further generalized to the support vector machine."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents 

- [I. Maximal Margin Classifiers](#i-maximal-margin-classifier)
- [II. Support Vector Classifiers](#ii-support-vector-classifier)
- [III. Support Vector Machines](#iii-support-vector-machines)
- [IV. Applications in R](#iv-applications-in-r)

</div>


### I. Maximal Margin Classifiers

The maximal margin classifier, or the **optimal separating hyperplane**, is the separating hyperplane[^1] that is farthest away from the training observations. This classifier depends directly on only a small subset of the training observations, called **support vectors**.

The maximal margin hyperplane is the solution to the optimization problem:

[^1]: A hyperplane in $$p$$-dimensional space is a flat affine subspace of dimension $$p - 1$$, and is defined by $$\beta_0 + \beta_1 X_1 + ... + \beta_p X_p = 0 $$. 

$$ \max_{\beta_0, \beta_1, ..., \beta_p, M} M \text{ subject to } 
\begin{cases} 
\sum_{j = 1}^p \beta_j^2 = 1 \\
y_i (\beta_0 + \beta_1x_{i1} + ... + \beta_p x_{ip}) \geq M, \forall i = 1, ..., n
\end{cases}$$

Where:
- $$M$$ is the **margin**, the minimal orthogonal distance from the training observations to the hyperplane
- The first condition guarantees a unique solution where the orthogonal distance from the $$i$$th observation to the hyperplane fulfills the second condition.
- The second condition guarantees that each observation is on the correct side of the hyperplane.

We then classify each observation based on where it lies in relation to the hyperplane derived from $$\beta_0, \beta_1, ..., \beta_p$$ from the optimization problem above.

---
---
---
### II. Support Vector Classifiers

If no separating hyperplane exists, then there is no maximal margin classifier. In the non-separable case, the generalization of the maximal margin classifier is called the support vector classifier.

The support vector classifier, or the **soft margin classifier**, provides greater robustness to individual observations, and better classification of most of the training observations than the maximal margin classifier.

$$ \max_{\beta_0 , ... , \beta_p, \epsilon_1, \epsilon_n, M} M \text{ subject to } 

\begin{cases} 
\sum_{j = 1}^p \beta_j^2 = 1 \\
y_i (\beta_0 + \beta_1x_{i1} + ... + \beta_p x_{ip}) \geq M (1 - \epsilon_i) \\
\epsilon_i \geq 0, \sum_{i = 1}^n \epsilon_i \leq C
\end{cases}$$

Where:
- Where $$\epsilon_i$$ are the slack variables whose sum is less than or equal to the tuning parameter $$C$$.
  - If $$\epsilon_i = 0$$, then the $$i$$th observation is on the correct side of the margin
  - If $$\epsilon_i > 0$$, then the $$i$$th observation is on the wrong side of the margin
  - If $$\epsilon_i > 1$$, then the $$i$$th observation is on the wrong side of the hyperplane

- $$C$$ is a "budget" for how much the margin can be violated. No more than $$C$$ observations can violate the hyperplane at once. 
  - Choose with cross validation. 
  - A higher $$C$$ means the model is more tolerant of margin violations.
  - If $$C = 0$$, then the solution is equivalent to that of the maximal margin classifier.
  - A smaller $$C$$ corresponds with high variance and low bias; a larger $$C$$ corresponds with high bias and low variance.


The support vectors in this model (the only observations that affect the hyperplane) are the observations that are on the margin or that violate the margin. Support vector classifiers are robust to observations far away from the hypersplane, unlike methods such as linear discriminant analysis.

---
---
---
### III. Support Vector Machines

In non-linear decision boundaries, we can modify the $$y_i \bigl( f(x) \bigr) \geq M (1 - \epsilon_i) $$ condition in the support vector classifier to include non-linear functions.

The solution to the linear support vector classifier involves only the inner products of the observations, $$ \langle x, x_i \rangle = \sum_{j = 1}^p x_{ij} x_{i'j} $$. It can be shown that the linear support vector classifier can be represented by:

$$f(x) = \beta_0 + \sum_{i \in S}^n \alpha_i \langle x, x_i \rangle $$

Where:
- There are $$n \choose 2$$ inner products between all pairs of training observations needed to estimate the parameters $$\alpha_1, ..., \alpha_n, \beta_0$$.
- $$S$$ is the collection of indices of support points, since $$\alpha_i$$ is only non-zero for support vectors.


We can generalize the support vector classifier with a kernel, $$K(x_i, x_{i'})$$, a function that quantifies the similarity of two observations, instead of the inner product (linear kernel). 

**Support vector machines use the support vector classifier with a non-linear kernel:** 

$$f(x) = \beta_0 + \sum_{i \in S} \alpha_i K(x, x_i)$$

Examples of different kernels:

- Linear Kernel (inner product):

$$K(x_i, x_{i'}) = \sum_{j = 1}^p x_{ij} x_{i'j} $$

- Polynomial Kernel of degree d 

$$K(x_i, x_{i'}) = \biggl( 1 + \sum_{j = 1}^p x_{ij} x_{i'j} \biggr)^d $$

- Radial Kernel (very local behavior)

$$K(x_i, x_{i'}) = \exp \biggl( - \gamma \sum_{j = 1}^p (x_{ij} x_{i'j})^2 \biggr) $$

SVMs are computational nice, since they only need to calculate $$K(x_i, x_{i'})$$ for $$n \choose 2$$ distinct pairs $$i, i'$$. This can be done without explicitly working in an enlarged feature space. The radial kernel feature space is implicit and infinite-dimensional, so we need a kernel to use it.

**An ROC curve** of the false positive rate against the true positive rate can help graphically compare the performance of SVMs.

<span class="boxheader">Extension to more than two classes</span>

One-versus-one classification
- Constructs $$k \choose 2$$ SVMs, each of which compares a pair of classes.
- Classify a test observation into each of the $$k \choose 2$$ pairs, tally the number of times it's classified into each of those $$K$$ classes, and assign it to the class for which it was the most frequently assigned.

One-versus-all classification
- Fit $$K$$ SVMs. For each fit, compare one of the $$K$$ classes to the other $$K - 1$$ classes.
- Assign observations to the class for which $$\beta_{0k} + \beta_{1k}x_1^* + ... + \beta_{pk} x_p^*$$ is largest.

Note: Support vector regression also exists.

<span class="boxheader">Relationship to logistic regression</span>

The loss + penalty function of SVMs, shown below, is very similar to the loss function of logistic regression.

$$\min_{\beta_0, ..., \beta_p} \Biggl\{ \sum_{i = 1}^n \max [0, 1 - y_i f(x_i)] + \lambda \sum_{j = 1}^p \beta_j ^ 2 \Biggr\} $$

$$\lambda$$ is a nonnegative tuning parameter. When $$\lambda$$ is large, then the coefficients are small and more violations to the margin are tolerated for lower variance and higher bias. This equation takes the "Loss + Penalty" form that many other models take, and the loss function of an SVM is known as "hinge loss."

SVMs perform better when classes are well-separated, while logistic regression performs better with overlapping classes.

A graphical comparison of SVM loss vs. logistic regression loss is shown below.[^2]

<center>{% include image.html path="documentation/05-01-svm-logit.png"
                      path-detail="documentation/05-01-svm-logit.png"
                      alt="SVM Loss vs Logit Loss" %}</center>

[^2]: Source: James et. al, *Intro to Statistical Learning 7th edition*, pg. 358

---
---
---
### IV. Applications in R

{% highlight R %}
library("ISLR")
library("tidyverse")
library("e1071")
{% endhighlight R %}

{% highlight R %}

### support vector classifier
# generate variables
set.seed(1)
x <- matrix(rnorm(20*2), ncol = 2)
y <- c(rep(-1, 10), rep(1, 10))
plot(x, col = 3-y)

# fit svm with svm()
data <- data.frame(x = x, y = as.factor(y))
fit_svm <- svm(y ~ ., data = data, kernel = "linear", cost = 10, scale = FALSE)
summary(fit_svm)
fit_svm$index # view the support vectors
plot(fit_svm, data)

# cross validation with tune()
svm_cv <- tune(svm, y ~ ., data = data, kernel = "linear",
               ranges = list(cost = c(.001, .01, .1, 1, 5, 10, 100)))
summary(svm_cv)
svm_best_model <- svm_cv$best.model

# generate test data set
xtest <- matrix(rnorm(20 * 2), ncol = 2)
ytest <- sample(c(-1, 1), 20, rep = TRUE)
xtest[ytest == 1,] <- xtest[ytest == 1,] + 1
data_test <- data.frame(x = xtest, y = as.factor(ytest))

ypred <- predict(svm_best_model, data_test)
table(predict = ypred, truth = data_test$y)

### support vector machine
# generate data
set.seed(1)
x <- matrix(rnorm(200*2), ncol = 2)
x[1:100,] <- x[1:100,] + 2
x[101:150,] <- x[101:150,] - 2
y <- c(rep(1, 150), rep(2, 50))
data <- data.frame(x = x, y = as.factor(y))

plot(x, col = y)

# split into training and testing and run svm
train <- sample(200, 100)
fit_svm <- svm(y ~ ., data = data[train,], kernel = "radial", gamma = 1, cost = 1)
plot(fit_svm, data[train,])

summary(fit_svm)

fit_svm <- svm(y ~ ., data = data[train,], kernel = "radial", gamma = 1, cost = 100000) 
plot(fit_svm, data[train,]) # note: higher cost -> more irregular boundary

# cross validation using tune()
set.seed(1)
svm_cv <- tune(svm, y ~ ., data = data[train,], 
               kernel = "radial", 
               ranges = list(cost = c(.1, 1, 10, 100, 1000), gamma = c(.5, 1, 2, 3, 4)))
summary(svm_cv) # best = cost 1, gamma .5

table(true = data[train, "y"], 
      pred = predict(svm_cv$best.model, newdata = data[train,]))

# predict with best model
table(true = data[-train, "y"], 
      pred = predict(svm_cv$best.model, newdata = data[-train,]))


### ROC curves with the ROCR package
library(ROCR)

rocplot <- function(pred, truth, ...) {
  predob <- prediction(pred, truth)
  perf <- performance(predob, "tpr", "fpr")
  plot(perf, ...)
}

par(mfrow = c(1, 2))

# for some reason we have to order the train/test by Y in order to get correct ROC curve (instead of reverse ROC)
trainz <- data[train,]
trainz <- trainz[order(trainz$y, decreasing = TRUE),]

testz <- data[-train,]
testz <- testz[order(testz$y, decreasing = TRUE),]

fit_svm_opt <- svm(y ~ ., data = trainz, kernel = "radial", gamma = .5, cost = 1, decision.values = T)
pred_values <- attributes(predict(fit_svm_opt, trainz, decision.values = TRUE))$decision.values
rocplot(pred_values, trainz$y, main = "Training Data")

fit_svm_flex <- svm(y ~ ., data = trainz, kernel = "radial", gamma = 50, cost = 1, decision.values = T)
pred_values <- attributes(predict(fit_svm_flex, trainz, decision.values = TRUE))$decision.values
rocplot(pred_values, trainz$y, add = T, col = "red")

pred_values <- attributes(predict(fit_svm_opt, testz, decision.values = TRUE))$decision.values
rocplot(pred_values, testz$y, main = "Test Data")
pred_values <- attributes(predict(fit_svm_flex, testz, decision.values = TRUE))$decision.values
rocplot(pred_values, testz$y, add = T, col = "red")


### SVM with multiple classes - one versus one approach
# generate data
set.seed(1)
x <- rbind(x, matrix(rnorm(50 * 2), ncol = 2))
y <- c(y, rep(0, 50)) # based on previous y
x[y == 0, 2] <- x[y == 0, 2] + 2

data <- data.frame(x = x, y = as.factor(y))
par(mfrow = c(1, 1))
plot(x, col = (y + 1))

# fit svm
fit_svm <- svm(y ~ ., data, kernel = "radial", cost = 10, gamma = 1)
plot(fit_svm, data)
{% endhighlight R %}
