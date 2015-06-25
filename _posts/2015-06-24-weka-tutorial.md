---
layout: post
title:  "Weka Tutorial"
date:   2015-06-24
categories: Weka
excerpt: machine learning
comments: true
---

### two types of inputs:
numeric: continous data; nominal: discretion

missing value ? is acceptable.

confusion matrix: diagonal means the correct result, others are missing.

use your model on supplied test set, more options -> check output prediction

simple baseline: always run rules->zeroR, using training sets

cross-validation: same folds number, each time use diff one to test
if data is small, use 10-fold stratified cross-validation. better than choose diff random number, because it reduces the variance of the estimate.

![alt text](https://cloud.githubusercontent.com/assets/5607138/8345509/058c1d00-1aa5-11e5-9360-c3448cd6887e.png)

else use percentage your input data

repeated training and testing: use percentage, expect slight variation results, get it by setting the random number seed. can calculate mean and standard deviation experimentally

rules -> OneR: one attribute does all the work
![alt text](https://cloud.githubusercontent.com/assets/5607138/8345510/059d102e-1aa5-11e5-8398-a168fd68a601.png)
![alt text](https://cloud.githubusercontent.com/assets/5607138/8345514/05a70c5a-1aa5-11e5-9cf6-ef4dead281a7.png)

OneR is always better than ZeroR.

overfitting : simple method cannot solve, a perfect rule....

the oneR classifier minbucketsize sets to 10, the model is overfit.  compare with zeroR

rules apply to training set might be good but when it turns to 10 fold, it goes bad.

Never evaluate on the training set.

divide data into training , test, validation sets.

### naive bayes: use all attributes: all equal and independent

weka will avoid 0 frequency by adding 1 to all the counts.

J48 recursive divide and conquer: select attribute for root node, split, repeat, stop

![alt text](https://cloud.githubusercontent.com/assets/5607138/8345513/05a66534-1aa5-11e5-8767-2e01f24e860e.png)

choose the header with biggest value of bits (entrophy shanon), then continue to split using biggest values (gain)

information gain is maximal if choose ID... not generalize
unprune and prune makes the tree is diff.

### nearest neighbor

![alt text](https://cloud.githubusercontent.com/assets/5607138/8345512/05a56594-1aa5-11e5-82a8-835c7a266135.png)
![alt text](https://cloud.githubusercontent.com/assets/5607138/8345511/05a45f28-1aa5-11e5-8998-8f7f7217b90b.png)

investigate effect of changing k, not k bigger and better. accurate but slow, assume all attrs are equally important, remedies against noisy instances

### class 4 more classifiers

![alt text](https://cloud.githubusercontent.com/assets/5607138/8345515/05a77776-1aa5-11e5-84d2-23993e4773a1.png)

class is numeric, nor nominal.

![alt text](https://cloud.githubusercontent.com/assets/5607138/8345516/05af2750-1aa5-11e5-8f28-436b62949cf2.png)

trees->M5P build non-linear regression models.

change nominal to numeric in two-class problem, Class set to null, and chose filter-> unsupervised->attribute->norminal to binary,  then select attributeindices by clicking the choose, select the index number of your class.

you can add a classification value using linear regression

logistic regression:

![alt text](https://cloud.githubusercontent.com/assets/5607138/8345519/05b714f6-1aa5-11e5-9eb3-189dffa59a4c.png)

logistic regression: return a possibility between 0 and 1.



### support vector machines (SVM): linear decision boundary
logistic regression => linear boundaries

![alt text](https://cloud.githubusercontent.com/assets/5607138/8345517/05b617ae-1aa5-11e5-89b4-a186684c7db0.png)
![alt text](https://cloud.githubusercontent.com/assets/5607138/8345520/05b7aee8-1aa5-11e5-8ea1-b80f1777e7b8.png)
![alt text](https://cloud.githubusercontent.com/assets/5607138/8345521/05b9fec8-1aa5-11e5-8a60-76b690df0526.png)
![alt text](https://cloud.githubusercontent.com/assets/5607138/8345518/05b6db26-1aa5-11e5-9beb-0c16d65beeb1.png)

data mining reveals correlation, not causation. it is experimental science. focus on evaluation and significance.
