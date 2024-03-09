---
title: Functional Programming with React [2]
date: "2020-03-30T12:53:37.121Z"
template: "post"
draft: false
slug: "/react/functional-programming-with-react-part-two"
category: "React"
tags:
  - "React"

description: "Understanding how functional programming works with React"
---

Now I have a general understanding about functional prograamming, I am going to talk about why understanding functional programming is import when working on a React project.

### React Component

- Similar to a pure function which has a return value, a `React` component must have an output via `render()`
- A component is like a function with returns an output based on its input like a pure function

React components are pure components, but more importantly, the `render` function inside a react componenet must be a pure function which always returns the same output if the input, which is `state` in this case, is the same.

### React Props

- React props are immutable
- Since React props are the input arguments of a component, their immutability avoids side effects

In functional programming, functions are not supposed to change its input values. Similarly, a React component cannot change its props

### Higher Order Components

- Higher Order Components(HOC) in React is similar to Higher Order Function in functional prograaming

When a higher order function takes an argument and returns a function, a higher order component takes an arguments and returns a component. Simple examples are `withRouter` in React-router, which is used in order to transfer a user to another webpage, or `connect` in Redux, which connects a React component to a Redux store.
