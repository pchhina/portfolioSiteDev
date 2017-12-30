---
title: "Exercise Quality Prediction using Machine Learning"
date: 2017-12-30T05:04:40-06:00
draft: false
---


## Introduction

Many devices exists today in the market (such as Jawbone Up, Nike FuelBand, and Fitbit) inexpensively record, track and analyze data on personal physical activity. Although by using these deviices, quantiy of the physical activity is often monitored, quality is rarely quantified. In this project, our goal is be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants in an attempt to predict whether the activity is performed correcly or not. To generate the data, subjects were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).


### Loading Packages and Downloading Data

We start by downloading the data from the source and reading it into `R`. Feature vectors from training data are stored separately from outcome. Testing data is stored in a separate object and is used only in the final stage for predictions.


```r
# Load Packages
library(caret)
library(dplyr)
library(ggplot2)
library(corrplot)
rm(list = ls())
# Download Data
src.trn <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
if (!file.exists("pml-training.csv")) {
    download.file(url = src.trn, destfile = "pml-training.csv")
}
src.test <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
if (!file.exists("pml-testing.csv")) {
    download.file(url = src.test, destfile = "pml-testing.csv")
}
training <- read.csv("pml-training.csv")
training_X <- training[, -160]
training_Y <- training[, 160]
testing <- read.csv("pml-testing.csv")
testing_X <- testing[, -160]
```

### Exploratory Data Analysis and Feature Selection

Before we apply machine learning techniques to the training data, it is important to carry out some exploratory analysis in an effort to address any problems with the data (for example, missing values).

#### *NA* Values

Since most machine learning algorithms generally have difficulty when some values for features have *NA* values, it is important to identify those values in the data set. Specific strategy (remove or impute) to handle these values  would depend on how many *NAs* are there and their distribution.

We will first identify number of NAs in each column.


```r
# count number os NAs in each variable
table(sapply(training_X, function(x) sum(is.na(x))))
```
 <table style="width:50%; border:1px solid grey;">
  <tr>
    <th>0</th>
    <th>19216</th>
  </tr>
  <tr>
    <td>92</td>
    <td>67</td>
  </tr>
</table> 

The table above shows that 92 variables have no NAs while there are 67 variables with 98% missing values. With so many NA values for these variables, it is probably good to remove those variables.


```r
training.na <- sapply(training_X, function(x) ((sum(is.na(x)))/dim(training_X)[1]) > 
    0.95)
training.small <- training_X[, !training.na]
testing.small <- testing_X[, !training.na]
```

#### Near Zero Variance Features

Now we will identify and remove near zero variance features. The cutoff used for the ratio of the most common value to the second most common value is 2. The cutoff for the percentage of distinct values out of the number of total samples used is 20. In addition, some redundant variables (timestamps, names etc.) are also removed.


```r
# Remove near zero variance columns
remove_cols <- nearZeroVar(training.small, names = TRUE, freqCut = 2, uniqueCut = 20)
allCols <- names(training.small)
training.smaller <- training.small[, setdiff(allCols, remove_cols)]
testing.smaller <- testing.small[, setdiff(allCols, remove_cols)]

# Remove redundant variables
rm.var <- names(training.smaller) %in% c("X", "user_name", "raw_timestamp_part_1", 
    "raw_timestamp_part_2", "cvtd_timestamp")
training.X <- training.smaller[!rm.var]
testing.X <- testing.smaller[!rm.var]
```

#### Correlated Features

Finally, we identify the features that are highly correlated and remove pairwise features with absolute correlation of 0.8 or greater. The two plots below show the correlation matrix of all features before and after removing the correlated variables.


```r
# Remove highly correlated variables
corr.mat <- cor(training.X)

corr.high <- findCorrelation(corr.mat, cutoff = 0.8)
names(training.X)[corr.high]
training.X.uncorr <- training.X[, -corr.high]
testing.X.uncorr <- testing.X[, -corr.high]
par(mfrow = c(1, 2))
corrplot(corr.mat, order = "hclust", tl.pos = "n")
corrplot(cor(training.X.uncorr), order = "hclust", tl.pos = "n")
mtext("Correlation Plot: Before and after highly correlated variables removed", 
    side = 3, outer = TRUE, line = -3)
```

{{<figure src="../images/corrplot.png" width="90%" >}}

### Model Building
In this section, we are going to build different models. This is a classification problem, so we will try classification tree (`rpart`). Since recursive binary partitioning approach used in classification tree may lead to overfitting, a better alternative may be to use random forest (`ranger`). In addition, we will explore k-nearest neighbor (`knn`) approach for its simplicity. Finally, we will also use `glmnet`, which is a form of generalized linear model but penalizes for large number of predictors. All the models are build using `caret` package.

#### Cross-Validation
To build the four models described above, a 5-fold cross validation is used to get a reasonable estimate of out-of-sample error. For the models to be comparable, a single `trainingControl` object is created and used for all four models.



#### Model Results

```r
plot(modelknn)
```
{{<figure src="../images/knn.png" width="90%" >}}
```r
plot(modelrpart)
```
{{<figure src="../images/rpart.png" width="90%" >}}
```r
plot(modelrf)
```
{{<figure src="../images/rf.png" width="90%" >}}
```r
plot(modelglm)
```
{{<figure src="../images/glm.png" width="90%" >}}


From the accuracy plots above, we note that *k nearest neighbors* have accuracy ranging from 74% to 78% with best accuracy achieved with 5 neighbors. *Classification tree* is only 33%-53% accurate with best accuracy for 0.02 complexity parameter. *glmnet* achieved best accuracy of 54%. Finally, *random forest* has accuracy ranging from 97% to 98% with best accuracy achieved with 18 randomly selected predictors.

### Model Selection

For model selection we will compare accuracy of all four models and pick the one with largest accuracy. Since we have used 5-fold cross validation in our model building in previous section, we will get confidence intervals from those accuracy estimates so we can make an informed selection decision.


```r
# Model Selection
mdl.list <- list(glmnet = modelglm, rf = modelrf, knn = modelknn, rpart = modelrpart)
resamps <- resamples(mdl.list)
summary(resamps)
dotplot(resamps)
```
{{<figure src="../images/resamps.png" width="90%" >}}

It is evident that random forest model gives the best accuracy among the choices explored. We will select this as our final model for evaluating the test data in the next section.

### Model Testing
Finally we are going to test the model using 20 data points provided in the test set. 


```r
testPred <- predict(mdl.list, newdata = testing.X.uncorr)
testPred$rf
```

### Acknowledgements

The author sincerely wants to thank Professor Hugo and his team (from the Departamento de Informática at the Pontifical Catholic University of Rio de Janeiro) for making this data publicly available. More information on this data set can be accessed from their project at http://groupware.les.inf.puc-rio.br/har.

