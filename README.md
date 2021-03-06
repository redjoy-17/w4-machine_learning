---
title: "To what class do you belong to?"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
directory<-"/Users/Gaia/Downloads"
setwd(directory)
library(caret)
library(RANN)
library(e1071)
library(randomForest)
```

## Prediction Assignment 

The aim of this assignment is to predict the classe of the testing set once the machine learning algorithm is trained on the training set. 

### Reading the data

First of all it is necessary to import the data in Rstudio. I define rispectively the dataset training and testing.

```{r reading files csv, echo=TRUE}
testing<-read.csv("pml-testing.csv",header=TRUE)
testing<-testing[,2:160]
training<-read.csv("pml-training.csv",header=TRUE)
training<-training[,2:160]
```

### Preprocessing data

Once imported the dataset it is necessary to preprocess the dataframes, i.e. training and testing, in order to eliminate the columns not important for the train of the machine learning algorithm and the columns full of NAs values. 

```{r preprocessing & training, echo=TRUE}
training<-training[,-(1:6)]
testing<-testing[,-(1:6)]
col_na_training<-colSums(sapply(training,is.na))
col_na_testing<-colSums(sapply(testing,is.na))
training<-training[,col_na_training==0 & col_na_testing==0]
testing<-testing[,col_na_training==0 & col_na_testing==0]
```

### The machine learning algorithm

The machine learning algorithm implemented consists of three  main steps:
1. Data Partition: the training set is split into two parts: train & test where train is composed by 13737 observations and test composed by 5885 observation;
2. Training Part: in this part it is used a random forest algorithm with cross fold validation (fold=3) and parallel computing equal to true in order to speed the computation and get the results in a faster time;
3. Testing Part: the output of the test set is predicted by the algorithm previously trained and then it is compared to the true output in order to see the accuracy of the result obtained.  

#### Data Partition 

```{r cross validation, echo=TRUE}
set.seed(32323)
result<-createDataPartition(y=training$classe, p=0.7, list=FALSE)
train<-training[result,]
test<-training[-result,]
```

```{r parallel, echo=FALSE}
library(parallel)
library(doParallel)
cluster<-makeCluster(detectCores()-1)
registerDoParallel(cluster)
```

#### Training Part 

```{r control, echo=TRUE}
fitControl<-trainControl(method="cv",number=3,allowParallel = TRUE)
modelFit<-train(classe~., data=train, method="rf", trControl=fitControl)
```             

```{r closure, echo=FALSE}
stopCluster(cluster)
registerDoSEQ()
```

#### Testing Part

```{r testing, echo=TRUE}
pred<-predict(modelFit,newdata=test)
table(pred,test$classe) #to do a comparison
```  

As it is possible to see from the table proposed the accuracy of the algorithm proposed is of 99,24%. 


### Testing dataset part

This part of the code is devoted to find the class correspondent to the testing set. The output is not displayed as it is part of the quiz.

