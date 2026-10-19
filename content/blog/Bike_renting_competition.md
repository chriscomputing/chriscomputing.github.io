+++
title = "Predicting bikeshare demand using machine learning"
description = "XGBoost regression to predict dynamic demand for bikesharing"
slug = "xgboost-bikeshare-predictions"
tags = ["Machine Learning", "Python", "Competition"]
date = 2026-08-16
draft = false
math = false
toc_open = true
+++

In this competition, we used Machine Learning to predict dynamic demand for bikesharing based on a Kaggle dataset. Since the organizers restricted us to regression, we decided to use XGBoost. The small competition was held last year at my university. I competed together with a friend of mine. You can see the code on my [Github](https://github.com/chriscomputing/bikeshare-competition).

## Bikesharing dataset

The dataset was provided by the organizers, but I believe it to be [this](https://www.kaggle.com/c/bike-sharing-demand/data/) dataset from Kaggle. The dataset contains for example information about the temperature, humidity, windspeed, if it was a holiday or workday and time and date. 

### Preprocessing the dataset

We used [seaborn](https://seaborn.pydata.org/), a statistical data visualization library for Python to analyze the dataset. As expected, a correlation matrix showed that `atemp` (appearent temperature) and `temp` are highly correlated, and that `season` and `month` are equally highly correlated. For this reason, we decided to drop `season` and `atemp`.

![Feature correlation matrix using matplotlib and seaborn in Python](/images/posts/bikeshare-comp/corr-matrix.png "Correlation matrix of all features. Blue is low correlation, red is high correlation.")

Some features were continuous data, for example temperature, while others were cyclic (like time  or workingday). Only few were categorical. This is a problem for regression, since it enforces a strict linear relation between these features and the prediction target. A regression will run into problems predicting usage (cnt, shored for count) based on temperature, since both very high and very low temperatures will lead to less usage. Even worse, weekday simply jumps from 7 back to 1 at the beginning of the week, marking a large and sudden shift at midnight. Therefore, we had to one-hot encode many features during preprocessing.

## Using XGBoost regression to predict bikeshare customers

After trying multiple models, the best one was the `XGBRegressor` from [XGBoost](https://xgboost.readthedocs.io/en/stable/), trained on a logratihmic scale. For hyperparameter search, we used the `RandomizedSearchCV` from sklearn across 9 different parameters.  

On the evaluation set, our model scored a Root Mean Squared Error (RMSE) of 37.78, while the Mean Absolute Error (MAE) was 23.27. That is not fantastic, but also not a terrible job. Unfortunately, we did not have time to improve our model further. In the end, our model scored an RMSE of 56.897 on the hidden evaluation set. Which is admittedly even worse, but it still got us **first place** in end. All teams had an unexpected high RMSE on the hidden evaluation set. Maybe it was not very representative of the entire dataset.

## What could be improved

As I already said, we did not have a lot of time for this competition. While being restricted to regression did not help, we could have done a more in-depth analysis on our worst performing predictions.