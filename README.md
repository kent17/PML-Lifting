PML-Lifting
===========

<i>Description of my solution for the PML Coursera assignment</i>

<h5>Exploratory analysis</h5>

The pml-training dataset, as well as the pml-testing one, seems to be split into two main parts:
- Window agregated data
- Real-time data

Columns correspond to either one or the other type of data, they are complementary, which leads to a lot of NAs. Only about 1/3 of columns concern real-time data (not counting the 'general information' columns like timeframe or name of the weightlifter). However the number of rows of window agregated data is... 2% of the total dataset. As we need to choose between the two types of data (which are two subsets of variables, whose meaningful ones are complementary), the best option is therefore to delete the 2% of window agregated data, and the corresponding variables. That way, we don't have any NA anymore, keep 98% of the data but 100% of the meaningful data (we deleted the agregated data based on the raw data we kept).

<h5>Finding the good model</h5>

Once I got the correct rows and variables, it is time to start training (and finding a good) model. I used a cross-validation split of 10 folds, on a shuffled dataset. To speed up the research of the best fitting algorithm, I first trained my model on only 1000 rows.

training2 <- training1[sample(nrow(training1)),]
training2 <- training2[1:1000,]
ctrl <- trainControl(method="cv", number=10)

I then tried to fit models with random forest, logistic regression and polynomial SVM algorithms (all are classification algorithms).

logRegFit <- train(training2[,-54], training2$classe,method="LogitBoost",tuneGrid=NULL,prox=TRUE,na.action=na.omit,trControl=ctrl)
rfFit <- train(training2[,-54], training2$classe,method="rf",na.action=na.omit,trControl=ctrl)
svmFit <- train(training2[,-54], training2$classe,method="svmPoly",na.action=na.omit,trControl=ctrl)

Here are the results for each algorithm:
<table>
<tr>
  <th></th><th>1000 samples</th>
</tr>
<tr>
  <th>Logistic Regression</th><th>0.87</th><th> </th>
</tr>
<tr>
  <th>Random Forest</th><th>0.92</th><th></th>
</tr>
<tr>
  <th>Polynomial SVM</th><th>0.86</th><th></th>
</tr>
</table>

For the testing step, we have to remove the same bunch of variables as for the training step. Here are the predictions by algorithm:
- Logistic Regression: <NA> A <NA> A A E D <NA> A A D <NA> B A E E <NA> <NA> <NA> B
- Random Forest: B A A A A E D D A A B C B A E E A B B B
- SVM: B A A A A E D A A A D C B A E E A B B B

It's quite obvious that logistic regression has serious difficulties with its model, maybe it's not trained enough to be able to predict clearly the test samples. The two other algorithms, at least, seem to be more reliable by predicting the same outcome. This step was just intended to have more information on the models reliability than just the cross-validation accuracy results. Let's try to train the models with more data, 5000 (in lieu of 1000 previously) seems more acceptable. 

<table>
<tr>
  <th></th><th>1000 samples</th><th><b>5000 samples</b></th>
</tr>
<tr>
  <th>Logistic Regression</th><th><b>0.92</b></th>
</tr>
<tr>
  <th>Random Forest</th><th><b>0.99</b></th>
</tr>
<tr>
  <th>Polynomial SVM</th><th><b>0.97</b></th>
</tr>
</table>

And here are the predictions on the test set:
- Logistic Regression: D A B <NA> A E D D A A B C B A E <NA> A B B B 
- Random Forest: B A A A A E D B A A B C B A E E A B B B
- SVM: B A B A A A D B A A D C B A E E A B B B

Logistic Regression has again some difficulties. RF and SVM do not converge more with a more well-trained model, but 2 disagreements between them seems acceptable. Given the higher score of the RF model compared to the others, I would choose this one prediction.

<h5> Going further </h5>

I tried to combine predictors, in particular Random forest and SVM ones. It was unsuccesful, I got a lot of errors trying to train SVM against new data.

We could make use of the timestamps. Shuffled data seems enough to have a very good accuracy, but maybe we could have even better scores by taking into account at which moment each row was recoreded. The order of the movements has its importance when lifting, the movements are not the same when lifting up and letting down the weights. A pattern in the time dimension could be detected by the algorithm (like the spatial position of some points at each record). 
In practical, that task would take some time. For each lifting of each participant, we should set the timestamp for the first entry to zero, and use all timestamps relatively to it. But more than time, it is here impossible: the test set asks the model to find a class with a single record, so it is not relevant in this case.
