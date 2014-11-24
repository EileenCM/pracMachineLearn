pracMachineLearn
================
I have not been able to push anything to Github, so I give up and have included my code here.
---
title: "Exercising_Correctly"
output: html_document
---

```{r}
setInternet2(use = TRUE)

download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", destfile = "training.csv")
exertrain <- read.csv("training.csv")

download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", destfile = "grading.csv")
exergrade <- read.csv("grading.csv")
```

When reviewing the dataset, it becomes clear that the majority of variables are almost empty.  This seems to be connected with Velleso, et al's discussion in the PDF of using "sliding windows" to predict the outcomes, which would require these averages.  Since only some 400 observations out of a total of 19620 had data, those variables don't seem to have a place here.  I also removed X (the index) and the timestamp and window data.  (The timestamps are artificially correlated with the classe outcomes.)


```{r}
smallexertrain<-exertrain[,c(2,8:11,37:49,60:68,84:86,102,113:124,151:160)]
smallexergrade<-exergrade[,c(2,8:11,37:49,60:68,84:86,102,113:124,151:160)]
```

Now to create the testing and training datasets, using the 70/30 split common in class.

```{r}
library(caret)
inTrain<-createDataPartition(y=smallexertrain$classe,p=0.7, list=FALSE)
training<-smallexertrain[inTrain,]
testing<-smallexertrain[-inTrain,]
```

First try at a model.

```{r, echo=FALSE}

fitControl <- trainControl(method = "cv", number = 4, repeats = 10)
set.seed(925)
rfFit1 <- train(classe ~ ., data = training, method = "rf",trControl = fitControl,
                 prox=TRUE)
rfFit1
pred<-(rfFit1,testing)
testing$predRight<-pred==testing$classe
table(pred,testing$classe)
```

Then lastly, apply the model to the graded set.

```{r}
answers<-(rfFit1,smallexergrade)


