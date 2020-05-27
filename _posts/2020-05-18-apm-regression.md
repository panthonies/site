---
layout: post
title: "Regression Techniques for Predictive Modeling"
description: "Buckle up! This is going to be a long one."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents
<ul>
<li><a href = "#introduction">Introduction</a></li>
<li><a href = "#i-linear-models">I. Linear Models</a>
    <ul class="hi"><li><a href = "#linear-regression">Linear Regression</a></li>
    <li><a href = "#partial-least-squares-and-principal-components-regression">Partial Least Squares and Principal Components Regression</a></li>
    <li><a href = "#penalized-models-ridge-lasso-and-elastic-net">Penalized Models: Ridge, Lasso, and Elastic Net</a></li></ul></li>

<li><a href = "#ii-nonlinear-models">II. Non-Linear Models</a>
    <ul class="hi"><li><a href = "#neural-networks">Neural Networks</a></li>
    <li><a href = "#multivariate-adaptive-regression-splines-mars">Multivariate Adaptive Regression Splines (MARS)</a></li>
    <li><a href = "#support-vector-machines">Support Vector Machines</a></li>
    <li><a href = "#k-nearest-neighbors">K-Nearest Neighbors</a></li></ul></li>

<li><a href = "#ii-regression-trees-and-rule-based-models">IV. Regression Trees and Rule-Based Models</a>
    <ul class="hi"><li><a href = "#basic-regression-trees">Basic Regression Trees</a></li>
    <li><a href = "#regression-model-trees">Regression Model Trees</a></li>
    <li><a href = "#rule-based-models">Rule-Based Models</a></li>
    <li><a href = "#bagged-trees">Bagged Trees</a></li>
    <li><a href = "#random-forests">Random Forests</a></li>
    <li><a href = "#boosting">Boosting</a></li>
    <li><a href = "#cubist">Cubist</a></li></ul></li>
</ul>
</div>



## Introduction
---
---
---

<small>Note: I will try not to repeat details I've covered in my previous posts on [linear regression](linear-regression), [regularization](linear-model-selection-regularization), [splines and GAMs](moving-beyond-linearity), [tree-based methods](tree-based-methods), and [SVMs](support-vector-machines). This post focuses more on model application, while my previous posts focus on defining and explaining the method.</small>


Regression models predict a numeric outcome. Their performance is usually measured with:
- **Root Mean Squared Error (RMSE)** is the most common perfromance measure, and represents how far (on average) the residuals are from zero or the average distance between the observed values and model predictions.
- $$R^2$$, the proportion of outcome variance explained by the model. When interpreting R-squared, remember that the value is dependent on the variation of the outcome -- a high value doesn't necessarily indicate a "good" model unless it meets your specific goals.  
- Spearman's rank correlation is often used if the model is judged by its ability to rank new samples as opposed to numerical predictive accuracy.

The **bias-variance tradeoff** can be illustrated with the mean squared error:

$$ \begin{align} 
\text{MSE} & = \frac{1}{n} \sum_{i = 1}^n (y_i - \hat y_i) ^ 2 \\
E\left[\text{MSE}\right] & = \sigma^2 + (\text{Model Bias})^2 + \text{Model Variance}
\end{align} $$ 

The **bias** reflects how close the functional form of the model can get to the true relationship between the predictors and outcome, and the **variance** reflects how sensitive the model is to additional data points. Generally, a simple model will have low variance and high bias, while a complex model will have high variance and low bias.

{% highlight r %}
library("caret")
R2(predicted, observed)
RMSE(predicted, observed)

cor(predicted, observed)
cor(predicted, observed, method = "spearman")
{% endhighlight r %}

---
---
---

# I. Linear Models
---
---
---

Linear regression-type models are highly interpretable, and standard errors can be computed to assess the statistical significance of each predictor. However, they assume that the relationship between the predictors and response falls along a hyperplane. Although linear models can be augmented to capture interactions and higher degree relationships, nonlinear relationships may not be adequately captured.

## Linear Regression
---

*[Note: I wrote a more comprehensive overview of linear regression* [*here.*](linear-regression)*]*

