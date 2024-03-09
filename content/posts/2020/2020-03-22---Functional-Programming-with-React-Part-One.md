---
title: Functional Programming with React [1]
date: "2020-03-22T12:53:37.121Z"
template: "post"
draft: false
slug: "/react/functional-programming-with-react-part-one"
category: "React"
tags:
  - "React"

description: "Understanding basic concepts of functional programming"
---

On the very first page of [reactjs.org](https://reactjs.org), it says that `React` is **declarative**, which is a major property of Functional codes.

Before talking about why functional programming is important when building a project with `React`, I want to talk about some basic concepts of functional programming.

### Pure Function

- A pure function always returns the same output if the same inputs are given.
- This means that a pure function does not have any side effects

In order to understand the definition of a pure function, you need to understand what side effects are. A side effect is anything that is observable other than the return value of the function such as `console.log()` as it allows you to observe a state change even when the value is not the return value of a function.

### Shared State

- Shared state is any variable, object, or memory space that exists in a shared scope, or as a property of an object being passed between scopes.
- Shared state must be avoided in functional programming

Share states could be helpful if you need to pass values to different components--like `props` in React. This is dangerous especially when the order of function execution is critical. If function B gets called before function A when a variable in function B must be updated via function A before its execution, it would lead you to a bug.

By avoiding shared state, **the timing and order of function calls do not change the result of calling the function**

### Immutability

- An immutable object is an object that cannot be modified after it is created while a mutable object can be modified.

To be honest, I do not completely understand why this is important with Functional Programming, but apparently the data flow of your project gets messy. I believe this is related to how a `React state` must be treated as **immutable**. And a `React prop` is literally **immutable**. This means that you cannot directly change a state in a React project without using `setState`.

### Higher Order Function

- Functional programming tends to reuse a common set of functional utilities to process data
- This is related to Higher Order Component in React, which can be reused, or used to wrap another components

A higher order function is a function whith takes a function as an argument and returns a function, or both the argument and a function.
