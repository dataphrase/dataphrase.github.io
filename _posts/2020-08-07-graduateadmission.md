---
title: "Graduate Admission prediction"
date: 2020-08-07
tags: [Python, Data Science, Masters]
header:
  image: "/images/adm_predict/cover.jpg"
excerpt: "Predicting admission for master's using various parameters."
---

# Introduction
In today's blog we will use this [dataset](https://www.kaggle.com/mohansacharya/graduate-admissions) where we try to predict graduate admissions based on various parameters like GRE and TOEFL Scores. Let us take a look at the dataset.

<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/data_head.PNG" alt="Data head">

Also, if you check information of the dataset, there are no NULL values here, so we can directly start exploring the data.

# Distribution of GRE and TOEFL Scores
<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/gre_dist.png" alt="GRE Scores distribution">


<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/TOEFL_dist.png" alt="TOEFL Scores distribution">

As you can see from the histogram that they are almost normally distributed. We can confirm that fact by checking their skewness. If skewness is equal to 0, it is normally distributed.

<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/skewness.PNG" alt="Skewness">

In this case skewness is very close to 0, hence it is normally distributed.

# Deciding thresholds

I wanted to convert this into a classification problem that is the target variable should be 0 if not admitted and 1 if admitted. So, for this we need to decide a threshold. Let us look at the distribution of chance of admit.

<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/chance_admit.PNG" alt="Chance of admit">

If we select a threshold of 0.5, we will not get a good classification as the dataset will become very imbalanced. In this case we will have to select a threshold close to 0.7 (I have taken 0.72 which is the mean). Let us now convert chance of admit into a binary classification.

```python
  targets = []

  for i in data['Chance of Admit ']:
      if i >= 0.72:
          targets.append(1)
      else:
          targets.append(0)

  data['Admit'] = targets
```
Here's the ratio of positive class to the negative class -

<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/ratio.PNG" alt="Ratio">

This means that for approximately every 12 people who get admitted, 10 are rejected. Now, we can drop chance of admit and serial no. columns.

```python
  cl_data = data.drop(['Serial No.', "Chance of Admit "], axis=1)
```
Its a good practice to store this in another variable so that its easy to look at the previous stages and go back and check each step.

# Splitting the data
To maintain that ratio of positive class to the negative class, it is preferable to use StratifiedShuffleSplit.

```python
  # split data using Stratified Shuffle split
  from sklearn.model_selection import StratifiedShuffleSplit

  # features
  X = cl_data.drop(['Admit'], axis=1)

  # target variable
  y = cl_data['Admit']

  sss = StratifiedShuffleSplit(n_splits=5, test_size=0.2, random_state=42)

  for train_index, test_index in sss.split(X, y):
      X_train = X.loc[train_index]
      X_test = X.loc[test_index]
      y_train = y.loc[train_index]
      y_test = y.loc[test_index]
  ```
We will keep it as an 80-20 split that is 20% of the data will be present in the test set.

# Cross-validation
Before touching the test set, we can use cross-validation and see how our algorithm performs on the training set. We will use Decision Trees here with max_depth as 2.

```python
  # decision-tree classifier
  from sklearn.tree import DecisionTreeClassifier

  tree_clf = DecisionTreeClassifier(max_depth=2)
```
We can check the performance using scikit-learn's cross_val_score function.

```python
  from sklearn.model_selection import cross_val_score

  print(cross_val_score(tree_clf, X_train, y_train, cv=5, scoring="accuracy"))
```
<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/c_val_score.PNG" alt="cross_val_score">

We have an accuracy ~80% which is pretty good to start with. But let's not just look at accuracy and decide whether our model is actually good or not. In this case, we will also look at precision and recall. To check these two, we need predictions, but since we have not used the test set, we will generate predictions on the training set itself using scikit-learn's cross_val_predict function.

```python
  from sklearn.metrics import precision_score, recall_score

  print(precision_score(y_train, y_train_pred))

  print(recall_score(y_train, y_train_pred))
```
This is what we get -

<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/prre.PNG" alt="precision & recall scores">

So, whenever there is a 1, that is a student will get admitted, our model is correct 84.71% of the time. More over it predicts 76% of the 1s. According to me, our model should be more accurate in terms of telling a student whether he can get a university or he cannot (high precision). So, our model is performing well in that sense. Also, note that if precision is increased, recall will decrease, this is called the Precision/Recall trade-off and as data scientists, we need to decide what is the correct way to evaluate our model.

# Decision Trees
We could also look at various values of max_depth and using cross_val_score we can compare the best accuracies and see which is the best in terms of max_depth.

```python
  # generate mean training accuracies for different depths
  depths = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

  for i in depths:
      print("Depth =", i)
      clf = DecisionTreeClassifier(max_depth=i)
      scores = cross_val_score(clf, X_train, y_train, cv=5)
      print(f"Mean acc. +/- sd => {np.mean(scores)} +/- {np.std(scores)}")
```
The cross_val_score function gives us a list, so, we can calculate average score and check which one is the best. Here, score is mean(scores) +/- standard_deviation(scores). This is what we get -

<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/depths.PNG" alt="Depths">

According to this, max_depth = 4 produces the best result in terms of accuracy. Hence let us make changes in our model and visualize how our decision tree looks like, with max_depth set to 4!

```python
  tree_clf = DecisionTreeClassifier(max_depth=4)
  tree_clf.fit(X_train, y_train)

  # visualizing decision tree
  from sklearn.tree import export_graphviz

  export_graphviz(
          tree_clf,
          out_file="adm_tree.dot",
          feature_names =list(X_train.columns),
          class_names = ['Not admit', 'Admit'],
          rounded= True,
          filled = True
          )
```

Using the export_graphviz function, we can visualize how a decision tree looks like. (This will produce a file with ".dot" extension, you need to change to png format to look at it in a better way)

<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/adm_tree.png" alt="Tree">

A few things to notice, nobody would expect not to get an admit when you look at the right side, where a student has a GRE score greater than 318.25. Concepts like Gini and dominating features will be covered deeply in the next blog, so stay tuned for that!

Its finally time to test our model!

```python
  from sklearn.metrics import precision_score, recall_score
  from sklearn.metrics import accuracy_score

  # finally let's look at how it performed on the test set
  y_pred = tree_clf.predict(X_test)

  print("Accuracy: ", accuracy_score(y_test, y_pred))

  print("Precision:", precision_score(y_test, y_pred))

  print("Recall: ", recall_score(y_test, y_pred))
```

The results are -

<img src="{{ site.url }}{{ site.baseurl }}/images/adm_predict/result.PNG" alt="Result">

Even though the accuracy is 81.25%, our precision is very good! We can improve the model in a lot of ways, using gridsearch for tuning the hyperparameters or using a completely different algorithm. For now, this is it, I hope you learned something new today and we'll be back next time to explain decision trees in more depth!
