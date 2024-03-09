---
title: Machine Learning - Neural Networks - Representation
date: "2020-11-04T23:34:37.121Z"
template: "post"
draft: false
slug: "/machine-learning/neural-networks-representation"
category: "MACHINELEARNING"
tags:
  - "Machine Learning"

description: "Coursera Machine Learning course: Neural Networks - Representation"
---

# Neural Networks

## Model Representation

1. input and output

   - x<sub>1</sub>, x<sub>2</sub> are input layers which go through a node and then generate an output layer
   - h<sub>θ</sub>(x) = 1 / (1 + -e<sup>θ<sup>T</sup>x</sup>)
   - x<sub>0</sub> is usually 1, which is also called a bias unit

2. sigmoid activation function

   - g(z) = 1 / (1 + e<sup>-z</sup>)

3. Neural network is a group of neurons put together
   - hidden layers are between the input layer and the output layer
   - weights are also matrix
   - s<sub>j</sub> units in layer j, s<sub>j+1</sub> units in layer j+1
   - the dimension will be s<sub>j+1</sub> \* (s<sub>j</sub> + 1)

## Intuition

1. Non-linear classification
   - if data can be clusterd, try to use a simple representation of a given data set
   - x<sub>1</sub> XOR x<sub>2</sub>: true if either one is true
   - x<sub>1</sub> XNOR x<sub>2</sub> : NOT (x<sub>1</sub> XOR x<sub>2</sub>)
   - x<sub>1</sub> AND x<sub>2</sub>
   - do some calculation and see if the function becomes 1 with x<sub>1</sub> and x<sub>2</sub> are either 0 or 1
