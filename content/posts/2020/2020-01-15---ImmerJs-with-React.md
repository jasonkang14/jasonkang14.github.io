---
title: React - ImmerJS and Its Benefits
date: "2020-01-15T23:27:37.121Z"
template: "post"
draft: false
slug: "/posts/react/immerjs-and-its-benefits"
category: "React"
tags:
  - "React"

description: "Basics of ImmerJS explained with its benefits"
---

# TL;DR

You don't have to `slice()` your array in order to `setState()` or `useState()`

# Explanation

According to the [official document](https://immerjs.github.io/immer/docs/introduction), `Immer` allows you to apply all the changes to a temporary **draftState**, which is a `proxy` of the **currentState**. When all the mutations to immutable states are completed, `Immer` produces the **nextState** based on the mutations to the **draftState**

From this explanation, I feel like what it is saying is this. When you have an array as a state, in order for you to change such state, you have to create a temp state using the `slice()` method, and then mutate the **sliced array** and then apply the changes using `setState()` or `useState()`. However, if you use `Immer`, you don't have to do this because `Immer` will allow you to use **draftState** instead of the **sliced array**

By looking at [producer](https://immerjs.github.io/immer/docs/produce), I think I am right

```javascript
import produce from "immer";

const baseState = [
  {
    todo: "Learn typescript",
    done: true,
  },
  {
    todo: "Try immer",
    done: false,
  },
];

const nextState = produce(baseState, (draftState) => {
  draftState.push({ todo: "Tweet about it" });
  draftState[1].done = true;
});
```

by using the `draftState` provided by the `producer`, instead of `slicing` your array, or the `baseState`, you can just push the new item to the array. Like the example below from the [official document](https://immerjs.github.io/immer/docs/example-setstate)

```javascript
onBirthDayClick2 = () => {
  this.setState(
    produce((draft) => {
      draft.user.age += 1;
    })
  );
};
```
