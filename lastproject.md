---
title: "Prediction Assignment Writeup"
author: "DIKSHA SAINI"
output:
  pdf_document: default
  html_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

### Executive Summary
This report analyzed the relationship between transmission type (manual or 
automatic) and miles per gallon (MPG). The report set out to determine which 
transmission type produces a higher MPG. The `mtcars` dataset was used for this
analysis. A t-test between automatic and manual transmission vehicles shows that
manual transmission vehicles have a 7.245 greater MPG than automatic 
transmission vehicles. After fitting multiple linear regressions, analysis 
showed that the manual transmission contributed less significantly to MPG, only
an improvement of 1.81 MPG.  Other variables, weight, horsepower, and number of 
cylinders contributed more significantly to the overall MPG of vehicles.


####Data Loading
The first step is to load the training and testing data to R. There are 160 variables on 19622 observations
```{r warning=FALSE,cache=TRUE}
library(caret)
setwd("C:\\Users\\Neetu Saini\\Desktop\\data science\\course8")
training = read.csv("C:\\Users\\Neetu Saini\\Desktop\\data science\\course8\\training_data.csv")
testing = read.csv("C:\\Users\\Neetu Saini\\Desktop\\data science\\course8\\testing_data.csv")
```


####Cleaning the data
On closer inspection, the original dataset has 160 variables (including the response variable). However, upon scanning through the summary of the dataset, the first 5 variables are unlikely to be explanatory since they are data identifiers for individual and time, and an additional 100 variables has 98% of their observations in NAs so they are highly unlikely to contain much value as well so they are also dropped.
```{r warning=FALSE,cache=TRUE}
training1 = training[,-c(1:5,12:36, 50:59, 69:83,87:101,103:112,125:139,141:150)]
testing1 = testing[,-c(1:5,12:36, 50:59, 69:83,87:101,103:112,125:139,141:150)]
```


#### Model selection
Due to the size of the dataset, it is diffcult to employ algorithms like random forest on all of the data. So the training dataset is further divided into 20 random subsets. The first subset is first used to train 4 types of models: random forest, boosting, classification trees, and linear discriminant.

The accuracy performance of classification trees (54%) and linear discriminant (67%) is far below that of random forest (88.7%) and boosting (87%). As such, only the latter two are tried on a second subset of the training dataset.

```{r warning=FALSE, cache=TRUE}
set.seed(2233)
folds<-createFolds(y=training1$classe,k=20,list=TRUE,returnTrain=FALSE)
training1_1<-training1[folds$Fold01,]
modFit1_1_rf<- train(classe~., data=training1_1, method = "rf", prox = TRUE)
modFit1_1_gbm<- train(classe~., data=training1_1, method = "gbm",  verbose = FALSE)

modFit1_1_rpart<-train(classe~., data=training1_1, method = "rpart")
modFit1_1_lda<- train(classe~., data=training1_1, method = "lda")

training1_2<-training1[folds$Fold02,]
modFit1_2_rf<- train(classe~., data=training1_2, method = "rf", prox = TRUE)
modFit1_2_gbm<- train(classe~., data=training1_2, method = "gbm",  verbose = FALSE)
```

On the second subset, accuracy of random forest is 88% and that of boosting is 87.2 %. Both still have very high accuracy. We will then deploy the model to test out-of-sample error with a third subset of data
```{r warning=FALSE, cache=TRUE}
training1_3<-training1[folds$Fold03,]
confusionMatrix(predict(modFit1_1_rf,training1_3),as.factor(training1_3$classe))
confusionMatrix(predict(modFit1_1_gbm,training1_3),as.factor(training1_3$classe))

training1_4<-training1[folds$Fold04,]
confusionMatrix(predict(modFit1_1_rf,training1_4),as.factor(training1_4$classe)) 
confusionMatrix(predict(modFit1_1_gbm,training1_4),as.factor(training1_4$classe))
```

##Appendix
###Plotting decision tree(method="rpart")
From the out-of-sample testing for subset 3 and 4, random forest model still outperforms boosting, with an accuracy rate of over 94%. As such, the random forest model 1 is chosen.
```{r warning=FALSE, echo=FALSE,cache=TRUE}

predict(modFit1_1_rf,testing1) 
```
