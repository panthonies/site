---
layout: post
title: "Linear Regression"
description: "Linear Regression is a simple approach for predicting a quantitative response, and is the foundation of many other types of statistical learning. It is especially convenient for establishing a the strength of relationships between specific variables to an output, and is easily interpretable."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents 

- [I. Simple Linear Regression](#i-simple-linear-regression)
- [II. Multiple Linear Regression](#ii-multiple-linear-regression)
- [III. Considerations and Potential Problems](#iii-considerations-and-potential-problems)
- [IV. Comparison to K-Nearest Neighbors Regression](#iv-comparison-to-k-nearest-neighbors-regression)
- [V. Applications in R](#v-applications-in-r)

</div>


### I. Simple Linear Regression

<span class="boxheader">The Basics</span> -- simple linear regression takes the form:

$$ y = {\beta}_0 + {\beta}_1 x $$ 

To find the line that is closest to the data, we use the least squares method and **minimize the residual sum of squares** (RSS), the amount of unexplained variability in the response after the regression:

$$ RSS = \sum_{i = 1} ^ n (y_i - \hat {y}_i) ^ 2 $$

Solving for the coefficient estimates $$ \hat {\beta}_1 $$ and $$ \hat {\beta}_0 $$, we find that:

$$ \begin{align}
\hat {\beta}_1 & = \frac { \sum_{i = 1}^n (x_i - \bar x) (y_i - \bar y)} {\sum_{i = 1}^n (x_i - \bar x)}\\
\hat {\beta}_0 & = \bar y - \hat {\beta}_1 \bar x
\end{align}
$$

- Note: we can also prove that these estimates are unbiased, i.e. $$ E(\hat {\beta}_0) = \beta_0 $$ and $$ E(\hat {\beta}_1) = \beta_1 $$

<span class="boxheader">Measuring accuracy of the coefficient estimates </span> -- using the general variance formula, $$ Var (\hat u) = SE ( \hat u ) ^ 2 = \frac {\sigma ^ 2} {n} $$, we can solve for the **standard error of the least squares coefficients**:

$$ \begin{align}
SE ( \hat {\beta}_1 ) ^ 2 & = \frac{ \sigma ^ 2} { \sum_{i = 1}^n (x_i - \bar x) ^ 2} \\
SE ( \hat {\beta}_0 ) ^ 2 & = \sigma ^ 2 \Biggl[ \frac {1} {n} + \frac{\bar x ^ 2} {\sum_{i = 1}^n (x_i - \bar x) ^ 2} \Biggr]
\end{align}
$$

- Note: $$ \hat \sigma ^ 2 $$, the **residual standard error** (RSE) is given by $$ \sqrt {RSS / (n - 2)} $$.

These standard errors can be used to compute **confidence intervals**; for example, $$ \hat {\beta}_1 \pm z_{\alpha / 2} * SE(\hat {\beta}_1) $$.  A 95% confidence interval means that if repeated samples were taken and 95% confidence intervals were computed for each sample, 95% of those intervals are expected to contain the population mean.

To perform **hypothesis testing**, we compute a t-statistic, given by $$ \frac{\hat {\beta}_1}{SE(\hat {\beta}_1)} $$. These have an associated p-value, which represents the probability, assuming the null hypothesis is true, that we received the observed sample results.

<span class="boxheader">Measuring accuracy of the model </span>

1. The residual standard error (RSE) is a measure of the lack of fit of the model in units of the response
2. The $$ R^2 $$ statistic is the proportion of variability in Y that can be explained using X, and is calculated with the formula:

$$ R ^ 2 = \frac{TSS - RSS}{TSS} = 1 - \frac{RSS}{TSS} $$

where the total sum of squares (TSS), given by $$ \sum(y_i - \bar y) ^ 2 $$, measures the total variance in the response before the regression.

- Note: The $$ R^2 $$ statistic is equal to the correlation between X and Y in simple linear regression with one predictor.

---
---
---
### II. Multiple Linear Regression

<span class="boxheader">The Basics</span> -- multiple linear regression takes the form:

$$ Y = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + ... + \beta_p x_p $$

Again, the least squares solution involves minimizing the residual sum of squares (RSS).

<span class="boxheader">Is there a relationship between the response and predictors? </span>

We can use a hypothesis test to investigate whether all of the regression coefficients are zero using an F-statistic:

$$ \begin{align}
H_0 & : \beta_1 = \beta_2 = ... = \beta_p = 0 \\ 
F & = \frac{(TSS - RSS) / p}{RSS / (n - p - 1)}
\end{align}
$$
 
If linear model assumptions are correct, then $$ E[RSS / (n - p - 1)] = \sigma ^ 2 $$, and if there is no relationship between the response and predictors, then $$ E[(TSS - RSS) / p] = \sigma ^ 2 $$. Therefore, when there is no relationship between the response and predictors, F-statistic will take on a value close to 1, but when there is a relationship, the F-statistic will be greater than 1.

- Note: The smaller the sample size n, the larger the F-statistic needs to be to reject the null hypothesis.

On the other hand, if you want to test if a particular subset of *q* coefficients are zero, then fit a second model that uses all of the variables except the last *q*, and note the residual sum of squares for that model, $$ RSS_0 $$. Then the null hypothesis and F-statistic are:

$$\begin{align}
H_0 & : \beta_{p - q + 1} = \beta_{p - q + 2} = ... = \beta_p = 0 \\
F & = \frac{(RSS_0 - RSS) / q}{RSS / (n - p - 1)}
\end{align}$$

<span class="boxheader">How do we decide on important variables? </span>

Variables can be selected through three classical approaches:
- Forward selection
	- Start with the null model, fit new models with each of the *p* predictors, then add the variable that resulted in the lowest RSS. Re-fit the base model, fit new models with remaining predictors, and repeat until a stopping condition.
- Backward selection
	- Start with all variables in the model, and remove the variable with the largest p-value. Re-fit the model, and repeat until a stopping condition.
	- Cannot be used if $$ p > n $$
- Mixed selection
	- Start with no variables in the model. Add variables that provide the best fit, but remove variables if the p-value for any variable rises above a certain threshold. 

<span class="boxheader">How well does the model fit the data? </span>

1. For multiple linear regression, $$ R^2 = Cor (Y, \hat Y) ^ 2 $$ . Note that $$ R^2 $$ will always increase for the training data set when adding more variables, so be sure to test the model on a testing data set.
2. Residual standard error is defined generally as: $$ \sqrt { \frac {RSS} {n - p - 1}} $$

The quality of a model can be judged by:
- Mallow's $$ C_p $$
- AIC (Akaike information Criterian)
- BIC (Bayesian information criterion)
- Adjusted $$ R^2 $$

To compare two models, we can perform ANOVA (analysis of variance) using an F-test in order to test the null hypothesis that a model $$M_1$$ is sufficient to explain the data against the alternative hypothesis that a more complex model $$M_2$$ is required. These must be nested models: the predictors in $$M_1$$ must be a subset of the predictors in $$M_2$$.

<span class="boxheader">How accurate are our model predictions?</span>

There are three types of uncertainty associated with prediction:

1. The least squares plane, $$ \hat Y = \hat \beta_0 + \hat \beta_1 x_1 + \hat \beta_2 x_2 + ... + \hat \beta_p x_p $$,  is only an estimate for the true population regression plane, $$ Y = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + ... + \beta_p x_p $$
. This inaccuracy in the coefficient estimates is related to the **reducible error** of the model, and we can estimate this uncertainty with a **confidence interval.**

2. Another source of **reducible error** comes from the **model bias**, since we are estimating the best linear approximation to the true surface, which may not be linear.

3. Even if we knew $$ f(X) $$, the response value can't be predicted perfectly because of the random error, $$ \epsilon $$ in the model. This is the **irreducible error**. We can estimate how much $$ Y $$ will vary from $$ \hat Y $$ with **prediction intervals**, which are always wider than confidence intervals because they incorporate both the error in the estimate for $$ f(X) $$ and the uncertainty as to how much an individual point will differ from the population regression plane.

---
---
---
### III. Considerations and Potential Problems

<span class="boxheader">Considerations</span>

- Qualitative Predictors
	- For predictors with two values/levels, we use dummy variable that takes the value of either 1 or 0. Note that the assigned values will affect interpretation.
	- For predictors with more than two values, we can create additional dummy variables (a total of 1 less than the number of levels).
	- We can also code qualitative variables in other ways to measure particular contrasts between variables, which will lead to equivalent models with different coefficients and interpretations.

- Extensions of the Linear Model
	- Linear regression assumes that the relationship between response and predictor is additive and linear.
		- **Non-additive relationships** can be representated using an **interaction effect** in addition to the main effects. Be sure to abide by the hierarchical principle -- if using an interaction term, always include the main effects as well since $$ X_1 * X_2 $$ is often correlated with $$ X_1 $$ and $$ X_2 $$.
		- **Non-linear relationships** can be represented using **polynomial regression**, among other methods.

<span class="boxheader">Potential Problems</span>

1. Non-linearity of the repsonse-predictor relationships
	- check the **residual plots** of $$e_i = y_i - \hat y_i $$ against $$ x_i $$ to make sure the errors are approximately random
	- if residuals exhibit any trends, then try using non-linear transformations of predictors, such as $$ \log x, \sqrt x, x^2 $$
2. Correlation of error terms
	- $$ \epsilon_i $$ should provide no information about $$ \epsilon_{i + 1} $$, otherwise the sample standard error will be an underestimate of the true standard error, a confidence interval will not be wide enough, and p-values will be lower than they should be.
	- in time series data, **residuals plotted against time** may exhibit tracking if adjacent residuals are similar; there are many methods (not shown here) that can be used to address this
	- outside of time-series data, good experimental design can mitigate the risk of correlated errors
3. Non-constant variance of error terms
	- heteroskedasticity, or a non-constant $$ \sigma^2 $$, looks like a **funnel-shape in residual plots**
	- potential solution: try transforming the response variable with $$ \log Y $$ or $$ \sqrt Y $$
	- potential solution: weighted least squares can solve this; for example, if the $$i$$th response is an average of $$ n_i$$ raw observations that are uncorrelated with their variance, then their average has variance $$ \sigma_i^2 = \sigma^2/n_i $$ and the non-constant variance is solved by using weighted least squares with observation weights $$ w_i = n_i $$
4. Outliers
	- outliers that can affect the model, standard error, confidence intervals, and $$ R^2 $$
	- check for outliers with **studentized residual plots**, where residuals are scaled by the standard error $$ \bigl( \frac {e_i}{\text{Std Err}} \bigr) $$. If studentized residuals are greater than 3, then the observations are potential outliers 
5. High-leverage Points
	- points with an unusual x-value are high-leverage points that can adversely affect the model
	- leverage can be quantified with the **leverage statistic**[^1]; this is always between $$ \frac {1} {n} $$ and 1, and the average is $$ \frac {p + 1} {n} $$
	- if a given observation greatly exceeds $$ (p + 1) / n $$, then the corresponding point may have high leverage
6. Collinearity
	- when two or more predictors are highly correlated, it reduces the accuracy of estimates -- the standard error of $$ \beta_j $$ increases, t-statistic decreases, and the power (chance of correctly determining a non-zero coefficient) of the hypothesis test goes down
	- potential solutions include dropping or combining collinear variables
	- use a correlation matrix to detect collinearity between two variables
	- use the **variance inflation factor (VIF)**, shown below, to assess multicollinearity
		- $$ R_{x_j \vert x_{-j}}^2 $$ is the $$ R^2 $$ of the regression of $$ X_j $$ onto all other predictors. 
		- Note that the smallest value of 1 indicates no collinearity; if it exceeds 5 or 10, then it may be a problem.


$$ \begin{align}
VIF & = \frac {Var(\hat \beta_j, \text{full model})} {\hat \beta_j, \text{fit on its own}}
    & = \frac {1} {1 - R_{x_j \vert x_{-j}}^2}
\end{align} $$


---
---
---
### IV. Comparison to K-Nearest Neighbors Regression

K-Nearest-Neighbors regression takes the form:

$$ \hat {f}(x_o) = \frac {1}{k} \sum_{x_i \in N_0} y_i $$ 

In other words, given a value for K and a prediction point $$x_0$$, KNN regression identifies the $$K$$ training observations that are closest to $$x_0$$, represented by $$N$$, and estimates $$ f(x_o) $$ using the average of all the training responses in $$N_0$$.

- The optimal K depends on the variance-bias tradeoff; a bigger K leads to a smoother, less flexible fit with low variance and high bias while a smaller K leads to a bumpier fit with high variance and low bias.
- A parametric approach will outperform a non-parametric form if it is close to the true form of $$ f $$, or when there is a small number of observations per predictor.
- KNN Regression suffers from the curse of dimensionality:
	- more predictors (dimensions) means neighbors are farther from each other
	- KNN performs poorly with lots of predictors
	- in higher dimensions, KNN often performs worse than linear regression


---
---
---
### V. Applications in R

{% highlight R %}
library("MASS") # lots of functions and data sets
{% endhighlight R %}


{% highlight R %}
model <- lm(y ~ x, data)

confint(model, level = .95) # 95% confidence interval

predict(model) # predicted values
plot(predict(model), residuals(model)) # plot residuals
plot(predict(model), rstudent(model)) # plot studentized residuals

plot(hatvalues(model)) # calculate and plot leverage statistics
which.max(hatvalues(model)) # display the index of the observation with largest leverage

vif(model) # calculates variance inflation factors

anova(model1, model2) # compare two nested models
contrasts(variable) displays information about the levels of a factor
{% endhighlight R %}

[^1]: For simple linear regression, the leverage statistic is given by $$ h_i = \frac {1} {n} + \frac{(x_i - \bar x) ^ 2} {\sum_{i = 1}^n (x_{i'}) - \bar x)} $$.