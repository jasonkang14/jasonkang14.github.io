---
title: React - How to prevent useEffect on initial render
date: "2020-07-10T10:53:37.121Z"
template: "post"
draft: false
slug: "/react/how-to-prevent-useeffect-on-initial-render"
category: "REACT"
tags:
  - "React"

description: "How to prevent useEffect on initial render"
---

I have a state called `loginStatus` which should be false if login fails or true if successful. I set the initial value to false because technically I have not received a 200 response from the server from my login request. And I set `loginStatus` as a dependency in my useEffect like below

```typescript
import React, {FunctionComponent, useEffect, useState} from 'react';

type LoginScreenProps

const LoginScreen: FunctionComponent<LoginScreenProps> = () => {
  const [loginStatus,  setLoginStatus] = useState(false)

  ...

  useEffect(() => {
    if (loginStatus) {
      setLoginError(!loginError);
    }
  }, [loginStatus]);

  ...
}
```

I first thought that if I set a dependency like above, the effect hook only runs when the dependency changes. However, I found out that every single effecthook runs on initial render. So I was always getting my login error modal on initial render, because `loginStatus === false`. So I needed something in order to prevent the if clause above from running.

So I decided to use a ref to check if the effect hook is running on initial render or not.

```typescript
import React, {FunctionComponent, useEffect, useRef, useState} from 'react';

type LoginScreenProps

const LoginScreen: FunctionComponent<LoginScreenProps> = () => {
  const [loginStatus,  setLoginStatus] = useState(false)
  const mounted = useRef(false)

  ...
  useEffect(() => {
    mounted.current = true
    return () => {
      mounted.current = false
    }
  }, [])

  useEffect(() => {
    if (mounted.current && loginStatus) {
      setLoginError(!loginError);
    }
  }, [loginStatus]);

  ...
}
```

I thought the if clause wouldn't run becuase the initial value of the ref is `false`. However, the if clause still ran because the first effect hook without a dependency runs before the effect hook with the `loginStatus` dependency. So I had to change my code like below.

```typescript
import React, {FunctionComponent, useEffect, useRef, useState} from 'react';

type LoginScreenProps

const LoginScreen: FunctionComponent<LoginScreenProps> = () => {
  const [loginStatus,  setLoginStatus] = useState(false)
  const mounted = useRef(false)

  ...

  useEffect(() => {
    if (mounted.current && loginStatus) {
      setLoginError(!loginError);
    }
  }, [loginStatus]);

  useEffect(() => {
    mounted.current = true
    return () => {
      mounted.current = false
    }
  }, [])

  ...
}
```

In this way, the effect hook with the `loginStatus` dependency runs before the hook the dependency of an empty array. So I prevented the email error modal from appearing on the initial render.
