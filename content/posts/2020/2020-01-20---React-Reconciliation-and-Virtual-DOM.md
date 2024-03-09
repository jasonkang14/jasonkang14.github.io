---
title: React - Reconciliation and Virtual DOM
date: "2020-01-20T13:27:37.121Z"
template: "post"
draft: false
slug: "/posts/react/reconciliation-and-virtual-dom"
category: "React"
tags:
  - "React"

description: "React reconciliation to improve efficiency"
---

# TL;DR

React uses Virtual DOM before updating the real DOM, or **reconciles** in order to maximize efficiency

# Explanation

React uses `Virtual DOM`, which is easier and faster to manipulate than `Real DOM`. When a state of a component changes, React updates `Virtual DOM` in order to apply such change then `reconciles`, or compares the `Virtual DOM` with the previous `Virtual DOM` and then only the changes are updated to the `Real DOM`

Sometimes, certain changes call `render()` on the Real DOM even though the `Virtual DOM` does not reflect such changes due to a side effect. This is where `shouldComponentUPdate()` comes in to check if `render()` must be called again. However, such operation could be difficult with deep fields.

But when you use immutable objects like an array as a state, it is difficult to check whether the real DOM should be updated using `shouldComponentUpdate()` since you have to compare every single element in order to check if the state has been changed. You also have to use methods like `slice()` in order to update such states because arrays are immutable.

Using native methods like `Array.slice()` or `Object.assign()` is not too bad, however, if the size of such states are big, it would be way to expensive to re-create such states to simply check whether to update components or not.

This is where `Immer` comes in. By updating the state using `Immer`, you don't have to re-create your states. And all the changes to `draftState` are applied to `newState` when all the mutations are complete. Therefore, the efficiency would increase as well.
