---
title: "Coursera Machine Learning Project"
author: "Daniel Veit"
date: "5/2/2022"
output:
  html_document: default
  pdf_document: default
---

# **Introduction**

### **Background**
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). 

### **Data**
The training data for this project are available here: 
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

The data for this project come from this source: http://groupware.les.inf.puc-rio.br/har. If you use the document you create for this class for any purpose please cite them as they have been very generous in allowing their data to be used for this kind of assignment. 

### **What you should submit**
The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. You may use any of the other variables to predict with. You should create a report describing how you built your model, how you used cross validation, what you think the expected out of sample error is, and why you made the choices you did. You will also use your prediction model to predict 20 different test cases. 

# **Analysis**

### **Summary of Approach**
1. Load the data.
2. Perform cross-validation to built a valid model, using 70% of the original data for model building (training data) and 30% of the data for testing (testing data).
3. Clean the data by excluding variables that are inexplainable or missing data.
4. Perform exploratory data analysis.
5. Perform Principle Component Analysis (PCA) to reduce the number of variables.
6. Apply Random Forest Method to build a model using the training data.
7. Validate the model using the testing data set.
8. Apply the model to estimate classes of 20 observations.

### **Load packages and libraries**
```{r}
suppressMessages(library(ggplot2))
suppressMessages(library(caret))
suppressMessages(library(randomForest))
```

### **1. Load the Data and Perform Exploratory Data Analysis**
```{r}
DataTraining <- read.csv("pml-training.csv")
DataTesting  <- read.csv("pml-testing.csv")
```

### **2. Cross validation**
Use 70% of the original data for model building (training data) and 30% of the original data for testing (testing data)
```{r}
Train <- createDataPartition(y=DataTraining$classe,p=.70,list=F)
Training <- DataTraining[Train,]
Testing <- DataTraining[-Train,]
```

### **3. Clean Training Data**
Exclude variables with over 95% missing data:
```{r}
Clean <- grep("name|timestamp|window|X", colnames(Training), value=F) 
TrainingClean <- Training[,-Clean]
TrainingClean[TrainingClean==""] <- NA
RateNA <- apply(TrainingClean, 2, function(x) sum(is.na(x)))/nrow(TrainingClean)
TrainingClean <- TrainingClean[!(RateNA>0.95)]
```

### **4. Perform Exploratory Analysis**
Perform exploratory data analysis:
```{r}
str(TrainingClean)
plot(as.factor(TrainingClean$classe), col="red", main="Variable Classe Frequency within Training Data", xlab="classe", ylab="Frequency")
```
\
Based on the plot above, level A has the highest frequency (exercise completed correctly) and the other levels are all within the same order of magnitude of each other.

### **5. Perform Principle Component Analysis (PCA)**
Perform PCA because the number of variables is exceedingly high:
```{r}
PCA <- preProcess(TrainingClean[,1:52],method="pca",pcaComp=25) 
TrainingPCA <- predict(PCA,TrainingClean[,1:52])
```

### **6. Apply Random Forest Method**
Apply RFM to build a model using the training data:
```{r}
RandomForestModel <- randomForest(as.factor(TrainingClean$classe) ~ ., data=TrainingPCA, do.trace=F)
importance(RandomForestModel)
```

### **7. Validate the Model**
Validate the model using the testing data set, first by cleaning the data:
```{r}
TestingClean <- Testing[,-Clean]
TestingClean[TestingClean==""] <- NA
RateNA <- apply(TestingClean, 2, function(x) sum(is.na(x)))/nrow(TestingClean)
TestingClean <- TestingClean[!(RateNA>0.95)]
```

Then performing the same PCA and RFM approach as the training data:
```{r}
TestPCA <- predict(PCA,TestingClean[,1:52])
confusionMatrix(as.factor(TestingClean$classe),predict(RandomForestModel,TestPCA))
```
As can be seen by the results of the confusion matrix, the model has an accuracy of 97.5% and is therefore a sufficient model to predict accurate classes. 

### **8. Apply the Model to Estimate Classes of 20 Observations**
First by cleaning the data:
```{r}
DataTestingClean <- DataTesting[,-Clean]
DataTestingClean[DataTestingClean==""] <- NA
RateNA <- apply(DataTestingClean, 2, function(x) sum(is.na(x)))/nrow(DataTestingClean)
```

Then performing the same PCA and RFM approach as the training data:
```{r}
DataTestingClean <- DataTestingClean[!(RateNA>0.95)]
DataTestingPCA <- predict(PCA,DataTestingClean[,1:52])
DataTestingClean$classe <- predict(RandomForestModel,DataTestingPCA)
DataTestingClean$classe
```

# **Conclusions**
In this study, a total of 19622 observations from weight lifting exercise were used to analyze and predict correct body movement from others during the exercise. From the 19622 observations, 70% of the observations were used to build a model using the random forest method, while the remaining 30% of the observations were used for model validation (cross-validation). The results of the random forest model yielded an accuracy of 97% for the testing set, which was not used to build the initial model. The specificity was over 99% for all classes and the sensitivity varied between 93%-99%. Overall, the model is well developed to predict the exercise classes during weight lifting.