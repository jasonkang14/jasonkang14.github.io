---
title: Machine Learning - Linear Regression with Multiple Variables
date: "2020-10-29T21:53:37.121Z"
template: "post"
draft: false
slug: "/machine-learning/linear-regression-with-multiple-variables"
category: "MACHINELEARNING"
tags:
  - "Machine Learning"

description: "Coursera Machine Learning course: Linear Regression with Multiple Variables"
---

# Machine Learning - Linear Regression with Multiple Variables

## Multiple Features

1. When more than one feature affects the output
2. In the example of estimating house price, size, # of bedrooms, # of floors may affect the price
3. features are denoted with x<sub>1</sub>, x<sub>2</sub>

   - n = # of features
   - x<sup>(i)</sup> = input of i<sup>th</sup> training example : vector
   - x<sub>j</sub><sup>(i)</sup> = value of feature j in i<sup>th</sup> training example : the value

4. Since there are more than one features, the hypothesis is going to get longer in order to include more terms/features

   - theta represents parameter: I think this is like weight
   - x<sub>0</sub> is assumed to be 1
   - x and theta are both (n+1) dimensional vectors
   - h<sub>θ</sub>(x) = θ<sup>T</sup>x

## Gradient Descent for Multiple Variables

- Pretty much the same as single variable, but you gotta multiply x<sub>j</sub><sup>(i)</sup> for each theta

## Feature Scaling

1. Making features on a similar scale

   - if the ranges are different between features, the contour plot could be skewed
   - divide the features by the largest possible value to make the contour plot to look like a circle as much as possible
   - because all the features lay within -1 to 1 range

2. Mean Normalization
   - subtract the feature by the mean of the values and divide that by the largest value
   - x<sub>i</sub> = (x<sub>i</sub> - μ<sub>i</sub>) / largest value
   - you could also divide the numerator by the difference between the smallest and the largest

## Learning Rate

1. Tips to ensure learning rate is working correctly

   - make sure that the cost function is decreasing as you iterate
   - J(θ) must decrease after every iteration
   - declare convergence if J(θ) decreases by less than 10<sup>-3</sup> in one iteration

2. Use smaller learning rate if not converging
   - a too large learning rate may overshoot and it may diverge instead of converging
   - a sufficiently small learning rate is the best
   - but if it is too small, it will take too long to converge

## Features and Polynomial Regression

1. combine two features like frontage and depth and multiply them to be an area
   - define a new feature by combining them to reduce the number of parameters/features
   - your cost function may become simpler

## Normal Equation

1. Method to solve for θ analytically instead of doing iterations

   - polynomial of θ, and take a derivative with respect to θ
   - take partial derivatives with respect to θ and set them to zero to find the θ<sub>i</sub>

2. θ = (X<sup>T</sup>X)<sup>-1</sup>X<sup>T</sup>y
3. Slow if n is really large becuase you need to take inverse of the matrix X
4. won't work if (X<sup>T</sup>X) is not inversible
   - you have to delete some features or use regularization
