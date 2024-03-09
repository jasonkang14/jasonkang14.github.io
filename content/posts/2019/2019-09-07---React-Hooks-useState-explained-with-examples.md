---
title: React Hooks[useState()] - setState has become easier
date: "2019-09-07T14:27:37.121Z"
template: "post"
draft: false
slug: "/posts/react/react-hooks-useState-explained-with-examples"
category: "react"
tags:
  - "React"

description: "How to use the state hook in React"
---

`React` has come out with `hooks` while ago, but it is still not that common to use hooks even though it helps a lot with program performance. I have already written about `useEffect()` in a different [post](https://jasonkang14.github.io/posts/React-Hooks-useEffect-update-when-you-want)

I personally find `hooks` a lot easier. `useEffect()` is a lot easier to use than `componentDidUpdate()`. I faced so many infinite loops while trying to use `componentDidUpdate()`, so ended up using different lifecycle methods such as [getDerivedStateFromProps](https://jasonkang14.github.io/posts/react/react-life-cycle-get-derived-state-from-props-with-mobx)

When you change a state in a react component, you have to `setState` like below;

```typescript
increasePrice = () => {
  this.setState({
    priceChange: !this.state.priceChange,
    price: this.state.price + 2000,
  });
};
```

By chaning the state of price, I was trying to update the total price that appears in a component.

If I were to use `useState()` instead, I can change the above method/function like below

```typescript
const [priceChange, setPriceChange] = useState(true);
const [price, setPrice] = useState(0);

increasePrice = () => {
  setPrieChange(!priceChange);
  setPrice(price + 2000);
};
```

The value inside `useState(value)` represents the intial value of the state. By using `useState`, there is no need for destructuring, and the code itself becomes a lot more straight-forward. The downside is that if you have a lot of states to manage within a single component, you have to declare all of them like I did in the second example.

`React Hooks` allows you to write your React project as a function rather than a class, which could increase the performance exponentially especially if your project is huge. I have actually did the entire project with function components without having to use class.
