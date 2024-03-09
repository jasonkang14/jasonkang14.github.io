---
title: React Hooks [useEffect()] how to update when you want
date: "2019-07-09T22:27:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Hooks-useEffect-update-when-you-want"
category: "React"
tags:
  - "React"

description: "How to use `useEffect()` instead of `componentDidUpdate()`"
---

I wanted to use `useEffect()` to work like `componentDidUpda()`. Whenever I click a certain button, I wanted the states updated so that React may render components. Last time I talked about how to add `[]` as a second argument to `useEffect()` in order to prevent infinte loops. This time, it is about how to call it when you want.

Check out the code below;

```typescript
useEffect(() => {
  if (joinPartyBtnClick) {
    const getToken = localStorage.getItem("wtw-token");
    fetch(`${ADDRESS}party/join`, {
      method: "post",
      headers: {
        "Content-Type": "application/json",
        Accept: "application/json",
        Authorization: getToken,
      },
      body: JSON.stringify({
        id: idJoinParty,
      }),
    }).then((response) => {
      if (response.status === 200) {
        setParticipationStatus(!participationStatus);
      }
      setJoinPartyBtnClick(!joinPartyBtnClick);
    });
  }
}, [joinPartyBtnClick]);
```

This is very straight-forward. If `joinPartyBtnClick === true`, you run the `fetch` as written. I can set that the `useEffect()` runs whenever the state of `joinPartyBtnClick` changes by adding it as the second argument like above.
