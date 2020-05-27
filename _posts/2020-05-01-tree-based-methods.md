---
layout: post
title: "Tree-Based Methods"
description: "Decision trees, which divide the predictor space into regions, are simple and useful for interpretation. Their predictive power can be improved with bagging, random forests, and boosting."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents

- [I. Overview of Decision Trees](#i-overview-of-decision-trees)
- [II. Regression Trees](#ii-regression-trees)
- [III. Classification Trees](#iii-classification-trees)
- [IV. Methods: Bagging](#iv-method-bagging)
- [V.  Methods: Random Forests](#v-method-random-forests)
- [VI. Methods: Boosting](#vi-method-boosting)
- [VII. Applications in R](#vii-applications-in-r)

</div>

### I. Overview of Decision Trees

Trees involve stratifying or segmenting the predictor space into a number of simple regions, where the prediction of an observation is typically the mean or mode of the training observations in the region to which it belongs.

$$ \begin{align}
\text{Trees: } f(x) & = \sum_{m = 1}^M c_m * 1_{(x \in R_m)}\\
\text{Linear models: } f(x) & = \beta_0 + \sum_{j = 1}^p x_j \beta_j
\end{align}$$

**Pros:**
- Easy to explain
- Can be displayed graphically and are easily interpreted
- Can easily handle qualitative predictors without dummy variables
- May more closely mirror human decision-making than least squares or other classical models

**Cons:**
- Not very robust to new observations
- In general, lower predictive accuracy than other regression and classification approaches unless we use bagging, random forests, and boosting, which increase predictive performance but decrease interpretability

---
---
---
### II. Regression Trees

A regression tree is typically drawn upside down, with the terminal nodes (leaves) at the bottom. The predictor space is split at a number of internal nodes connected by branches. To build a regression tree:

1. Divide the predictor space, the set of all posible values for $$X_1, X_2, ..., X_p$$ into $$J$$ distinct, non-overlapping regions, $$R_1, R_2, ..., R_J$$.
2. For every observation that falls into region $$R_j$$, predict the response as the mean of the response values for the training observations in $$R_j$$.

We choose the regions to minimize the RSS: $$ \sum_{j = 1}^J \sum_{i \in R_j} (y_i - \hat y_{R_j}) ^ 2 $$, where $$\hat y_{R_j}$$ is the mean response of training observations in the $$j$$th box. This is a top down, greedy approach called **recursive binary splitting.**[^1]

<span class="boxheader">Recursive Binary Splitting</span>

[^1]: Recursive binary splitting is top-down because it begins at the top of the tree where all observations begin at a single region. It is greedy because at each step of the tree-building process, the best split is made at that step, rather than looking ahead and picking a split that will lead to a better tree in a future step.

To perform recursive binary splitting, we consider $$X_1, ..., X_p$$ and all possible cutpoints $$s$$ for each predictor, then choose the predictor and cutpoint to minimize RSS. In formal terms,

$$ \text{Define half-planes } R_1(j, s) = \{ X \vert X_j < s \} \text{ and } R_2(j, s) = \{ X \vert X_j \geq s \} $$

$$\min_{j, s} \sum_{i: x \in R_{1(j, s)}} (y_i - \hat y_{R_1}) ^ 2 + \sum_{i: x \in R_{2(j, s)}} (y_i - \hat y_{R_2}) ^ 2 $$

Where $$\{ X \vert X_j < s \}$$ represents the region of the predictor space in which $$X_j$$ takes as value less than $$s$$, and $$\hat y_{R_1}$$ and $$\hat y_{R_2}$$ are the mean responses for the training observations in $$R_1 (j, s)$$ and $$R_2 (j, s)$$, respectively. We repeat this process by splitting one of the two previously identified regions until a stopping condition is reached. 

<span class="boxheader">Cost Complexity Pruning</span>

The recursive binary splitting process is likely to create an overly complex tree and overfit the data. To address this, we can lower the variance with cost-complexity pruning or weakest link pruning with a nonnegative tuning parameter $$\alpha$$:

For each value of $$\alpha$$, $$\exists T \subset T_0$$ such that the following equation is minimized:

$$\sum_{m = 1}^t \sum_{i: x \in R_m} (y_i - \hat y_{R_m}) ^ 2 + \alpha \vert T \vert $$

- $$ \vert T \vert $$ is the number of terminal notes
- $$R_m$$ is the subset of predictor space corresponding to the $$m$$th terminal node
- $$\hat y_{R_m}$$ is the predicted response
- $$\alpha$$ controls the tradeoff beween complexity and fit; as $$\alpha \to \infty$$, a small subtree is chosen

We choose $$\alpha$$ with K-fold cross validation, averaging results for each value of $$\alpha$$, and selecting $$\alpha$$ to minimize the average error.

<span class="boxheader">Summarized Algorithm:</span>
1. Use recursive binary pruning to grow a large tree until each terminal node reaches $$X$$ minimum number of observations.
2. Apply cost-complexity pruning to obtain a sequence of best subtees as a functio of $$\alpha$$.
3. Use k-fold cross-valudation to choose $$\alpha$$ by repeating the first two steps on the $$k$$th fold and averaging all of the MSEs.
4. Return the subtree from step 2 for the chosen best value of $$\alpha$$.

---
---
---
### III. Classification Trees

Classification trees are similar to regression trees; they are grown with recursive binary splitting (refer to the previous section). The predicted class is chosen by the most common class in the region.

However, we need an alternative to RSS to use as criterion for making the binary splits. There are a several types of error we can choose to minimize when building classification trees:

- **Classification error** 
  - $$ E = 1 - \max_k (\hat p_{mk}) $$, where $$\hat p_{mk}$$ is the proportion of training observations in region $$m$$ from class $$k$$.
  - this type of error is not very good to use with trees

- **Gini Index**
  - $$G = \sum_{k = 1}^K \hat p_{mk} (1 = \hat p_{mk}) $$ ;
  - this is a measure of total variance across the $$K$$ classes
  - measures node purity; a smaller value means that the node contains predominantly observations from a single class
  - node purity is important because it is a marker for prediction certainty

- **Entropy**
  - $$D = - \sum_{k = 1}^K \hat p_{mk} \log \hat p_{mk} $$ ;
  - note that entropy is always positive, since $$ - \hat p_{mk} \log \hat p_{mk} > 0 $$
  - this is small if $$ \hat p_{mk}$$'s are all near 0 or all near 1 (i.e. if the $$m$$th node is pure)
  - similar to the Gini index, can be used as an alternative


---
---
---
### IV. Method: Bagging

Bagging, or bootstrap aggregation, reduces the variance of a statistical learning method. Decision trees suffer from high variance, so we can reduce the variance by taking many training sets from the population.

Generate $$B$$ different bootstrapped data sets, train the method on the $$b$$th data set to obtain $$\hat f ^ {*b} (x)$$, then average all of the predictions to obtain the function obtained from bagging:

$$ \hat f_{\text{bag}} (x) = \frac{1}{B} \sum_{b = 1}^B \hat f ^ {*b} (x) $$

For qualitative responses, the overall prediction of the bagging tree is is the most commonly occurring class among the $$B$$ predictions.

$$B$$, the number of trees, is not a critical parameter! In practice, we just need to use a large B so that the error has settled down, which is typically a value greater than 100.

**Out-of-bag Error Estimation** is a straightforward way to estimate the test error of a bagged model.
- Each bagged tree makes use of about 2/3 of the data.
- For the $$i$$th observation, there are about $$B$$/3 predictions for $$i$$ in which $$i$$ is *out-of-bag*.
- Obtain a single prediction for the $$i$$th observation with an average (if regression) or majority vote (if classification)
- Calculate the out-of-bag MSE to estimate the test error. This is virtually equivalent to LOOCV.

**Variable importance measures**
- Note that bagging increases prediction accuracy, but lowers interpretability.
- To measure variable importance for **regression**, take the average over all $$B$$ trees of the total amount that RSS degreases due to splits over a given predictor.
- To measure variable importance for **classification**, take the average over all $$B$$ trees of that total amount that the Gini index decreases by splits over a given predictor.


---
---
---
### V. Method: Random Forests

Random Forests are an extension of bagging, where we generate $$B$$ different bootstrapped data sets. However, each time a split is considered, a random sample of $$m$$ predictors is chosen as split candidates. Usually, $$m \approx \sqrt p$$. 

- If there is 1 very strong predictor, then bagging will not substantially reduce the variance of the tree fit.
- In random forests, $$\frac{p - m}{p}$$ predictors will not consider the strong predictor, which decorrelates the trees, making the average less variable and more reliable. In particular, use a small value of $$m$$ when there are a large number of correlated variables.
- Like bagging, any value of $$B$$ will not overfit the data, so choose $$B$$ such that the error has settled down. 

---
---
---
### VI. Method: Boosting

Boosting is a general approach that can be used for many statistical learning methods, including decision trees. It works similarly to bagging, except trees are grown sequentially, and fit on a modified version of the original data set.

There are three **tuning parameters** with boosting:
1. The number of trees, $$B$$.
  - choose with cross validation, since a large $$B$$ can lead to overfitting (slowly)
2. The shrinkage parameter, $$\lambda$$, which controls the rate at which boosting learns.
  - typically set to .01 or .001
  - note that a very small $$\lambda$$ can require a very large $$B$$ for good performance
3. The number of splits in each tree, $$d$$, which controls the complexity of the boosted trees
  - often, $$d = 1$$ works well; this is when each tree is a stump with a single split
  - $$d$$ is also the interaction depth, and controls the interaction order of boosted models

**Algorithm: Boosting for Regression Trees**
1. Set $$\hat f(x) = 0$$ , and $$r_i = y_i$$ for all $$i$$ in the training set
2. For $$b = 1, 2, ..., B$$:
  - Fit a tree $$\hat f^b$$ with $$d$$ splits ($$d+1$$ terminal nodes) to the training data $$(X, r)$$
  - Update $$\hat f$$ by adding a shrunken version of the new tree: $$ \hat f(x) \leftarrow \hat f(x) + \lambda \hat f^b(x_i) $$
  - Update the residuals: $$ r_i \leftarrow r_i - \lambda \hat f^b(x_i) $$ 
3. Output the boosted model:

$$ \hat f(x) = \sum_{b = 1}^B \lambda \hat f^b(x) $$

Boosting learns slowly by fitting decision trees to the residuals of the current tree. Trees are grown sequentially, and the shrinkage parameter $$\lambda$$ slows down the process.

Note: smaller trees can help with interpretability. For example, using stumps can also be interpreted as creating an additive model.


---
---
---
### VII. Applications in R

{% highlight R %}
library("ISLR")
library("tree")
library("tidyverse")
library("caret")
{% endhighlight R %}

{% highlight R %}
attach(Carseats)

### fitting classification trees

# prepare data
High <- ifelse(Sales <= 8, "No", "Yes")
Carseats <- data.frame(Carseats, High)

# using the tree() function
tree_carseats <- tree(High ~ .-Sales, Carseats)
summary(tree_carseats)
plot(tree_carseats) # display tree structure
text(tree_carseats, pretty = 0) # display node labels

# split training and test
train <- sample(1:nrow(Carseats), 200)
test_Carseats <- Carseats[-train,]
test_High <- High[-train]

# run model on training data
tree_carseats <- tree(High ~ . - Sales, Carseats, subset = train)
tree_pred <- predict(tree_carseats, test_Carseats, type = "class")

table(tree_pred, test_High) # correct prediction rate = .7

# cross validation for tree pruning to select final # of nodes
cv_carseats <- cv.tree(tree_carseats, FUN = prune.misclass)
cv_carseats # lowest cv error rate (dev) = tree with 9 terminal nodes

par(mfrow = c(1,2))
plot(cv_carseats$size, cv_carseats$dev, type = "b") # plot cv error rate vs tree size
plot(cv_carseats$k, cv_carseats$dev, type = "b") # plot cv error rate vs number of terminal nodes

prune_carseats <- prune.misclass(tree_carseats, best = 3)
par(mfrow = c(1,1))
plot(prune_carseats)
text(prune_carseats, pretty = 0)

# check prediction on test data set
tree_pred <- predict(prune_carseats, test_Carseats, type = "class")
table(tree_pred, test_High) # correct prediction rate = .72


### fitting regression trees
library("MASS")

set.seed(1)
train <- sample(1:nrow(Boston), nrow(Boston)/2)

tree_boston <- tree(medv ~ ., Boston, subset = train)
summary(tree_boston)
plot(tree_boston)
text(tree_boston, pretty = 0)

# cross validation to prune tree
cv_boston <- cv.tree(tree_boston)
plot(cv_boston$size, cv_boston$dev, type = "b") # pruning does not help

# if you wanted to prune, you would do this:
#prune_boston <- prune.tree(tree_boston, best = 7)
#plot(prune_boston)
#text(prune_boston, pretty = 0)

# predict the test set
tree_predict <- predict(tree_boston, newdata = Boston[-train,])
test_boston <- Boston[-train,"medv"]
plot(tree_predict, test_boston)
abline(0,1)
mean((tree_predict - test_boston)^2) # test MSE = 35.3


### bagging and random forests
library("randomForest")

# bagging
bag_boston <- randomForest(medv ~ ., data = Boston, subset = train, mtry = 13, importance = TRUE)
bag_boston

bag_predict <- predict(bag_boston, newdata = Boston[-train,])
plot(bag_predict, test_boston)
abline(0, 1)
mean((bag_predict - test_boston)^2) # test MSE: 23.5

# random forest - change the mtry parameter
rf_boston <- randomForest(medv ~ ., data = Boston, subset = train, mtry = 6, importance = TRUE)
rf_predict <- predict(rf_boston, newdata = Boston[-train,])
plot(rf_predict, test_boston)
abline(0, 1)
mean((rf_predict - test_boston)^2) # test MSE: 20.2

# rank the importance of each variable
importance(rf_boston)
varImpPlot(rf_boston)

### boosting
library("gbm")

# note: set distribution = "gaussian" for regression, "bernoulli" for classification

boost_boston <- gbm(medv ~ ., 
                    data = Boston[train,], 
                    distribution = "gaussian", # gaussion = regression
                    n.trees = 5000,            # 5000 trees, default 100
                    interaction.depth = 4,     # limits the depth of each tree to 4, default 1
                    shrinkage = .2)          # shrinkage parameter, default .1

# variable importance plots
summary(boost_boston) # relative influence statistics
par(mfrow = c(1,2)) 
plot(boost_boston, i = "rm") # partial dependence plots
plot(boost_boston, i = "lstat")

# predict medv in test set
boost_predict <-  predict(boost_boston, newdata = Boston[-train,], n.trees = 5000)
mean((boost_predict - test_boston) ^ 2) # test MSE: 17.1

{% endhighlight R %}
