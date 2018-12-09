# Caret_Project
#### The caret package (short for _C_lassification _A_nd _RE_gression _T_raining) is a set of functions that attempt to streamline the process for creating predictive models. The package contains tools for:
#### data splitting
#### pre-processing
#### feature selection
#### model tuning using resampling
#### variable importance estimation
#### Handling missing values
```
preProcess(data, method=c("bagImpute","knnImpute"));predict(pro, newdata)
```
#### Centralization, standardization
```
preProcess(data, method=c("center","scale"))
```
#### Feature selection
```
rfeControl,rfe
```
#### Sampling data partition
```
createDataPartition()
createFold()…
```
#### Model training
```
trainControl()//Set the number of training cross-validation, repeat several times, etc.
train()// Set which model to use for training
```
#### forecast result
```
predict()
```
## Multidrug resistance reversal agent
#### 528 obs. of  342 variables

```
library(lattice)
library(ggplot2)
library(caret)
data(mdrr)
newdata <- mdrrDescr[, -nearZeroVar(mdrrDescr)]
```
#### find the correlation
```
descrCorr <- cor(newdata)
newdata2 <- newdata[, -findCorrelation(descrCorr)]
```
#### Remove collinearity (if any)
```
comboInfo <- findLinearCombos(newdata2)
if(!is.null(comboInfo)){
  newdata3 <- newdata2[, -comboInfo$remove]
} 
```

#### If there is a missing value, use bagImpute, knnImpute to calculate the fill
```
if(nrow(newdata2[!complete.cases(newdata2),])!=0)
{
  process <- preProcess(newdata2, method="bagImpute")
  pre <- predict(process, newdata2)
}
```
#### feather selection
```
subsets <- seq(2, ncol(newdata2), by=2)
```
#### define sbfControl (Control Object For Selection By Filtering (SBF)) Controls the execution of models with simple filters for feature selection.The external resampling method is 'cv', set 5 complete sets of folds to compute for repeated k-fold cross-validation only
```
sbfControls_rf <- sbfControl(  functions = rfSBF,  method = 'cv',  repeats = 5)

```
#### sbf: feature selection
```
pro <- sbf(newdata2, mdrrClass, sizes = subsets, sbfControl=sbfControls_rf)
summary(pro)
```
####  feature selected variables
```
pro$optVariables

```
#### Training model
#### Get the data after feature selection
```
newdata4 <- newdata2[, pro$optVariables]
```
#### Training data and test data
```
index <- createDataPartition(mdrrClass, p=3/4, list=F)

trainx <- newdata4[index,]
trainy <- mdrrClass[index]
testx <- newdata4[-index,]
testy <- mdrrClass[-index]
```
#### Set model training parameters and fit the model
```
fitControl <- trainControl(method="repeatedcv", number=10, repeats=3, returnResamp="all")


gbmGrid <- expand.grid(interaction.depth=c(1,3), n.trees=seq(50,300,by=50), shrinkage=0.1,n.minobsinnode = 20)


gbmFit1 <- train(trainx, trainy, method="gbm", trControl=fitControl, tuneGrid= gbmGrid, verbose=F)
trainControl
png("foo.png",family="GB1")
plot(gbmFit1)


dev.off()
```
#### Use a trained model for predict
```
predict(gbmFit1, newdata=testx)
```
#### Confusion matrix view results
```
table(testy, predict(gbmFit1, newdata=testx))
```
#### testy      Active Inactive
#### Active       61       13
#### Inactive     12       45

#### The classifier made a total of 131 predictions (e.g., 131 patients were being tested for the presence of that Multidrug resistance reversal agent). Out of those 131 cases, the classifier predicted "active" 73 times, and "inactive" 58 times. In reality, 74 patients in the sample have the Multidrug resistance reversal agent, and 57 patients do not. 
#### Accuracy: Overall, how often is the classifier correct? (TP+TN)/total = (45+61)/131 = 0.809
#### Misclassification Rate(Error Rate):0.191
#### Precision: When it predicts inactive, how often is it correct? 0.776
#### Prevalence: How often does the inactive condition actually occur in our sample? 57/131 = 0.435
#### Use another model (treebagg method)
```
gbmFit2 <- train(trainx, trainy, method="treebag", trControl=fitControl)
table(testy, predict(gbmFit2, newdata=testx))
```
#### testy      Active Inactive
####   Active       64       10
####   Inactive     13       44
#### The classifier made a total of 131 predictions (e.g., 131 patients were being tested for the presence of that Multidrug resistance reversal agent). Out of those 131 cases, the classifier predicted "active" 77 times, and "inactive" 54 times. In reality, 74 patients in the sample have the Multidrug resistance reversal agent, and 57 patients do not. 
#### Accuracy: Overall, how often is the classifier correct? (TP+TN)/total = (44+64)/131 = 0.824
#### Misclassification Rate(Error Rate):0.176
#### Precision: When it predicts inactive, how often is it correct? 0.815
#### Prevalence: How often does the inactive condition actually occur in our sample? 57/131 = 0.435
```
models <- list(gbmFit1, gbmFit2);
predValues <- extractPrediction(models, testX=testx, testY=testy)
```
