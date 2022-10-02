---
layout: post
title: "Moving Beyond Linearity"
description: "Linear models can have significant limitations in terms of predictive power, because the limear assumption can be a poor assumption. Methods such as polynomial regression, step functions, regression and smoothing splines, local regression, and generalized additive models can help us flexibly model non-linear relationships."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents 

- [I. Basis Functions](#i-basis-functions)
- [II. Regression Splines](#ii-regression-splines)
- [III. Smoothing Splines](#iii-smoothing-splines)
- [IV. Local Regression](#iv-local-regression)
- [V. Generalized Additive Models](#v-generalized-additive-models)
- [VI. Applications in R](#vi-applications-in-r)

</div>


### I. Basis Functions

Basis functions are a family of functions of transformations, $$ b_1 (X), ... , b_k (X) $$, applied to $$X$$ to and take the form:

$$ y_i = \beta_0 + \beta_1 b_1(x_i) + \beta_2 b_1(x_i) + ... + \beta_k b_k(x_i) + \epsilon_i $$

Two very common types of basis functions are **polynomial regression**, with $$b_j(x_i) = x_i^j$$, and **piecewise constant step functions**, with $$b_j(x_i) = I(c_j \leq x_i < c_{j+1})$$.

<span class = "boxheader">Polynomial Regression</span>

$$ y_i = \beta_0 + \beta_1 x_i + \beta_2 x_i^2 + ... + \beta_d x_i^d + \epsilon_i $$

This is a simple way to extend the linearity assumption. However, it's unusual to use powers of greater than 3 or 4 due to weirdness at the boundaries.

Note that the variance, $$Var(\hat f (x_0)) $$ can be computed with the variance estimates for each of the coefficients $$\hat \beta_j$$ and the covariances between pairs of the coefficient estimates.[^1]

[^1]: For the degree 4 case, $$Var[\hat f(x_0)] = l_0^T \hat C l_0$$, where $$\hat C$$ is the 5x5 covariance matrix of the $$\hat \beta_j$$ and $$l_0^T = (1, x_0, x_0^2, x_0^3, x_0^4) $$

<span class = "boxheader">Step Functions</span>

Step functions break $$X$$ into bins, so it becomes an ordered categorical variable. To create a step function, create cutpoints $$c_1, c_2, ..., c_k$$, then create new variables:
- $$C_0(x) = I(X < c_1)$$ ;
- $$C_1(x) = I(c_1 \leq X < c_2)$$ ;
- ...
- $$C_1(x) = I(c_k \leq X)$$ ;

$$I()$$ is an indicator function that returns a 1 if the condition is true, and 0 otherwise. We use least squares to fit a linear model using each of these new variables as predictors:

$$ y_i = \beta_0 + \beta_1 C_1 (x_i) + \beta_2 C_2 (x_i) + ... + \beta_K C_K (x_i) + \epsilon_i $$ 

These functions are popular in biostatistics and epidemiology, but one downside is that it can miss important action in the data unless there are natural breakpoints.

---
---
---
### II. Regression Splines

Regression splines are also a class of basis functions that extend upon polynomial regression and piecewise constant regression methods in the previous section, but I'm giving them their own section because they are more complicated and deserve some more space!

Splines of degree $$d$$ are **piecewise degree-$$d$$ polynomials** with **continuity in derivatives up to degree $$d - 1$$ at each knot** to ensure continuity (knots are the breakpoints where the polynomial coefficients change).

A cubic spline with $$k$$ knots uses a total of $$4+k$$ degrees of freedom, with continuity of the first two derivatives. This is because piecewise cubic functions have a total of $$k+1$$ sections, each section adds 4 degrees of freedom (one for each coefficient), and each knot has 3 contraints (continuity of the function, its 1st derivative, and its 2nd derivative).

**Example: A Cubic Spline Representation**

$$ y_i = \beta_0 + \beta_1 x_i + \beta_2 x_i^2 + \beta_3 x_i^3 + \beta_4 h(x_i, \xi_1) + \beta_5 h(x_i, \xi_2)  + \epsilon_i $$

This is a representation of a cubic spline with two knots, $$\xi_1$$ and $$\xi_2$$. We add one truncated power basis per knot to guarantee our continuity constraints,[^2] for a total of $$k+3$$ basis functions. In this case, 2 knots + 3 degrees of the polynomial = 5 total basis functions.

The truncated power basis is defined as: $$h(x, \xi) = (x - \xi)^3_{+} = 
\begin{cases} (x - \xi)^3 & \text{if } x > \xi \\
0 & \text{otherwise} 
\end{cases} $$

In other words, fitting a cubic spline means that we're performing least squares regression with the predictors: $$ \text{ intercept, } X, X^2, X^3, h(x, \xi_1), ..., h(x, \xi_k)$$. Note that there are a total of $$k+4$$ coefficients.

A **natural spline** is a regression spline that is required to be linear at the boundary, which provides more stable estimates at the boundaries. The additional boundary constraints lead to relatively narrower confidence intervals at the boundaries.

<span class="boxheader">Important Information about Regression Splines:</span>
- In practice, it's common to place knots uniformly along the data
- Use cross-validation to choose the number of knots, $$k$$ based on the value that minimizes RSS.
- In general, regression splines give superior results to polynomial regression, and also gives more stable estimates at boundaries.

[^2]: It is possible to prove that adding a term of the form $$\beta_4 h(x, \xi)$$ to the model for a cubic polynomial will lead to a discontinuity in only the third derivative at $$\xi$$, and the function will remain continuous, with conitnuous first and second derivatives, at each of the knots. 

---
---
---
### III. Smoothing Splines

In minimizing RSS, we want to find some function $$g(x)$$ that fits the observed data well, but we don't want it to interpolate all of the data, since it overfits the data by being too flexible. Instead, we want to find $$g(x)$$ that makes RSS small, but is also sufficiently smooth.

We ensure that $$g(x)$$ is smooth by minimizing the loss function:

$$ \sum_{i = 1}^n (y_i - g(x_i))^2 + \lambda \int g''(t)^2 dt $$

Where $$\lambda$$ is a non-negative tuning parameter that controls the bias-variance trade-off. If $$\lambda = 0$$, then the smoothing spline will be a 100% fit to the data, and as $$\lambda \to \infty$$, the model will be a straight line.

This takes on the "Loss + Penalty" form that we also see in ridge regression and lasso. The second half of the equation above is a penalty term that penalizes variability in the model, since the second derivative $$g''(t)$$ measures the "roughness" of the model, and is large if $$g$$ is wiggly.

<span class="boxheader">Special Properties of the Smoothing Spline</span>

The smoothing spline is a **natural cubic spline**. Compared to the basis function approach of a regression spline, the smoothing spline is a **shrunken version**, where $$\lambda$$ controls the levels of shrinkage.

- it is a piecewise cubic polynomial with continuous 1st and 2nd derivatives
- knots at the unique values of $$x_1, ..., x_n$$
- linear in regions outside of the extreme knots

**Choosing the tuning parameter $$\lambda$$:** As $$\lambda$$ increases from $$0$$ to $$\infty$$, the effective degrees of freedom $$\text{df}_\lambda$$ decreases from $$n$$ to $$2$$. We use effective degrees of freedom since although a smoothing spline has $$n$$ parameters and hence $$n$$ nominal degrees of freedom, those parameters are heavily constrained or shrunk down. $$\text{df}_\lambda$$ is a better measure of the flexbility of a smoothing spline.

We define the effective degrees of freedom as $$\text{df}_\lambda = \sum_{i = 1}^n \{ S_\lambda \}_{ii}$$ ; or the sum of the diagonal elements of $$S_\lambda$$ where $$S_\lambda^{n x n} * y = \hat g_\lambda$$, the solution to a particular $$\lambda$$.

Leave-one-out cross validation is very efficient (the same cost as computing a single fit) for choosing $$\lambda$$:

$$\text{RSS}_{CV(\lambda)} = \sum_{i = i}^n \bigl( (y_i - \hat g_\lambda^{(-i)} (x_i) \bigr) ^ 2 = \sum_{i = 1}^n \Biggl[ \frac {y_i - \hat g_{\lambda} (x_i)}  {1 - \{ S_\lambda \}_{ii}} \Biggr] ^ 2 $$

Where $$\hat g_\lambda^{(-i)} (x_i)$$ indicates the fitted value for this smoothing spline evaluated at $$x_i$$, where the fit uses all of the training observations except for the $$i$$th observation $$(x_i, y_i)$$. Note that we can compute all of the LOOCV fits using only the original fit to all of the data!

---
---
---
### IV. Local Regression

For local regression, we compute the fit at a target point $$x_0$$ using only the nearby training observations. 

**Algorithm:**
1. Gather the fraction $$s = \frac{k}{n} $$ of training points whose $$x_i$$ are closest to $$x_0$$
2. Assign a weight $$K_{i0} = K(x_i, x_0) $$ to each point in the neighborhood where the closest has the highest weight, the farthest has 0 weight, and all other points have 0 weight.
3. Fit a weighted least squares of $$y$$ onto $$x_i$$: $$\min_{\hat \beta_0, \hat \beta_1} \sum_{i = 1}^n K_{i0} (y_0 - \beta_0 - \beta_1 x_i) ^ 2 $$
4. The fitted value at $$x_0$$ is $$\hat f (x_0) = \hat \beta_0 + \hat \beta_1 x_0 $$

**Notes:**
- This is a memory-based procedure, which means it needs all training data for each calculation of a prediction.
- The span, $$s$$, is the tuning parameter that we choose based on cross-validation. A small span means the fit is more local and wiggly with higher bias/lower variance, while a large span will lead to a more global fit with lower bias/higher variance.
- Local regression can be used for 1, 2, or 3 variables, but is bad when there are more than 3 or 4 variables due to the curse of dimensionality.
- This can also be generalized to a multiple linear regression with some global and some local variables. These are called *varying coefficient models*.

---
---
---
### V. Generalized Additive Models

Generalized additive models (GAMs) provide a general framework for extending a standard linear model by allowing non-linear functinos of each of the variables while maintaining additivity.

**Pros:**
- Allows fitting a non-linear $$f_j$$ to each $$X_j$$ to model non-linear relationships. This means we don't have to try transformations on every variable
- Can make more accurate predictions of $$Y$$
- Can examine the effect of each $$X_j$$ on $$Y$$ individually, which is good for inference and interpretability
- Smoothness of $$f_j$$ for variable $$X_j$$ can be summarized with the degrees of freedom.

**Cons:**
- Models are restricted to being additive, so important interactions can be missed. However, we can add interactions terms or functions manually to fix this problem.

For **regression problems**, models can combine natural splines, smoothing splines, local regression, polynomial regression, etc. This is a compromise between linear models and fully nonparametric models, and can be represented by:

$$\begin{align}
y & = \beta_0 + \sum_{j = 1}^p f_j (x_{ij}) + \epsilon_i \\
& = \beta_0 + f_1 (x_{i1}) + f_2 (x_{i2}) + ... + f_p (x_{ip}) + \epsilon_i 
\end{align}$$

For **classification problems**, logistic GAMs can be represented by:

$$  \log \Biggl( \frac{p(x)} {1 - p(x)} \Biggr) = \beta_0 + f_1(x) + f_2(x_2) + ... + f_p (x_p) $$

We can use ANOVA to test if certain coefficients in generalized additive models are significant.

---
---
---
### VI. Applications in R

{% highlight R %}
library("ISLR")
library("splines")
library("gam")
library("tidyverse")
{% endhighlight R %}

{% highlight R %}
# polynomial regression - compare degrees
fit_poly1 <- lm(wage ~ education + age,data=Wage)
fit_poly2 <- lm(wage ~ education + poly(age,2),data=Wage)
fit_poly3 <- lm(wage ~ education + poly(age,3),data=Wage)
fit_poly4 <- lm(wage ~ education + poly(age,4),data=Wage)
fit_poly5 <- lm(wage ~ education + poly(age,5),data=Wage)
anova(fit_poly1,fit_poly2,fit_poly5,fit_poly5,fit_poly5) # quadratic fit is sufficient

# step function using cut()
fit_step <- lm(wage ~ cut(age, 4), data = Wage)
summary(fit_step)

# regression splines with bs() - generate basis function with specified knots
fit_regspline <- lm(wage ~ bs(age, knots = c(25, 40, 60)), data = Wage) # fit with pre-specified knots
fit_regspline <- lm(wage ~ bs(age, df = 6), data = Wage) # fit with pre-specified df, knots at uniform quantiles

agelims <- range(Wage$age)
age_grid <- seq(from = agelims[1],to = agelims[2])
pred_regspline <- predict(fit_regspline, newdata = list(age = age_grid), se = T)

plot(Wage$age, Wage$wage, col = "gray")
lines(age_grid, pred_spline$fit, lwd = 2)
lines(age_grid, pred_spline$fit + 2 * pred_spline$se, lty = "dashed", col = "red")
lines(age_grid, pred_spline$fit - 2 * pred_spline$se, lty = "dashed", col = "red")

# natural splines with ns()
fit_natspline <- lm(wage ~ ns(age, df = 4), data = Wage) # fit with pre-specified df

pred_natspline <- predict(fit_natspline, newdata = list(age = age_grid), se = T)
lines(age_grid, pred_natspline$fit, col = "blue", lwd = 2)

# smoothing splines with smooth.spline()
fit_smoothspline <- smooth.spline(Wage$age, Wage$wage, df = 16) # fit with pre-specified df 

fit_smoothspline_cv <- smooth.spline(Wage$age, Wage$wage, cv = TRUE) # fit with LOOCV-chosen df
fit_smoothspline_cv$df

# local regression with loess()
fit_loess <- loess(wage ~ age, span = .2, data = Wage)
fit_loess <- loess(wage ~ age, span = .5, data = Wage)

### GAMs

# use lm() to model GAMs that can be written out with basis functions
gam_basic <- lm(wage ~ ns(year, 4) + ns(age, 5) + education, data = Wage) 
plot.Gam(gam_basic, se = TRUE, col = "red")

# use gam() when using components without basis function form. i.e. s() fits smoothing splines
gam_3 <- gam(wage ~ s(year, 4) + s(age, 5) + education, data = Wage)
summary(gam_3)

par(mfrow = c(1, 3))
plot(gam_3, se = TRUE, col = "blue")

# compare GAMs against each other and to a linear fit
gam_1 <- gam(wage ~ s(age, 5) + education, data = Wage)
gam_2 <- gam(wage ~ year + s(age, 5) + education, data = Wage)
anova(gam_1, gam_2, gam_3, test ="F") # gam_2 is preferred

# gam with local regression with lo()
gam_lo1 <- gam(wage ~ s(year, df = 4) + lo(age, span = .7) + education, data = Wage)
gam_lo2 <- gam(wage ~ lo(year, age, span = .5) + education, data = Wage) # local regression with interaction terms

library("akima") # to plot local regression interaction
plot(gam_lo2)

# logsitic regression GAM
gam_logit <- gam(I(wage > 250) ~ year + s(age, df = 5) + education, family = binomial, data = Wage)
par(mfrow = c(1, 3))
plot(gam_logit, se = T, col = "green")

{% endhighlight R %}
