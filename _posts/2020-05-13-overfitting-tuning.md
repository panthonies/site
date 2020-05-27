---
layout: post
title: "Model Tuning and Overfitting"
description: "An overview of the model tuning process, including data splitting, resampling techniques, and recommendations for choosing parameters and models."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents

- [I. Model Tuning](#i-model-tuning)
- [II. Recommendations](#ii-recommendations)
- [III. Applications in R](#iii-applications-in-r)

</div>

<small>Note: Because I covered similar material in my [Resampling Methods](resampling-methods) post, I won't be going into great detail about the techniques described here.</small>

### I. Model Tuning

Behold! A nice picture of the parameter tuning process.[^1]

[^1]: Source Kuhn and Johnson, *Applied Predictive Modeling (2013)*, pg. 66


<center>{% include image.html path="documentation/05-13-resampling.png"
                      path-detail="documentation/05-13-resampling.png"
                      alt="Model Tuning Overview" %}</center>

**Overfitting** is when the model has learned the characteristics of a particular sample's unique noise, and will have poor accuracy when predicting a new sample. Splitting the data into test and training sets or using resampling techniques to measure error mitigates the effect of overfitting.

<span class="boxheader">Data Splitting</span>

When $$n$$ is not large, a strong case can be made for using cross-validation instead of splitting the data, which will allow the model to learn on more data points. Types of data splitting include:

- A simple random sample of the data
- Stratified random sampling of the data, for balanced outcomes in the test dataset
- Maximum dissimilarity sampling: initialize the test set with a single sample, and allocate the most dissimilar unallocated sample (must specify a dissimilarity measure). Repeat until the target test set size is achieved. 

<span class="boxheader">Resampling Techniques</span>

- K-fold cross-validation
    - In practice, $$K=5$$ or $$K=10$$ are commonly used.
    - A bigger $$K$$ leads to lower bias and higher variance.
    - Generally has high variance compared to other validation methods. 
    - Leave-one-out cross-validation is a special case where $$K = n$$.

- Repeated Training/Test Splits
    - AKA, "Monte Carlo Cross-Validation" or "Leave-group-out Cross-Validation"
    - Rule of thumb is to use around 75-80% of the data for training splits.
    - Use a large number of repetitions (say, 50 to 200+)

- Bootstrap error
    - Fit the model on a bootstrap sampling of the data, then predict the out-of-bag data points for an error estimate
    - In general, less uncertainty than k-fold CV and has similar bias to k-fold CV with $$k \approx 2$$
    - On average, 63.2% of the data points are represented in the bootstrap sample.
    - An alternate "632 method" for estimating error is $$.632 * \text{ Bootstrap Error Rate } + .368 * \text{ Apparent Error Rate}$$

---
---
---
### II. Recommendations

**Choosing the final tuning parameters**:
1. Choose the model associated with the numerically best performance estimates.
2. Choose the simplest model whose performance is within a single standard error of the numerically best value (one-standard-error method).
3. Choose the simplest model that is within a certain tolerance of the numerically best value (% decrease in performance from the numerically optimal value, $$O$$, can be calculated by $$(X - O) / O$$).

**Data Splitting and Resampling**:
- For small datasets, use 10-fold cross validation.
- No resampling method is uniformly better than another.
- For large sample sizes, differences become less pronounced and computational efficiency becomes more important.
- For choosing between models, consider using one of the bootstrap procedures since they have very low variance.

**Choosing between models**:
1. Start with the most flexible models (boosted trees, SVMs) to get a sense of the empirically optimum results.
2. Investigate simpler, more interpretable models (MARS, PLS, GAMs, Naïve Bayes).
3. Consider using the simplest model that reasonably approximates the performance of the more complex models.

- Note: A paired t-test can be used to evaluate the hypothesis that the models have equivalent average accuracies 


---
---
---
### III. Applications in R

{% highlight r %}

# Data Splitting ----------------------------------------------------------

library(AppliedPredictiveModeling)
library(caret)
data(twoClassData) # predictors; classes

# stratified random sampling
set.seed(1)
trainingRows <- createDataPartition(classes, p = .8, list = FALSE)
head(trainingRows)

trainPredictors <- predictors[trainingRows, ]
trainClasses <- classes[trainingRows]
testPredictors <- predictors[-trainingRows, ]
testClasses <- classes[-trainingRows]

# maximum dissimilarity sampling
# maxDissim()


# Resampling --------------------------------------------------------------

library(caret)
set.seed(1)

# repeated training/test splits
repeatedSplits <- createDataPartition(trainClasses, p = .8, times = 3)
str(repeatedSplits)

# indicators for 10-fold CV
cvSplits <- createFolds(trainClasses, k = 10, returnTrain = TRUE)
str(cvSplits)

fold1 <- cvSplits[[1]]
cvPredictors1 <- trainPredictors[fold1, ]
cvClasses1 <- trainClasses[fold1]



# Model Building ----------------------------------------------------------

library(caret)

trainPredictors <- as.matrix(trainPredictors)
(knnFit <- knn3(x = trainPredictors, y = trainClasses, k = 5))

testPredictions <- predict(knnFit, newdata = testPredictors, type = "class")
confusionMatrix(table(testClasses, testPredictions)) # 66% accuracy



# Determining Tuning Parameters -------------------------------------------
# NOTE: the code in this section is an edited version of code from the 'chapters'
# directory of the AppliedPredictiveModeling package that is included  
# for reference/educational purposes only.

library(caret)
data(GermanCredit)

## Assume we have completed some data pre-processing and split the 
## data into GermanCreditTrain and GermanCreditTest training/test sets

library(kernlab)
set.seed(231)
sigDist <- sigest(Class ~ ., data = GermanCreditTrain, frac = 1)
svmTuneGrid <- data.frame(sigma = as.vector(sigDist)[1], C = 2^(-2:7))

set.seed(1056)
svmFit <- train(Class ~ .,
                data = GermanCreditTrain,
                method = "svmRadial",
                preProc = c("center", "scale"),
                tuneGrid = svmTuneGrid,
                trControl = trainControl(method = "repeatedcv", 
                                         repeats = 5,
                                         classProbs = TRUE))

# note--different resampling methods in trainControl include:
# cv, LOOCV, LGOCV, boot, boot632


## Print the results

svmFit

## A line plot of the average performance. The 'scales' argument is actually an 
## argument to xyplot that converts the x-axis to log-2 units.

plot(svmFit, scales = list(x = list(log = 2)))

## Test set predictions

predictedClasses <- predict(svmFit, GermanCreditTest)
str(predictedClasses)

## Use the "type" option to get class probabilities

predictedProbs <- predict(svmFit, newdata = GermanCreditTest, type = "prob")
head(predictedProbs)



# Choosing Between Models -------------------------------------------------
# continued from previous section

set.seed(1056)
glmProfile <- train(Class ~ .,
                    data = GermanCreditTrain,
                    method = "glm",
                    trControl = trainControl(method = "repeatedcv", 
                                             repeats = 5))
glmProfile

resamp <- resamples(list(SVM = svmFit, Logistic = glmProfile))
summary(resamp)

modelDifferences <- diff(resamp)
summary(modelDifferences)

## The actual paired t-test:
modelDifferences$statistics$Accuracy


{% endhighlight r %}