The objective of ordinary least squares linear regression is to find the plane that minimizes the sum of squared errors between the observed and predicted response.

There are no tuning parameters. However, we must use training and validation techniques to understand its predictive ability. When reampling a dataset with many predictors (for example, 100 samples and 75 predictors), be mindful that resampling may not be able to find a unique set of regression coefficients if the resampling scheme only uses ~2/3 of the data. 

<span class="boxheader">Drawbacks:</span>
- Does not work when the number of predictors is greater than the number of samples. If the number of predictors is large, reduce the dimension of the predictor space with techniques such as PCA.
- Not good when a predictor is a linear combination of other predictors. When collinearity exists, then a linear model can still be fit but we lose the ability to meaningfully interpret the coefficients. We can diagnose multicollinearity with the variance inflation factor (VIF).
- Not good for modeling non-linear relationships. Diagnose this with residual plots.
- Not good at handling to outliers, since they have exponentially larger residuals. We can make linear regression more robust by minimizing a metric other than SSE, such as mean absolute error and the Huber function (see below[^1]).

<center>{% include image.html path="documentation/05-22-loss.png"
                      path-detail="documentation/05-22-loss.png"
                      alt="loss" %}</center>

[^1]: Source: Kuhn and Johnson, *Applied Predictive Modeling* (2013), pg. 110

## Partial Least Squares and Principal Components Regression
---

