Overview
--------

One thing that people regularly do is quantify how much of a particular
activity they do, but they rarely quantify how well they do it. In this
project, my goal will be to use data from accelerometers on the belt,
forearm, arm, and dumbell of 6 participants and predict the manner in
which they did the exercise.

More information is available from the website here:
<http://groupware.les.inf.puc-rio.br/har> (see the section on the Weight
Lifting Exercise Dataset).

Loading Data
------------

Let’s begin with loading the training and testing datasets and
performing a quick overview of the data we have.

Loading the data can request some time.

    trainDataURL <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
    testDataURL <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
    temp <- tempfile() 
    download.file(trainDataURL, temp)
    training <- read.csv(temp, stringsAsFactors = FALSE)
    unlink(temp)
    temp <- tempfile() 
    download.file(testDataURL, temp)
    testing <- read.csv(temp, stringsAsFactors = FALSE)
    unlink(temp)

    dim(training)

    ## [1] 19622   160

As you see above, training dataset contains 160 variables and 19622
observations. Thus you can find the information about the variables on
the official site (<http://groupware.les.inf.puc-rio.br/har>)

Pre-processing Data
-------------------

There are many NA values in the data set, so we use KnnImpute method to
impute those values. Besides, we try to standardize each features and
use PCA to reduce features.

    library(caret)

    ## Loading required package: lattice

    ## Loading required package: ggplot2

    library(RANN)
    set.seed(12)
    training$classe <- as.factor(training$classe)
    training <- training[,-nearZeroVar(training)]
    training <- training[,-c(1,2,3,4,5,6,7)]
    inTrain <- createDataPartition(y=training$classe, p=0.75, list=FALSE)
    training <- training[inTrain,]
    validation <- training[-inTrain,]
    preProc <- preProcess(training[,-length(training)],method=c("center", "scale", "knnImpute", "pca"), thresh=0.9)
    cleanData <- predict(preProc,training)

Now data is ready for building a model.

Building model
--------------

After getting the clean data set from the above processing, we use "knn"
method to build the model. We use testing data to evaluate the
performance of our model. It will take a time to fit our model.

    fit <- train(classe ~., data=cleanData, method="knn")

Now we can check the accuracy of our model on the validation data.

    test <- predict(preProc, validation[,-length(validation)])
    confusionMatrix(validation$classe, predict(fit,test))

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1065    2   10    5    0
    ##          B   11  659   14    1    0
    ##          C    3    7  615    6    1
    ##          D    1    0   14  595    0
    ##          E    1    6    4    2  667
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.9761          
    ##                  95% CI : (0.9707, 0.9808)
    ##     No Information Rate : 0.293           
    ##     P-Value [Acc > NIR] : < 2.2e-16       
    ##                                           
    ##                   Kappa : 0.9698          
    ##  Mcnemar's Test P-Value : 0.0008566       
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.9852   0.9777   0.9361   0.9770   0.9985
    ## Specificity            0.9935   0.9914   0.9944   0.9951   0.9957
    ## Pos Pred Value         0.9843   0.9620   0.9731   0.9754   0.9809
    ## Neg Pred Value         0.9939   0.9950   0.9863   0.9955   0.9997
    ## Prevalence             0.2930   0.1827   0.1781   0.1651   0.1811
    ## Detection Rate         0.2887   0.1786   0.1667   0.1613   0.1808
    ## Detection Prevalence   0.2933   0.1857   0.1713   0.1654   0.1843
    ## Balanced Accuracy      0.9893   0.9846   0.9652   0.9861   0.9971

As you can see we have reached accuracy of 0.9761. It is a good result.

Prediction
----------

After fitting the model, we can predict data for the test data. The
result you can see below.

    testing <- testing[,names(testing) %in% names(training)]
    test <- predict(preProc, testing)
    predict_result <- predict(fit, test)
    predict_result

    ##  [1] B A C A A E D B A A D C B A E E A B B B
    ## Levels: A B C D E

Conclusion
----------

-   We can predict new data with the model we build.
-   Accuracy of our model is 0.9761.
