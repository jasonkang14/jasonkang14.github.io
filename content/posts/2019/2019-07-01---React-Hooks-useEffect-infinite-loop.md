---
title: React Hooks[useEffect()] How to avoid infinite loops
date: "2019-07-01T14:27:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Hooks-avoid-infinite-loops"
category: "React"
tags:
  - "React"

description: "How to avoid potential infinite loops of while using useEffect."
---

React came out with a new feature called `React Hooks`. As I work on a group project with three other people at a coding bootcamp, I decided to use this new feature.

Before `React Hooks`, you had to use a `class` in order for your React component to have a state. But with `React Hooks`, your function component can have a state like below.

```typescript
const Party = () => {
    const [joinBtnClicked, setJoinBtnClicked] = useState(true);
```

This is very straight-forward.

1. I made a state called `joinBtnClicked`.
2. `useState()` method allows me to declare the initial `state` of `joinBtnClicked`.
3. I can change the state of `joinBtnClicked` with a method called `setJoinBtnClicked` like below.

```typescript
const displayPartyGenerationField = () => {
  setJoinBtnClicked(false);
};
```

`React Hooks` makes my code a lot more efficient and easier to understand.

I don't need to use lifecycle methods such as `componenetDidMount()` or `componentWillUnmount()` anymore. I just use one method called `useEffect()` which deals with all the lifecycle methods. But I actually ran into a problem while using this method.

```typescript
useEffect(() => {
  fetch(`${ADDRESS}party`, { mode: "cors" }).then((response) => {
    response.json().then((data) => {
      const length = data.length;
      for (let i = 0; i < length; i++) {
        for (let key in data[i]) {
          oldParty[key] = data[i][key];
        }
        oldParty.time = oldParty.time.slice(0, -3);
        oldPartyArr.push(oldParty);
        oldParty = {};
      }
      displayParties();
      setPartyArr(oldPartyArr);
    });
  });
});
```

With the above code, I am retrieving a list of parties that had been formed by other users, so that I can pick a party to join myself. However, when I executed this code, `useEffect()` kept re-rendering while I needed it to render only once.

According to `React.js`,
`If you want to run an effect and clean it up only once (on mount and unmount), you can pass an empty array ([]) as a second argument. This tells React that your effect doesn’t depend on any values from props or state, so it never needs to re-run.`

So, I simply added [] as an argument like below and the problem was solved.

```typescript
useEffect(() => {
  fetch(`${ADDRESS}party`, { mode: "cors" }).then((response) => {
    response.json().then((data) => {
      const length = data.length;
      for (let i = 0; i < length; i++) {
        for (let key in data[i]) {
          oldParty[key] = data[i][key];
        }
        oldParty.time = oldParty.time.slice(0, -3);
        oldPartyArr.push(oldParty);
        oldParty = {};
      }
      displayParties();
      setPartyArr(oldPartyArr);
    });
  });
}, []);
```
