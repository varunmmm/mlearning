# Predict
Varun Yadav  
Sunday, May 24, 2015  

Given 2 files, we are asked to develop a machine learning paradigm to predict the type of exercise performed based an a variety of measurements. The first file is the training file, used to develop our algorithm, and the second is used to make our final predictions.

The training data is: https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The testing data is: https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

Once downloaded to our working directory, we read in the file to perform some basic exploratory data analysis. We notice there are many blanks and NA values (I do not show here in the interest of space), so I shall re-read the file so that all non-valid entries (blanks, DIV/0, NA), are read in as NA in R. I continue to examine and remove columns which contain NA's, as well as remove columns which I do not believe have any outcome on the class.



```r
setwd("G:/Analytics/R machine learning/")
a=read.csv("pml-training.csv",na.strings=c("","NA"))
b=a[,!apply(a,2,function(x) any(is.na(x)) )]
c=b[,-c(1:7)]
```

This leave us with 19622 observations and 53 predictors (one of which is the response variable)

To continue with the analysis we download the necessary packages


```r
##install.packages("randomForest")
library("randomForest")
```

```
## Warning: package 'randomForest' was built under R version 3.1.3
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```r
##install.packages("caret")
library("caret")
```

```
## Warning: package 'caret' was built under R version 3.1.3
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
##install.packages("e1071")
library("e1071")
```

```
## Warning: package 'e1071' was built under R version 3.1.3
```

For cross validation, We split our testing data into sub groups, 60:40


```r
subGrps=createDataPartition(y=c$classe, p=0.6, list=FALSE)
subTraining=c[subGrps,]
subTesting=c[-subGrps, ]
dim(subTraining);dim(subTesting)
```

```
## [1] 11776    53
```

```
## [1] 7846   53
```

We see there are 11776 in the subTraining group, and 7846 in the subTesting group.

I then continue to make a predictive model based on the random forest paradigm, as it is one of the best performing, using the subTraining group. Once the model is made, we predict the outcome of the other group, subTesting, and examine the confusion matrix to see how well the predictive model performed


```r
model=randomForest(classe~., data=subTraining, method="class")
pred=predict(model,subTesting, type="class")
z=confusionMatrix(pred,subTesting$classe)
```


```r
z$table
```

```
##           Reference
## Prediction    A    B    C    D    E
##          A 2231   14    0    0    0
##          B    0 1500    1    0    0
##          C    0    4 1363    8    4
##          D    0    0    4 1277    0
##          E    1    0    0    1 1438
```


```r
z$overall[1]
```

```
##  Accuracy 
## 0.9952842
```

Based on this, The accuracy is 99.31%. The out of sample error, that is the error rate on a new (subTesting) data set, here is going to be 0.69%, with a 95% confidence interval of 0.52% to .9%.

Final Data Set Analysis and Predictions
This is very good, so I continue with the final testing data set. I read it in and preporcess it the same way as the training set previously.



```r
setwd("G:/Analytics/R machine learning/")
d=read.csv("pml-testing.csv",na.strings=c("","NA"))
e=d[,!apply(d,2,function(x) any(is.na(x)) )]
f=e[,-c(1:7)]
predicted=predict(model,f,type="class")
```

The final prediction for the 20 ends up as:



```r
predicted
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```


