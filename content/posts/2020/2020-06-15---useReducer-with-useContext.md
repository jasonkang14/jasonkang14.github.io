---
title: React Native - useReducer with useContext + TypeScript
date: "2020-06-15T10:53:37.121Z"
template: "post"
draft: false
slug: "/react-native/usereducer-with-usecontext"
category: "REACT"
tags:
  - "React"

description: "How to use useReducer with useContext"
---

[Redux](https://redux.js.org/) has a `Provider`, which allows you to access the **redux store** from all the components. Similar to that, `useContext` allows you to create a `Context Provider` in order for you to have an access to **context** from all the components.

Here I am going to create `UserInfoContext`, which has information about a user's name and age. And also implement a `reducer` to change user information.

First, you have to `createContext`

```javascript
// UserInfoContext.tsx

import React, { createContext } from "react";

type ActionProps = {
  type: string,
  payload: {
    age: number,
  },
};

const UserInfoContext = createContext({
  userInfo: {
    username: "Jason Kang",
    age: 20,
  },
  changeUserInfo: (action: ActionProps) => {},
});

export default UserInfoContext;
```

Now you create a `reducer` to change user information

```javascript
// UserInfoReducer.tsx
type ActionProps = {
  type: string,
  payload: {
    name: string
    age: number
  }
}

type IUserInfo = {
  username: string,
  age: number
}

const userInfoReducer = (state: IUserInfo, action: ActionProps) => {
  switch(action.type) {
    case 'CHANGE_USERINFO':
      const {username} = action.payload;
      return {
        ...state,
        username
      }

    case 'CHANGE_AGE':
      const {age} = action.payload;
      return {
        ...state,
        age
      }

    default:
      throw new Error(`Unhandled action type: ${action.type}`);
  }
}
```

And then you import this context in your `App.tsx` to create a **context provider**. And you also need to `useReducer` and pass it to all the components using the provider so that you can change userinfo in every single component.

```typescript
// App.tsx

import React, { useReducer } from "react";
import ComponentA from "~/ComponentA";
import ComponentB from "~/ComponentB";
import ComponentC from "~/ComponentC";
import UserInfoContext from "~contexts/UserInfoContext";
import UserInfoReducer from "~reducers/UserInfoReducer";

type IUserInfo = {
  username: string;
  age: number;
};

const App = () => {
  const initialUserInfo: IUserInfo = {
    username: "Jason Kang",
    age: 20,
  };

  const [state, dispatch] = useReducer(UserInfoReducer, initialUserInfo);

  const userInfoValue = {
    initialUserInfo,
    changeUserInfo: (type: string, payload: IUserInfo) => {
      dispatch({ type, payload });
    },
  };

  return (
    <UserInfoContext.Provider value={userInfoValue}>
      <ComponentA />
      <ComponentB />
      <ComponentC />
    </UserInfoContext.Provider>
  );
};

export default App;
```

Now you can `useContext` in all the components to change user information

```typescript
// ComponentA.tsx

import React, {FunctionComponent, useContext} from 'react'
import {TouchableOpacity, Text} from 'react-native

import UserInfoContext from '~contexts/UserInfoContext';


type ComponentAProps = {}

const ComponentA: FunctionComponent<ComponentAProps> = props => {
  const userInfoContext = useContext(UserInfoContext)

  const changeName = (): void => {
    userInfoContext.changeUserInfo({
      type: 'CHANGE_NAME',
      payload: {
        name: 'James Kang'
      }
    })
  }

  const changeAge = (): void => {
    userInfoContext.changeUserInfo({
      type: 'CHANGE_AGE',
      payload: {
        name: 30
      }
    })
  }

  return (
    <TouchableOpacity onPress={changeName}>
      <Text>Change Name</Text>
    </TouchableOpacity>

    <TouchableOpacity onPress={changeAge}>
      <Text>Change Age</Text>
    </TouchableOpacity>
  )
}
```

Now you can change username and age in every single component
