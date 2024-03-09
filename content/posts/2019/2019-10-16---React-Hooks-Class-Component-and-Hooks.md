---
title: React Hooks - Rewriting Class Component Using React Hooks
date: "2019-10-16T23:27:37.121Z"
template: "post"
draft: false
slug: "/posts/react/class-component-and-hooks"
category: "React"
tags:
  - "React"

description: "React.js class component and hooks: "
---

I have already written about how `componentDidMount`, `componentDidUpdate` could be replaced with `useEffect()` in React Hooks in this [post](https://jasonkang14.github.io/posts/React-Hooks-useEffect-update-when-you-want).

And instead of declaring `this.state` inside `constructor()`, you can just use `useState()`, which is supposed to be faster as explained in this [link](https://jasonkang14.github.io/posts/react/react-hooks-useState-explained-with-examples).

There is another hook called `useRef`, which can be used to ensure that a certain function is called only when the component is called for the first time. `useRef` hook is used to remember/store the initial value.

```typescript
const Profile = () => {
  const [name, setName] = useState("jason");
  const isFirstRef = useRef(true);
  if (isFirstRef.current) {
    isFirstRef.current = false;
    callApi();
  }
};
```

In the function above, `isFirstRef` is initially set `true`, which is why the if clause works, and then the function `callApi()` gets called when the Profile component is called for the first time. However, after calling `callApi()`, the value of `isFirstRef` changes to `false`, therefore, the if clause does not work if the component gets called again.

If you need to use the same logic in different components, you can create a custom hook like below.

```typescript
// useOnFirstRender.js
const useOnFirstRender = (func) => {
  const isFirstRef = useRef(true);
  if (isFirstRef.current) {
    isFirstRef.current = false;
    func();
  }
};

// Profile.js

import useOnFirstRender from "path";

const Profile = () => {
  const [name, setName] = useState("jason");
  useOnFirstRender(callApi);
};
```

Changing `getDerivedStateFromProps` is interesting. I posted about `getDerivedStateFromProps` [here](https://jasonkang14.github.io/posts/react/react-life-cycle-get-derived-state-from-props-with-mobx). Instead of comparing a newly received props and a previously stored state as you would do in a class component, you can easily do it using `useState()`

```typescript
const SpeedIndicator = ({ speed }) => {
  const [isFaster, setIsFaster] = useState(false);
  const [prevSpeed, setPrevSpeed] = useState(0);

  if (speed !== prevSpeed) {
    setIsFaster(speed > prevSpeed);
    setPrevSpeed(speed);
  }
};
```

The values of `isFaster` and `prevSpeed` changes right away as the state changes. Apprently this is slightly less efficient than using 1`getDerivedStateFromProps`, but it shouldn't be too bad since it happens before forming DOM.

`useDebounce` provides a debounce, which is a higher-order function that returns another function. To be honest, I am not sure why you would use this.

```typescript
const useDebounce = ({ callback, ms, args }) => {
  useEffect(() => {
    const id = setTimeout(callback, ms);
    return () => clearTimeout(id);
  }, args);
};

const Profile = () => {
  let [name, setName] = useState("");
  let [nameTemp, setNameTemp] = useState("");
  useDebounce({
    callback: () => setName(nameTemp),
    ms: 1000,
    args: [nameTemp],
  });
};
```

It looks like, after a second, the `useDebounce` hook `setName` the `name` state as `nameTemp`. I feel like a custom hook is just creating a separate function that you might use in different components. Personally, I don't see a point of using a `hook` called `useDebounce` when you can just create a simple function that has `setTimeout`. If I get to think otherwise later on, I will post about it as well.

According to `usehooks.com`: When used in conjunction with useEffect, as we do in the recipe below, you can easily ensure that expensive operations like API calls are not executed too frequently.

`useHasMounted` seems useless. and it is not found anywhere.
