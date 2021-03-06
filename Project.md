---
title: "Identification of Proper and Improper Excercize Techniques"
author: "Benjamin Sanders"
date: 06/25/2016
---


## Abstract:
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. The goal of this project is to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants to identify correct and incorrect form. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

To acheive this goal we used a random forest classification model with 5-fold cross validation. This proved to be around 99% accurate, which was sufficient for this purpose.


## Data

The training data for this project are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv


The data for this project came from this source: http://groupware.les.inf.puc-rio.br/har.


## Data Preparation

First we read in the data sets, making sure that NA strings are accounted for.


```r
setwd("~/Coursera/Machine Learning")
train <- read.csv("pml-training.csv", na.strings = c("", "NA", "NULL"))
verify  <- read.csv("pml-testing.csv",  na.strings = c("", "NA", "NULL"))
```

Next we remove the near Zero Variance Variables because they will not contribute much.

```r
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.1.3
```

```
## Loading required package: lattice
```

```
## Warning: package 'lattice' was built under R version 3.1.3
```

```
## Loading required package: ggplot2
```

```
## Warning: package 'ggplot2' was built under R version 3.1.3
```

```r
NZV <- nearZeroVar(train, saveMetrics = TRUE)
train <- train[, !NZV$nzv]
verify <- verify[, !NZV$nzv]
```

The columns which have a substancial amount of NA values will not be able to be used reliably, so they are removed as well.


```r
rows <- nrow(train)
condition <- colSums(is.na(train))/rows<0.3
train <- train[,condition]
verify <- verify[,condition]
```

Finally, we remove some columns which do not support the goals of the assignment and clean up the environment for the modelling phase.


```r
toRemove <- c("X","user_name", "raw_timestamp_part_1", "raw_timestamp_part_2", "cvtd_timestamp", "num_window")
toKeep <- names(train)[!(names(train) %in% toRemove)]
train <- train[,toKeep]
# verify has a problem_id column rather than a classe column.
toKeep <- names(verify)[!(names(verify) %in% toRemove)]
verify <- verify[,toKeep]

rm(list = c("NZV","condition","rows","toKeep","toRemove"))
```

## Model Creation

We use 70%of the data for training and 30% for testing.

```r
inTrain <- createDataPartition(y=train$classe, p = 0.7, list = FALSE)
test <- train[-inTrain,]
train <- train[inTrain,]
```

Random forest is frequently a good place to start. We will use 5-fold cross validation to reduce overfitting.

```r
library(randomForest)
```

```
## Warning: package 'randomForest' was built under R version 3.1.3
```

```
## randomForest 4.6-12
## Type rfNews() to see new features/changes/bug fixes.
```

```r
library(e1071)
```

```
## Warning: package 'e1071' was built under R version 3.1.3
```

```r
modelRF <- train(classe ~ ., data = train, method = "rf", trControl = trainControl(method = "cv", 5), ntree = 250)
```

## Estimation of Out-Of Sample Error

```r
answers <- predict(modelRF, newdata = test)
sum(answers == test[["classe"]]) / nrow(test)
```

```
## [1] 0.9923534
```

Testing on an out of sample data set we see a 99.18437% accuracy. That means that we can expect an out of sample error rate of around 0.815633%. This is an acceptable error rate for our purposes, so no further model development is needed.

