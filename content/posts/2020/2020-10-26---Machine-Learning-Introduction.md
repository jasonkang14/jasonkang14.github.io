---
title: Machine Learning - Introduction
date: "2020-10-26T10:53:37.121Z"
template: "post"
draft: false
slug: "/machine-learning/introduction"
category: "MACHINELEARNING"
tags:
  - "Machine Learning"

description: "Coursera Machine Learning course: Introduction"
---

# Machine Learning - Introduction

## TL;DR

- Teaching a machine to learn its task without being explicitly programmed
  - The machine is practicing/learning on its own to improve its performance.

## Supervised Learning

1.  right answers are given for a data set

    - so do your best to produce a right answer for another input

2.  Types of supervised learning:

    - Regression: Predict continuous valued output

      - something like house prices.

    - Classification: Discrete answers

      - either 0 or 1 : true or false
        - could be also 0, 1, 2, 3
      - what is the probability that the input turns out to be true or false?

      - if you need to consider more than one parameter, the graph may look different

## Unsupervised Learning

1. No right answers given

   - no labeling for a data.
   - don't know if it's true or false
   - but there are some sorts of clusters: you could organize them

2. something like organizing news.

   - not given how each article is related, but Google somehow organizes them by headline
   - you have to find the relationship between/among the given data set

## Model Representation

1. Notation

   - training set: a dataset given to train a model
   - m : # of training examples
   - x: input variables/features
   - y: output variable/ target variable
   - (x<sup>i</sup>, y<sup>i</sup>) = ith training example (ith row from the training set table)

2. Training set -> Fed into learning algorithm -> hypothesis(h)
   - hypothesis takes an input and produce an output
   - h maps from x's to y's
   - how to represent h ?
   - h<sub>θ</sub>(x) = h(x) = θ<sub>0</sub> + θ<sub>1</sub>x (for a linear function, think of it like a f(x))

## Cost Function

1. Measures the performance of a machine learning model: the goal is to find thetas that minimize the cost function

   - θ<sub>0</sub> and θ<sub>1</sub> are parameters
   - x<sub>1</sub> and x<sub>2</sub> are features
   - choose the best θ<sub>0</sub> and θ<sub>1</sub> to make it close to y as much as possible
   - so minimize (1/2m) \* SIGMA ((h<sub>θ</sub>(x)-y)<sup>2</sup>)(sum of these is the cost function I think)

2. Finding θ<sub>1</sub> which minimizes J(θ) when θ<sub>0</sub> = 0

   - this is where differential equation comes in.
   - minimum when differential = 0

3. Contour plots are used to indicate cost functions

## Gradient Descent

1. A way to minimize the cost function J
2. Start at some random θ<sub>0</sub> and θ<sub>1</sub> and keep changing them **simultaneously** in order to reduce J(θ<sub>0</sub>, θ<sub>1</sub>)

   - θ<sub>j</sub> := θ<sub>j</sub> -α \* derivative_with_respect_to\*θ<sub>j</sub>(J(θ<sub>0</sub>, θ<sub>1</sub>))
   - α is called learning rate
   - temp0 := θ<sub>0</sub> equation
   - temp1 := θ<sub>1</sub> equation
   - θ<sub>0</sub> := temp0
   - θ<sub>1</sub> := temp1
   - same theta's have to be used in order to calculate a temp value

3. Intuition

   - repeat until convergence: until you reach the global minimum
   - think about the equation: θ<sub>j</sub> := θ<sub>j</sub> -α \* derivative_with_respect_to\*θ<sub>j</sub>(J(θ<sub>0</sub>, θ<sub>1</sub>))
   - if the derative is positive, you know that your entire term is getting smaller, and if negative, its getting bigger
   - if the learning rate is too small, gradient descent can be slow. because you would need more iteration
   - if the learning rate is too large, gradient descent can overshoot the minimum -> could even diverge
   - you don't need to decrease the learning rate over time because the derivate term will decrease as you approach the minimum

4. Batch
   - Each step of gradient descent uses all the training examples
   - you are computing all the sums to take the next step in gradient descent
