---
layout: post
title: "Data Pre-Processing"
description: "How the predictors are encoded, called feature engineering, has a significant impact on model performance (i.e. predictor combinations, ratios, etc). This post covers unsupervised approaches to data pre-processing."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents

- [I. Data Pre-Processing](#i-data-pre-processing-unsupervised)
- [II. Applications in R](#ii-applications-in-r)

</div>

### I. Data Pre-Processing (Unsupervised)

When given a dataset, it's important to make sure it's in the best format for your chosen model.

- If the data is **skewed** or has **outliers**, consider performing a transformation (i.e. Box Cox, spatial sign)
- If the data has **many predictors**, consider feature extraction/dimension reduction methods (i.e. PCA)
- If the data has **missing values**, consider imputation (i.e. k-nearest neighbors, bagging, mean/median, regression)
- If the data has **near-zero variance predictors**, consider dropping those predictors
- If the data has **multicollinearity**, consider dropping high-correlation variables


#### <span class="boxheader">Data Transformation for Individual Predictors</span>

For many models that are sensitive to scale, magnitude, and balance, it's appropriate to transform individual predictors:

1. **Centering and scaling** generally improve the numerical stability of some calculations and models, but can lead to a loss in interpretability.
2. **Transformations** can resolve skewness in the data, which also leads to more stable models.

**Skewness:** A right-skewed distribution has a tail on the right, and a left-skewed distribution has a tail on the left.
Considered skewed if the ratio of the highest value to the lowest value is greater than 20. Or use the sample skewness statistic, which becomes more positive with right skew and more negative with left skew:

$$ \begin{align}
\text {skewness } & = \frac {\sum (x_i - \bar x) ^ 3} {(n - 1) v^{3/2}} \\
\text {where } v & = \frac {\sum (x_i - \bar x) ^ 2} {(n - 1)} 
\end{align}$$


**The Box Cox family of transformations** can determine the type of transformation for the sample data based on maximum likelihood to estimate $$\lambda$$:

$$ x^* = \begin{cases}
\frac{x^\lambda - 1}{\lambda} & \text{ if } \lambda \neq 0
\log x \text{ if } \lambda = 0
\end{cases}$$

The Box Cox method can identify the log transformation, square ($$\lambda = 2$$), square root ($$\lambda = .5$$), inverse ($$\lambda = -1$$), and others.


#### <span class="boxheader">Data Transformation for Multiple Predictors</span>

**Outliers**
- Make sure that suspected outliers are scientifically valid
- Be careful when changing values especially when sample sizes are small.
- Models that are resistant to outliers include tree-based classification models and support vector machines.
- The **spatial sign transformation** can minimize the outlier problem by projecting the predictor values onto a multidimensional sphere, so that all samples are the same distance from the center of the sphere. Each sample is divided by its squared norm (make sure to center and scale prior to transformation):

$$ x_{ij}^* = \frac{x_{ij}} {\sum_{j = 1}^P x_{ij}^2} $$



**Data Reduction and Feature Extraction** (aka Signal Extraction) 
These methods reduce the data by generating a smaller set of predictors that captures a majority of the information in the original variables.

- PCA captures the most variance possible and creates uncorrelated components. However, predictors should be transformed to fix any skew, then centered and scaled. Can be used in data exploration to see which predictors are associated with each component. 

#### <span class="boxheader">Missing Values</span>

Missing data can be structurally missing, or the data was just not collected. Some tree-based models can specifically account for missing data. However, in other cases, missing data can be imputed.

Note: if resampling to select tuning parameter values, imputation should be incorporated within the resampling.

A popular method is **K-nearest neighbor imputation**:
- Find the samples in the training set "closest" to the missing data point and average those nearby points for an estimate.
- This ensures the imputed data is within the range of the training set values, but the entire training set is required every time a missing value needs to be imputed. 
- The number of neighbors and the method for determining closeness are both tuning parrameters, but this approach is fairly robust to the tuning parameters.

#### <span class="boxheader">Removing Predictors</span>

Advantages to removing predictors include:
- Decreased computational time and complexity
- If highly correlated, removing one predictor should not compromise model performance
- Some models can be crippled by predictors with degenerate distributions

**Near-Zero Variance Predictors** are those that have no or very little information and can easily be discarded. It may be advantageous to remove these variables from the model. The rule of thumb for detecting  near-zero variance predictors is:
- the fraction of unique values over the sample size is low (say 10%)
- the ratio of the frequency of the most prevalent value to the frequency of the second most prevalent value is large (say around 20)

**Between-Predictor Correlations (Collinearity and Multicollinearity)**
- PCA can be used to characterize the magnitude of the problem
- Redundant predictors add more complexity than information to the model
- Can lead to unstable models and performance in techniques like linear regression.
- Diagnose multicollinearity for linear models with the variance inflation factor (VIF)
- Alternatively, deal with the issue by removing the minimum number of predictors to ensure that all pairwise correlations are below a certain threshold:
  1. Calculate the correlation matrix of the predictors
  2. Determine the two predictors A and B associated with the largest absolute pairwise correlation
  3. Determine the average correlation between A and the other variables, and do the same for B
  4. If A has a larger average correlation, remove it; otherwise remove predictor B
  5. Repeat 2-4 until no absolute correlations are above the threshold.

If we wanted a model particularly sensitive to collinearity, we can apply a threshold of .75 and eliminate the minimum number of predictors to achieve all pairwise correlations less than .75.


#### <span class="boxheader">Adding Predictors</span>

Dummy variables encode categorical predictors into different levels. Adding complex combinations of the data to simpler models may be preferred to models that generate complex, nonlinear relationships.

One way to augment the prediction data with complex combinations of the data is calculate calculate the "class centroids" for classification models, and for each predictor, add the distance to each class centroid to the model (Forina et al. 2009).

#### <span class="boxheader">Binning Predictors</span>

There are many issues with manual binning of continuous data.
- Limits the potential of complex relationships between predictor and outcomes
- Loss of precision in predictions when predictors are categorized
- Categorizing predictors can lead to a high rate of false positives
- The perceived gain interpretability gained by manual categorization is usually offset by a significant loss in performance.

Several models, such as classification/regression trees and multivariate adaptive regression splines, estimate cut points mathematically in the process of model building to maximize accuracy. This is different than manual binning.

---
---
---
### II. Applications in R

{% highlight r %}

# preparation --------------------------------------------------------------
library(caret)
library(e1071)
library(corrplot)
library(AppliedPredictiveModeling)
data(segmentationOriginal)

# extract training data, then cellID, class, and case
segData <- subset(segmentationOriginal, Case == "Train")
cellID <- segData$Cell
class <- segData$Class
case <- segData$Case
segData <- segData[, -(1:3)]

# use only non- "Status" columns
segData <- select(segData, str_which(names(segData), "Status", negate = TRUE))

# skewness ----------------------------------------------------------------

skewness(segData$AngleCh1) # one variable 
map_dbl(segData, skewness) # across a data frame

# transformations ---------------------------------------------------------

# box cox transformation
(Ch1AreaTrans <- BoxCoxTrans(segData$AreaCh1)) # one variable
predict(Ch1AreaTrans, head(segData$AreaCh1))

# principle components analysis
pcaObject <- prcomp(segData, center = TRUE, scale = TRUE)
percentVariance <- pcaObject$sd ^ 2 / sum(pcaObject$sd ^ 2) * 100

head(pcaObject$x[, 1:5]) # "x" stores transformed values of data
head(pcaObject$rotation[, 1:3]) # "rotation" stores the variable loadings

# PREPROCESS: box cox transformation, center, scale, and perform PCA
# note, order of transformation is: transform, center, scale, impute, feature extraction, spatial sign
(trans <- preProcess(segData,
                    method = c("BoxCox", "center", "scale", "knnImpute", "pca")))
transformed_data <- predict(trans, segData)


# filter near-zero variance predictors ------------------------------------

nearZeroVar(segData) # returns vector of integers for which columns should be removed


# correlations between predictors -----------------------------------------

correlations <- cor(segData)
correlations[1:4, 1:4]
corrplot(correlations, order = "hclust")

# remove the min # predictors so that all pairwise correlations are below a threshold
highCorr <- findCorrelation(correlations, cutoff = .75)
filteredSegData <- segData[, -highCorr]


# create a full set of dummy variables ------------------------------------
# recommended for tree-based models

# without interaction effect
simpleMod <- dummyVars(~ Petal.Width + Species,
                       data = iris,
                       levelsOnly = TRUE) # remove var name from the column name

predict(simpleMod, head(iris)) # assumes the effect of petal width is same for every species

# with interaction effect
simpleMod_withInteraction <- dummyVars(~ Petal.Width + Species + Petal.Width:Species,
                                       data = iris,
                                       levelsOnly = TRUE) # remove var name from the column name
predict(simpleMod_withInteraction, head(iris)) 


# all together -----------------------------------------------------------

data(BloodBrain) # data = bbbDescr; outcome = logBBB

bbbDescr <- bbbDescr[, -nearZeroVar(bbbDescr)] # remove 7 vars w/near zero variance

corrplot(cor(bbbDescr), order = "hclust")
highCorr <- findCorrelation(cor(bbbDescr), cutoff = .75)
filteredbbbDescr <- bbbDescr[, -highCorr] # remove 65 highly correlated vars

(trans <- preProcess(filteredbbbDescr,
                     method = c("BoxCox", "center", "scale", "pca")))

transformed_data <- predict(trans, filteredbbbDescr) # 30 variables explain 95% of variance

{% endhighlight r %}