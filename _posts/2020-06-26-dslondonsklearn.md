---
title: "Data Science London + Scikit Learn - Kaggle Competition"
date: 2020-06-26
tags: [Python, Machine Learning, Kaggle, Data Science, dimensionality reduction, XGBoost, PCA]
header:
  image: "/images/londonsklearn/title.jpg"
excerpt: ""
---
> Written by Manan Jhaveri and Devanshu Ramaiya

## Introduction
In this blogpost, we will talk about an interesting Kaggle competition dataset: [Data Science London + Scikit Learn](https://www.kaggle.com/c/data-science-london-scikit-learn).
It is a synthetic data set of 40 features, representing objects from two classes (labeled as 0 or 1). The training set has 1000 samples and the testing set has 9000.
As it is a synthetic dataset, there is no  particular information about the features. We have to directly take a dive into preparing the data and training a model.

## Importing packages and loading the dataset

``` python
import pandas as pd
import numpy as np
from sklearn.decomposition import PCA
from sklearn.preprocessing import Normalizer
from sklearn.metrics import accuracy_score
from sklearn.model_selection import RandomizedSearchCV, GridSearchCV
from sklearn.pipeline import Pipeline

train = pd.read_csv("train.csv")
test = pd.read_csv("test.csv")
y = pd.read_csv("trainLabels.csv")
```

## Understanding the data
If we use the .head() and .describe(), there isn't any useful information for us to find.
<img src="{{ site.url }}{{ site.baseurl }}/images/londonsklearn/1.PNG" alt="Understanding data">


Also, if there is no substantial correlation between the features.
<img src="{{ site.url }}{{ site.baseurl }}/images/londonsklearn/2.PNG" alt="correlation">


As mentioned before, we should dive into data preparation and training models.

## Preparing the dataset

We have 40 features with low correlation. The ideal 1st step for preparing the data should be to apply a dimensionality reduction technique to decompose the data to a smaller dimension.

We will use Principal Component Analysis. First, we will have to scale the data and find appropriate value for n_components hyperparameter for PCA.
We will plot cumulative explained variance vs n_components, and select the value of n_components which preserves 80% variance.

```python

scaler = Normalizer()
scaled = scaler.fit_transform(train)
pca = PCA()
pca.fit(scaled)

exp_variance = pca.explained_variance_ratio_
cum_exp_variance = np.cumsum(exp_variance)

import matplotlib.pyplot as plt
fig, ax = plt.subplots()
ax.plot(range(pca.n_components_), cum_exp_variance)
ax.axhline(y=0.8, linestyle='--')
ax.set_xlabel("n_components")
ax.set_ylabel("cumulative explained variance")
```
<img src="{{ site.url }}{{ site.baseurl }}/images/londonsklearn/3.PNG" alt="PCA n_components">

We can see, that at n_components = 12, 80% variance is preserved.
Let's transform the data using a pipeline.

```python
n_components = 12
pipeline = Pipeline([("n", StandardScaler()), ("pca", PCA(n_components))])
pca_train = pipeline.fit_transform(train)
pca_test = pipeline.transform(test)
```

28 features are still a lot. Some other technique to further reduce the dimensions also has to be used.
We will try Bayesian Gaussian Mixture models:

```python
from sklearn.mixture import GaussianMixture

X_combined = np.r_[pca_train, pca_test]

lowest_bic = np.infty
bic = []
n_components_range = range(1, 7)
cv_types = ['spherical', 'tied', 'diag', 'full']
for cv_type in cv_types:
    for n_components in n_components_range:
        gmm = GaussianMixture(n_components=n_components,covariance_type=cv_type)
        gmm.fit(X_combined)
        bic.append(gmm.aic(X_combined))
        if bic[-1] < lowest_bic:
            lowest_bic = bic[-1]
            best_gmm = gmm

best_gmm.fit(X_combined)
gmm_train = best_gmm.predict_proba(pca_train)
gmm_test = best_gmm.predict_proba(pca_test)
```  
Now, We will start training a model.
We will use XGBoost. It is very popular in competitions and many winning entries came using XGBoost. It uses different simple models and combines results to make predictions.

Before that, we will split the dataset for training and testing.

```python
from sklearn.model_selection import train_test_split
train_X, test_X, train_y, test_y = train_test_split(pca_train, y.Label, test_size = 0.3, random_state = 10)
```
Next step is to use GridSearchCV to find the right hyperparameters for our model and check the accuracy of the model on the test set.

```python
from xgboost import XGBClassifier
xgb = XGBClassifier()
params = {"eta": [0.01, 0.1, 1], "max_depth": [10, 15, 20], "gamma": [0.1, 1, 10]}
xgb_gs = GridSearchCV(xgb, params, cv = 3)
xgb_gs.fit(train_X, train_y)

```
We get the following as best paramaeters:
###### 'eta': 0.01, 'gamma': 0.1, 'max_depth': 15, 'subsample': 0.5

And the accuracy on the test set is 92%. Not bad.
We will train an XGBoost model again on the entire train dataset now and make predictions on the test data, save the predictions in a file and submit it!

```python
xgb = XGBClassifier(eta = 0.01, gamma = 0.1, max_depth = 15, subsample = 0.5)
xgb.fit(gmm_train, y.iloc[:,0])

result = xgb.predict(gmm_test)

#saving in a file
df = pd.DataFrame({"Id": list(np.arange(1,9001, 1)), "Solution": result})
df.set_index("Id", inplace = True)
```

###### Wonderful!!! We have achieved an accuracy of 98.025!!!
<img src="{{ site.url }}{{ site.baseurl }}/images/londonsklearn/final.PNG" alt="Result">

Hope you all learned something from this blog. Try these methods, tweak and explore different hyperparameters, train different models and compare them and you might find a better result.
