---
title: Machine Learning - Logistic Regression
date: "2020-11-03T00:34:37.121Z"
template: "post"
draft: false
slug: "/machine-learning/logistic-regression"
category: "MACHINELEARNING"
tags:
  - "Machine Learning"

description: "Coursera Machine Learning course: Logistic Regression"
---

# Logistic Regression

## Classification and Representation

### Classification

1. yes or no question where outputs are discrete

   - 0: negative class (benign tumor)
   - 1:positive class (malignant tumor)
   - there could be multi-class classifiction where there are more than two possible outputs

2. You could use this: h<sub>θ</sub>(x) = θ<sup>T</sup>x
   - if h<sub>θ</sub>(x) = θ<sup>T</sup>x > 0.5, y = 1 (0.5 is the threshold)
   - if h<sub>θ</sub>(x) = θ<sup>T</sup>x < 0.5, y = 0
   - but what happens if the input range increases? : if the threshold remains the same, some cases that could be benign are now considered to be malignant
   - but in some cases, y could be greater than 1 or smaller than 0
   - logistic regression ensures that 0 < h<sub>θ</sub>(x) < 1

### Hypothesis Representation

1.  a function that represents hypothesis that satifies 0 < h<sub>θ</sub>(x) < 1
2.  h<sub>θ</sub>(x) =g(θ<sup>T</sup>x) where g(z) = 1 / (1 + e<sup>-z</sup>)
3.  h<sub>θ</sub>(x) = 1 / (1 + -e<sup>θ<sup>T</sup>x</sup>)

    - sigmoid function / logistic function
    - asymptote at 0 and 1
    - ensures that 0 < h<sub>θ</sub>(x) < 1

4.  h<sub>θ</sub>(x) = estimated probability that y = 1
    - h<sub>θ</sub>(x) = P(y=1 | x;θ)
    - P(y=0 | x;θ) + P(y=1 | x;θ) = 1
    - P(y=0 | x;θ) = 1 - P(y=1 | x;θ)

### Decision Boundary

1. g(z) > 0.5 when z > 0

   - h<sub>θ</sub>(x) =g(θ<sup>T</sup>x) > 0.5 when θ<sup>T</sup>x > 0 where θ<sup>T</sup>x = z

2. h<sub>θ</sub>(x) = g(θ<sub>0</sub> + θ<sub>1</sub>x<sub>1</sub> + θ<sub>2</sub>x<sub>2</sub>)
3. non-linear decision boundaries
   - sometimes your equation may not be linear

## Logistic Regression Model

### Cost Function

1. Linear Regression:

   - J(θ) = Cost(h<sub>θ</sub>x, y) =(1/2)(h<sub>θ</sub>x - y)<sup>2</sup>

2. For Logistic Regression:
   - the cost function ends up non-convex if square is used
   - many local optima may appear
   - log(h<sub>θ</sub>x) if y = 1
   - -log(1-h<sub>θ</sub>x) if y = 0

### Simplified Cost Function and Gradient Descent

1.  J(θ) = Cost(h<sub>θ</sub>x, y) = -y\*log(h<sub>θ</sub>x) -(1-y)log(1-h<sub>θ</sub>x)

### Advanced Optimization

- Conjugate gradient, BFGS, L-BFGS
  - no need to manually pick a learning rate
  - often faster than gradient descent, but more complex

## Regularization

### Problem of Overfitting

- trying your best to fit the training set
- could be like 5 orders

1. underfitting

   - does not fit the training set very well
   - also called high bias

2. overfitting

   - graph looks weird to best fit the data
   - also called high variance

3. How to solve the overfitting problem
   - reduce number of features
   - model selection algorithm
   - regularization: keep all the features but reduce the magnitudes of parameters

### Cost Function

1. Small values for parameters

   - simpler hypothesis
   - less prone to overfitting
   - add a lambda to deal with parameters

2. Regularization term which includes lambda
   - too large of a lambda results in underfitting
