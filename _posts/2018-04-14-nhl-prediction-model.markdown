---
layout: post
title:  "NHL Prediction Model"
date:   2018-04-14 09:24:31 -0400
---
Last summer I was invited by [Emmanual Perry](https://twitter.com/MannyElk), who created [corsica.hockey](http://corsica.hockey), to participate in a prediction contest for the upcoming NHL season. Having never built a prediction model, and not really following the NHL, I figured it would be a fun challenge undertake. Manny was kind enough to share a dataset with over 5000 variables for all games since the 2007-08 season. This saved me a lot of time that would have been spent gather and cleaning data. Below are the details on how I built my model.

### Imputing Missing Values
There were some pieces of data missing for some games so I needed to impute the missing values for those games. For any stat over the last n games, whenever the team had not yet played n games, the cumulative value for that stat was used. For cumulative stats on a team's first game of the season, the value from the prior season was used. Any other missing values were replaced with the mean value for the stat.

### Cross Validation
I didn't want to randomly split the data into a training set and test set since including data from the same season in both the training set and test set could cause the model to learn things it shouldn't know.  For each season, I held out all data for that season as the test set and trained a model on the data from all remaining seasons, except for the next season. The reason the next season was excluded is because it contains prior season variables that could impact predictions on the test set. Since the contest was being scored using log loss, I computed the log loss of the predictions on the test set and averaged those across all seasons to get the model validation score.

### Reducing the Feature Set
Through [feature selection](http://scikit-learn.org/stable/modules/feature_selection.html) and some reading up on which team stats best predict future success, I narrowed down the 5000+ variables to around 100. At this point I started building models with different combinations of the reduced feature set. This was done mostly through trial and error by trying different subsets of the reduced feature set and checking the out of sample validation score. My general strategy for each feature was to see whether including it in the model improved out of sample predictions and if it did try different variations of that stat (cumulative, last 10 games, last 20 games, adjusted/raw, different strength states, ...) and see which one did the best on out of sample predictions. Doing this for many combinations of variables, I got down to the final feature set listed below.

### Ensemble
My final model was an ensemble model of three different models. Those models were a [logistic regression](http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html) model, a [catboost](https://catboost.yandex/) classifier model, and a [naive bayes](http://scikit-learn.org/stable/modules/generated/sklearn.naive_bayes.GaussianNB.html) model. I also built a [random forest](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html) model and an [adaboost](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.AdaBoostClassifier.html) classifier, but including those in the ensemble did not improve the out of sample predictions.

### Feature Set
The following variables were used in the final model, in no particular order:

* last season adjusted goal +/- per 60 at all strength states
* is the team on the 2nd game of back-to-back?
* games played in last 3 days
* cumulative [K](http://www.corsica.hockey/misc/K_Manuscript.pdf)
* cumulative adjusted expected goal +/- per 60 at all strength states
* cumulative adjusted fenwick at 5-on-5
* cumulative goal +/- at all strength states
* last 20 games adjusted corsi for at 5-on-5
* last 25 games goal +/- at all strength states
* last 25 games adjusted expected goal +/- at all strength states
* last 10 games adjusted corsi +/- per 60 at 5-on-5
* minute weighted forward [WAR](http://www.corsica.hockey/blog/2017/05/20/the-art-of-war/) per game
* minute weighted defensemen WAR per game
* minute weighted forward [Game Score](https://hockey-graphs.com/2016/07/13/measuring-single-game-productivity-an-introduction-to-game-score/) per game
* minute weighted defensemen Game Score per game
* starting goalie WAR
* starting goalie Game Score
* starting goalie goals saved above average per 30 shots at all strength states
* (power play adjusted expected goals for per 60) - (opponent penalty kill adjusted expected goals against per 60)
* rest advantage (-1 if on a back-to-back vs rested opponent, 1 if rested vs opponent on back-to-back, 0 otherwise)

### Results
The model finished with a log loss of 0.6697 and an accuracy of 59.2%. Both of those were 4th best in the contest. [Final Standings](https://twitter.com/CorsicaHockey/status/983175602470572032)

### Improvements
With all the cumulative and last 10-25 games stats, the model performed poorly for the first ~20 games of the season relative to other models in the contest. One way to improve the model would be to figure out a way to improve early season predictions.