*[Note: I wrote a more comprehensive overview of PLS and PCR* [*here.*](linear-model-selection-regularization#iv-dimension-reduction-techniques)*]*

**Partial Least Squares (PLS)** finds linear combinations of the variables chosen to maximally summarize their covariance with the response. PLS is a supervised dimension reduction procedure that should be used when there are correlated predictors and a linear regression-type solution is desired.

**Principal Components Regression (PCR)** finds linear combinations of the variables to maximally summarize their variance without any regard to the response. However, if the variability of the predictors is not related to the variability of th eresponse, then PCR can have difficulty identifying a predictive relationship when one might actually exist. 

In practice, PLS and PCR have **similar predictive ability**. The number of components needed in PLS is always less than or equal to that of PCA to achieve the same predictive power.

<span class="boxheader">Good to know:</span>
- It's important to pre-process the data by **centering and scaling** to enure that variation in each variable is treated equally. 
- **"Variable importance in the projection" (VIP)** can be assessed by the size of the normalized weight and amount of variation in the response that is explained by the component. In other words, the importance of the $$j$$th predictor is given by a fraction where:
    - the numerator is a weighted sum of the normalized weights corresponding to the $$j$$th predictor. The $$j$$th normalized weight of the $$k$$th component, $$w_{kj}$$, is scaled by the amount of variation in the response explained by the $$k$$th component.
    - the denominator is the total amount of response variation explained by all $$k$$ components.
    - the rule of thumb is that values over 1 have important predictive power.

There have been many advances in computational time and dealing with nonlinear predictor spaces. PLS and PCR require significant effort to model nonlinear relationships, especially when the number of predictors is large. For cases where non-linear modeling is important, other techniques that can more naturally identify nonlinear structures are recommended over PLS and PCR.

In R, PLS and PCR have been implemented in the ```pls``` package, and can be specified in ```caret``` with method values of ```"pls"```,```"oscorespls"```, ```"simpls"```, and ```"widekernelpls"```.

## Penalized Models: Ridge, Lasso, and Elastic Net
---

*[Note: I wrote a more comprehensive overview of lasso and ridge regression* [*here.*](linear-model-selection-regularization#iii-shrinkage)*]*

When the linear regression model overfits the data, or when there are issues with collinearity leading to high variance, regularizing the parameter estimates can increase the bias of the model and increase predictive accuracy. It is common that a small increase in bias can produce a substantial drop in variance.

**Ridge Regression** imposes an $$L_2$$ penalty, which signifies that a second order penalty (the square) is being used on the parameter estimates. This shrinks the coefficient estimates as the penalty becomes large, but does not conduct feature selection. Ridge reg

**Lasso** imposes an $$L_1$$ penalty, penalizes the sum of the absolute values of the coefficients, and performs feature selection. The $$L_1$$ penalty has also been extended for use in LDA, PLS, and PCA. 

The **Elastic Net model** is a generalization that combines the $$L_1$$ and $$L_2$$ penalties and more effectively deals with groups of high-correlated predictors.

$$ \text{SSE}_{\text{Enet}} = \sum_{i = 1}^n (y_i - \hat y_i)^2 + \lambda_1 \sum_{j = 1}^P \beta_j^2 + \lambda_2 \sum_{j = 1}^P \vert \beta_j \vert $$

<span class="boxheader">Good to know:</span>
- When dealing with **correlated predictors**, ridge regression shrinks their coefficients toward each other, allowing them to borrow strength from each other, while lasso will tend to pick one and ignore the rest.
- Lasso was significantly advanced with the **Least Angle Regression (LARS)** model, which is a broad framework that encompasses the lasso and similar model. LARS can be used to fit lasso models more efficiently, especially in high-dimensional problems. 

In R, penalized models have been implemented in ```MASS```, ```elasticnet```, ```glmnet```, and many other packages. They can be trained in ```caret``` with method values of ```"ridge"```, ```"lasso"```, ```"enet"```, ```"glmnet"```, and more. Implementations of variations on lasso include the packages ```biglars``` (large data sets), ```FLLat``` (fused lasso), ```grplasso``` (group lasso), and ```relaxo``` (relaxed lasso).

---
---
---
# II. Non-Linear Models
---
---
---
In the inherently nonlinear models described below, the exact form of nonlinearity does not need to be known explicitly or specified prior to model training. 


## Neural Networks
---

In neural networks, the outcome is modeled by a set of unobserved variables, called hidden units, consisting of linear combinations of the original predictors which have been transformed by a nonlinear function $$g(\cdot)$$. There are no constraints on the hidden units, so the coefficients in each unit probably don't represent any coherent information.

A common neural network implementation has hidden units that have been transformed by the logistic/sigmoidal function:

$$\begin{align} 
h_k(x) & = g \biggl( \beta_{0k} + \sum_{i = 1}^P x_j \beta_{jk} \biggr) \text{, where } \\
g(u) & = \frac{1}{1 + e^{-u}} 
\end{align}$$

The hidden units then model the outcome with a linear relationship:

$$f(x) = \gamma_0 + \sum_{k = 1}^H \gamma_k h_k$$

Note that there are a total of $$H*(P+1) + H + 1$$ parameters. The coefficients and intercepts of the $$H$$ hidden units have $$H * (P + 1)$$ parameters, and the final linear relationship has an additional $$H$$ coefficients plus the intercept.

To minimize the sum of squared residuals, the parameters are usually initialized to random values and optimized. However, It is common that the calculated solution is not a global solution. Neural networks also have a tendency to overfit due to a large number of regression coefficients.

**Weight decay** addresses overfitting by adding a penalty for large regression coefficients. They must significantly decrease the error in order to be added to the model. Formally, we add a constraint so that the sum of squared errors of the model for a given $$\lambda$$ minimizes:

 $$\sum_{i = 1}^n (y_i - f_i(x))^2 + \lambda \sum_{k = 1}^ H \sum_{j = 0} ^ P \beta_{jk}^2 + \lambda \sum_{k = 0}^H \gamma_k^2$$

As $$\lambda$$ increases, the model becomes more smooth (higher bias); reasonable values of $$\lambda$$ are between 0 and .1. Be sure to scale and center predictors before using cross validation to select the **two tuning parameters**: the **number of hidden units**, and the **regularization value $$\lambda$$**.

The model structure described above is a **single-layer feed-forward neural network**, the simplest neural network architecture. There are many extensions to this model, including those that have more than one hidden layer, loops going both directions between layers, and Bayesian approaches. For example, a "self-organizing map" takes a similar approach.

<span class="boxheader">Good to know:</span>

- Use cross-validation to select the two tuning parameters of a single-layer feed-forward neural network: the **number of hidden units**, and the **regularization value $$\lambda$$** (between 0 and .1).
- **Center and scale** the predictors, since the regression coefficients are being summed
- Neural networks are **adversely affected by high correlation** among predictors. To address this, we can pre-filter predictors or use feature extraction (PCA) before applying neural networks. 
- **Averaging the results** of neural networks initialized with different starting values can significantly improve performance. Do it if possible!

In R, neural networks have been implemented in the packages ```nnet```, ```neural```, and ```RSNNS```. In ```caret```, neural networks can be trained with the methods ```"nnet"``` (from ```nnet``` package) and ```"avgNNet"``` (with model averaging), among others.

## Multivariate Adaptive Regression Splines (MARS)
---

MARS predicts the outcome by splitting the predictors into piecewise linear models at cut points that achieve the smallest error. 

The **hinge function** that splits the predictors can be written as: $$h(x) \begin{cases} x, x>0 \\ 0, x \leq 0 \end{cases}$$, and a pair of hinge functions is written as $$h(x - a); h(a - x)$$ where $$a$$ is the cut point.


**Algorithm for an Additive MARS Model:**
- consider each data point for each predictor as a cutopint
- choose the cutpoint that achieves the smallest error when fitting piecewise regression
- repeat for additional features until a stopping point
- prune features taht do not contribute significantly to the model equation

**Tuning parameters:**
1. The degree of features added to the model
2. The number of retained terms (choose this with the LOOCV shortcut, defined [here](resampling-methods))

In a **second degree MARS model**, conducts the same search of a single term that improves the model. Then, it searches again for a new cut to couple with each of the original features. In effect, this is adding interaction terms to each hinge function. To illustrate: a single cutpoint splits a predictor into hinge functions $$A$$ and $$B$$, then we find two more cutpoints for hinge functions $$C, D, E,$$ and $$F$$ such that $$A \times C, A \times D, B \times E$$, and $$B \times F$$ minimize error.

**Advantages:**
- automatic feature selection
- interpretability of predictors
- very little pre-processing required; transformations and filtering predictors are not needed, although correlated predictors can complicate interpretation.
- **Variable importance** can be quantified by tracking the reduction in RMSE for each feature. 

In R, MARS models are implemented in the ```earth``` package. In ```caret```, they can be trained with the ```"earth"``` method.


## Support Vector Machines
---
*[Note: I wrote a more comprehensive overview of SVM for classification* [*here.*](support-vector-machines)*]*


We use a technique called $$\epsilon$$-insensitive regression to define SVMs in the framework of robust regression (minimizing the effect of outliers). The SVM regression coefficients **minimize the cost function**, given an $$\epsilon$$-insensitive function $$L_\epsilon$$ and cost penalty tuning parameter set by the user:

$$\text{Cost Penalty} * \sum_{i = 1}^n L_\epsilon (y_i - \hat y_i) + \sum_{j = 1}^p \beta_j^2$$

Data points within the threshold (cost) do not contribute to the regression fit, while others contribute a linear scale amount. This sounds unintuitive, but it works -- a visualization of the residuals versus its contribution to the regression line is shown below.[^2]

[^2]: Source: Kuhn and Johnson, *Applied Predictive Modeling* (2013), pg. 154

<center>{% include image.html path="documentation/05-22-loss2.png"
                      path-detail="documentation/05-22-loss2.png"
                      alt="loss2" %}</center>


For a new sample $$u$$, the parameter estimates $$\beta_1 ... \beta_n$$ can be written as functions of a set of unknown parameters $$\alpha_i$$ and the training data points, so that the SVM prediction equation is:

$$\hat y = \beta_0 + \sum_{i = 1}^n \alpha_i \biggl(\sum_{j = 1}^p x_{ij} u_j \biggr)$$

Note that $$\alpha_i = 0$$ where the data is within $$\pm \epsilon$$ of the regression line. Only the subset of points where $$\alpha \neq 0$$ are needed for prediction. Those points are called **support vectors**.

The SVM formula can be generalized to:

$$f(u) = \beta_0 + \sum_{i = 1}^n \alpha_i K(x_i, u)$$ 

In this formula, $$K(x_i, u)$$ is the kernel. Popular kernels include linear, polynomial, radial basis, hyperbolic tangent. Radial kernels are very effective, with an additional $$\sigma$$ parameter that controls scale. 

$$\begin{align}
\text{linear} & =  \sum_{j = 1}^p x_{ij} u_j = \mathbf{x'_i u} \\
\text{polynomial} & = (\phi (\mathbf{x'u}) + 1)^{\text{degree}} \\
\text{radial basis function} & =  \exp {(- \sigma \lVert \mathbf{x - u} \rVert ^ 2)}\\
\text{hyperbolic tangent} & =  \tanh (\sigma (\mathbf{x'u}) + 1) \\
\end{align} $$

<span class="boxheader">Good to know:</span>
- A large cost means the model is more flexible (high variance, low bias). 
- Tuning parameters are: **cost** (for example, between $$2^{-2} to 2^{10}$$), and other kernel-specific parameters like scale and degree.
- It is possible to tune over $$\epsilon$$, but the general recommendation is to fix $$\epsilon$$ and tune over other parameters. 
- Make sure to **center and scale predictors**, since SVM uses sum of cross products.

In R, there are a number of implementations of SVMs including the ```e1071``` and ```kernlab``` (more comprehensive for regression) packages. In ```caret```, SVMs can be trained with the methods ```"svmRadial"``` (radial basis function), ```"svmPoly"``` (polynomial SVM), and ```"svmLinear"```.


## K-Nearest Neighbors
---

KNN predicts new observations from the K closest samples to that observation. The distance between samples is usually calculated as Euclidian distance:

$$\sqrt {\sum_{j = 1}^p (x_{aj} - x_{bj}) ^ 2}$$

A generalization of Euclidian is Minkowski distance. Note that when $$q = 2$$, the Mindowski distance is the same as Euclidian distance.

$$ \biggl( \sum \vert x_{aj} - x_{bj} \vert ^ q \biggr) ^ {\frac{1}{q}}$$ 

<span class="boxheader">Good to know:</span>

- **Center and scale predictors**, since KNN measures "distance"
- Missing values are problematic (try imputation if possible)
- Choose the tuning parameter $$K$$ with resampling (smaller values leads to more flexible fits and higher variance)
- Computationally not very efficient, since distances between the new sample and all other samples must be computed
- Remove irrelevant or noisy predictors, since they lead to poor performance with KNN
- Improve predictions by weighting neighbors contribution based on distance

In R, KNNs can be trained by ```caret``` with the ```"knn"``` method.  

---
---
---
# III. Regression Trees and Rule-Based Models
---
---
---
Tree-based models are popular because they generate a set of conditions that are interpretable and easy to implement, handle many types of data without pre-processing, do not require the user to specify the form of the predictors' relationship to the response, effectively handle missing data, and implicitly conduct feature selection. 

However, trees have the weaknesses of model instability, where slight changes in the data can drastically change the structure of the tree, and single trees have poor performance. 

## Basic Regressions Trees
---

Basic regression trees partition the data into smaller groups that are most homogenous with respect to the response. The **CART (classification and regression tree) methodology** for regression begins with the entire data set, $$S$$, and searches every value of every predictor to find the predictor and split value that partitions the data into two groups such that the overall sum of squares error are minimized. 

$$ SSE = \sum_{i \in S_1} (y_i - \bar y_1)^2 + \sum_{i \in S_2} (y_i - \bar y_2)^2 $$

This method is repeated to grow the tree, and is called **recursive partitioning**. 

After the tree is grown, it is pruned to prevent over-fitting with a process called cost-complexity tuning, penalizing the error rate using the size of the tree with a complexity parameter $$c_p$$:

$$ SSE_{c_p} = SSE + c_p \times (\# \text{Terminal Nodes})$$

We find the best pruned tree by evaluating the data across a sequence of $$c_p$$ values with cross-validation, and choosing the tree with either the lowest RMSE or using the one standard-error rule. Predictions are calculated with the average of the training set outcomes in each terminal node.

<span class="boxheader">Good to know:</span>

- CART trees can handle missing data. For each split, alternatives (called surrogate splits) are evaluated whose results are similar to the original splits. It a surrogate split approximates the original split well, then it can be used when the predictor data for the original split is missing.
- Variable importance can be calcaulated by the overall reduction in the optimization criteria for each predictor
- Trees intrinsically conduct feature selection, but the choice of split for highly correlated predictors is somewhat random
- **Disadvantages**: 
    - single trees have sub-optimal predictive performance
    - the number of possible predicted outcomes is determined by the number of terminal nodes
    - individual trees tend to be unstable
    - they suffer from selection bias (predictors with a higher number of distinct values are favored)
    - as the number of missing values increases, the selection of preedictors becomes more biased.

Another approach to the problem is **conditional inference trees**, which uses hypothesis tests to evaluate the difference between the means of each possible split point and computes a p-value that the split leads to a significant improvement. Features of conditional inference trees include:
- Predictors on disparate scales are able to be compared, thanks to the p-value.
- Multiple comparison corrections can be applied to p-values to reduce the bias resulting from a large number of split candidates. Predictors are increasingly penalized as the number of splits increases, reducing bias.
- By default, they are not pruned, but their complexity should still be chosen via resampling techniques.
- There is one tuning parameter: the significance threshold to determine whether additional splits should be created.

In R, the CART methodology is implemented in the ```rpart``` package, and the conditional inference tree framework is implemented in the ```party``` package. Single regression trees can be trained in ```caret``` using the methods ```"rpart"``` (CART tree tuned over complexity), ```"rpart2"``` (CART tree tuned over maximum depth), ```"ctree"``` (conditional inference tree tuned over the minimum criterion that must be met to continue splitting), and ```"ctree2"``` (conditional inference tree tuned over maximum depth).

## Regression Model Trees
---

Simple regression trees use the average of the training set outcomes in each terminal node as the prediction, which often measn that they underpredict samples in the extremes. The **model tree approach, also called M5,** addresses this problem by having its terminal nodes predict the outcome using a linear model.

The initial split is found using an exhaustive search over the predictors and training set samples using the **expected reduction in the node's error rate**:

$$ \text{reduction} = \text{SD}(S) - \sum_{i = 1} ^ P \frac{n_i}{n} \times \text{SD}(S_i) $$

Where $$S$$ denotes the entire set of data split into $$P$$ subsets, $$\text{SD}$$ is the standard deviation, and $$n_i$$ is the number of samples in partition $$i$$. This determines if the total variation in the splits, weighted by sample size, is lower than in the presplit data.

The **tree is grown recursively based on reducing the overall error rate**, and the error associated with each linear model is used in place of $$\text{SD}(S)$$ to determine the reduction in error rate for the next split until there are no further improvements or not enough samples to continue.

Next, **each linear model is simplified using an adjusted error rate** that penalizes models with large numbers of parameters. Terms are dropped from the model as long as the adjusted error rate decreases :

$$\text{Adjusted Error Rate} = \frac{n^* + p}{n^* - p} \sum_{i = 1}^{n^*} \vert y_i - \hat y_i \vert $$

- $$n^*$$ is the number of data points used to build the model
- $$p$$ is the number of parameters

Model trees also **incorporate smoothing to decrease the potential for over-fitting**. As the sample goes down the branches of the tree, the tree generates a new prediction for each node.The predictions are combined using:

$$ \hat y_{(p)} = \frac{n_{(k)} \hat y_{(k)} + c \hat y_{(p)}}{n_{(k)} + c}$$

- $$\hat y_{(k)}$$ is the prediction form the child node
- $$n_{(k)}$$ is the number of training set data points in the child node
- $$\hat y_{(p)}$$ is the prediction from the parent node
- $$c$$ is a constant with a default value of 15.

Smoothing can have a significant positive effect on the model when the models across nodes are very different. This is because (1) the number of available training set samples decreases as new splits are added, and (2) the linear models derived by the splitting process may suffer from significant collinearity.

Once the tree is fully grown, it is **pruned back using the adjusted error rate** (similar to CART trees, see previous section).

In R, the main implementation for model trees is accessed with the ```RWeka``` package, and can be tuned in ```caret``` using the method ```"M5"``` to evaluate model trees, rule-based versions, and the use of smoothing/pruning.

## Rule-Based Models
---

A rule is defined as a distinct path through a tree, and the number of samples it affects is called its coverage. In some cases, rules created by trees may have redundant terms, and it may be advantageous to remove conditions becasue they do not contribute much to the model.

One approach to creating rules from model trees (Holmes et al, 1993) uses the "separate and conquer" strategy, which derives rules from many different model trees:
1. Create an initial model tree (recommend unsmoothed) and save only the rule with the largest coverage. 
2. Remove the samples covered by the first rule from the dataset, create another model tree with the remaining data, and save only the rule with the largest coverage.
3. Repeat until all the traiing set data have been covered by eat least one rule.
4. A new sample is predicted by determining which rule(s) it falls under, then applies the linear model associated with the largest coverage.

In R, rule-based models are also part of the ```RWeka``` package, and can be tuned in ```caret``` using the method ```"M5Rules"```.


## Bagged Trees
---
*[Note: I wrote a more comprehensive overview of bagging* [*here.*](tree-based-methods#iv-method-bagging)*]*

Bagging, or bootstrap aggregation, uses bootstrapping with any regression or classification model to construct an ensemble. Bagging trees consists of generating $$m$$ bootstrap samples of the original data, training an unpruned model on the sample, calculating a prediction for each model for new samples, and averaging all $$m$$ predictions for the final prediction. 

The main choice for bagging is the **number of bootstrap samples to aggregate**. Often, small improvements can be made using bagging ensembles up to size 50, but if performance is not at an acceptable level after ~50 bagging iterations, then try a more powerfully predictive ensemble method such as random forests or boosting.

**Advantages of Bagged Models:**
- Effectively reduces variance of a prediction through the aggregation process. (note: bagging stable, lower variance models like linear regression and MARS offers less improvement in predicive performance).
- They provide their own internal estimate of predictive performance by with predictions for out-of-bag samples.

**Disadvantages of Bagged Models:**
- Computational costs and memory requirements increase with the number of samples, but this can be mitigated with parallel computing since bagging is easily parallelized.
- They are much less interpretable, but measures of variable importance can be constructed by combining measures of importance from individual models across the ensemble.

In R, the ```ipred``` package can create bagged trees. Several other packages have functions for bagging, including ```caret```, which can bag many model types with the ```bag()``` function.

## Random Forests
---
*[Note: I also wrote about random forests* [*here.*](tree-based-methods#v-method-random-forests)*]*

The trees in bagging are not independent of each other, since all predictors are considered at every split. This **tree correlation** in bagging prevents it from optimally reducing variance of the predicted values. Random forests reduce correlation among predictors by adding randomness to the tree construction process.

<span class="boxheader">Algorithm for building random forests:</span>
1. Select the number of models to build (recommendation: at least 1,000), commonly referred to as $$m_{try}$$
2. Generate a bootstrap sample of the original data, and train an unpruned tree on the bootstrap sample. For each tree split, randomly select $$k (< P)$$ of the original predictors as options for partitioning the data.
3. Repeat the previous step for the predetermined number of times. 

<span class="boxheader">Good to know:</span>

- Random forests are more computationally efficient than bagging, since the tree-building can be parallel processed and the tree-building only evaluates a fraction of the original predictors
- Like boosting, random forests can be built with CART trees or conditional inference treees as the naive learner.
- The default tuning parameter for regresion, $$m_{try} = P/3$$, tends to work well for a quick assessment of random forest performance.
- Variable importance can be calculated by the improvement in node purity based on the performance metric for each predictor across the forest. 
    - However, correlations between variables have significant impact on the importance values. Correlations dilute the importance of key predictors, and uninformed predictors can have large importance values if correlated to important variables.
    - In addition, the $$m_{try}$$ tuning parameter can also have a serious effect on importance values.

In R, the main implementation for random forest comes from the ```randomForest``` package. ```Caret``` can train random forests over $$m_{try}$$ and the number of bootstrap samples with the ```"rf"``` (CART trees) or ```"cforest"``` (conditional inference trees) methods.

## Boosting
---
*[Note: I wrote a more comprehensive overview of boosting* [*here.*](tree-based-methods#vi-method-boosting)*]*

Given a loss function (squared error) and a weak learner (regression trees), find an additive model that minimizes the loss function. The algorithm is initialized with the best guess of the response (mean of repsonse in regression), the gradient (residual) is calculated, and a model is fit to the residuals to minimize the loss function. The current model is added to the previous model, and the procedure continues for a specified number of iterations.

Trees are an excellent base learner because their depth is flexible, they're easily added togehter, and can be generated very quickly. The two tuning parameters of simple gradient boosting are **tree depth (interaction depth)** and the **number of iterations**.

Unlike random forests, which creates independent trees with maximum depth that contribute equally to the final model, the boosting creates trees dependent on past trees with minimum depth that contribute unequally to the final model. The computational time for boosting is often greater than for random forests, since random forests can be easily parallel processed.

The **learning rate**, $$\lambda$$, addresses the greediness of boosting by employing regularization (shrinkage) and adding only a fraction of the current predicted value to the previous iteration's predicted value. 
- $$\lambda$$ takes a value between 0 and 1.
- Small values of $$\lambda$$ ($$<0.01$$) work best, but the value of the parameter is inversely proportional to the computation time required because more iterations are necessary, and more memory is required for storing the model.

In stochastic gradient boosting, a **fraction of the data is randomly selected** for each tree-building iteration. This improves accuracy while reducing computational resources. A value of around $$0.5$$ is recommended, but the fraction can be tuned like any other parameter.

**Variable importance** is a function of the reduction in squared error. The importance in squared error due to each predictor is summed within each tree in the ensemble, and averaged across the entire ensemble to yield an overall importance value. Compared to random forests, the importance profile for boosting will often have a much steeper importance slope because the trees from boosting are dependent on each other and have correlated structures. Many of the same predictors will be selected across trees.

In R, the ```gbm``` package implements stochastic gradient boosting. In ```caret```, stochastic graient boosting can be trained with the ```"gbm"``` method, and tuned over the number of trees, interaction depth, shrinkage, and proportion of observations to be sampled.

## Cubist
---

Cubist is a complicated rule-based model that has several changes compared to model trees and rule-based trees:

- different techniques for linear model smoothing, creating rules, and pruning
- optional boosting-like procedure called *committees*
- generated predictions can be adjusted using nearby points from the training set data

I'm not going to completely summarize this model, so read Chapter 8.7 of *Applied Predictive Modeling* and check out [www.RuleQuest.com](www.RuleQuest.com) for more details. 

<span class="boxheader">The Algorithm:</span>

1. **Build an [M5 model tree](apm-regression#regression-model-trees)** with the usual method, splitting predictors by reducing the overall error rate, simplifying with the adjusted error rate, and incorporate smoothing.
    - The smoothing process is a linear combination of two models
    - The smoothing coefficient is determined by the variance and covariance between the two models' residuals, giving the model with the smallest RMSE a higher weight.
2. The final model is used to construct the initial set of rules. The **sequence of linear models at each node is collected into a single, smoothed representation** of the models.
    - The adjusted error rate is used to prune or combine rules.
    - If the adjusted error rate does not increase from the deletion of a condition or rule, that condition or rule is dropped.
3. **Model committees are created by generating a sequence of rule-based models.** The training set outcome is adjusted based on the prior model fit, and then builds a new set of rules using the pseudo-response.
    - Underpredicted points will have increased sample values in hopes that the model will produce a larger prediction in the next iteration.
    - Overpredicted points will have decreased sample values in hopes that the model will produce a smaller prediction in the next iteration.
4. Once the full set of model committees are created, **new samples are predicted using each model and averaged** for the final rule-based prediction.
5. After the rule-based model is finalized, Cubist can **adjust the prediction with the $$K$$ most similar neighbors** from the training set weighted by distance. 
    - Cubist uses Manhattan (city-block) distances to determine nearest neighbors.
    - Neighbors are only included if they are "close enough" to the prediction sample.
    - Custom weights are also computed based on the distance to each neighbor.


**Tuning parameters**:
- The # of committees (try checking 0 to 100)
- The # of neighbors (checking 0 to 9)

There is no established method for measuring variable importance in Cubist models.

In R, the ```Cubist``` package has an implementation of the model. In ```caret```, Cubist can be tuned over the number of committees and neighbors with the ```"cubist"``` method.
