---
title: React - Context Explained
date: "2019-08-27T22:27:37.121Z"
template: "post"
draft: false
slug: "/posts/react/react-advanced-guide-context-explained"
category: "react"
tags:
  - "React"

description: "React Context Explained"
---

##This post is a summarized version of [react doc](https://reactjs.org/docs/context.html)

1. When to use it
   According to the official React docs, `Context` is designed to share data that can be considered “global” for a tree of React components.

   - you use `Context` when you have to pass `props` to a grandchild-of-a-grandchild-of-a-grandchild component

   - so `Context` can be used instead of `Redux` or `MobX` unless you are familiar with them or you choose to use an outside library. I am familiar with `MobX`, so I tried my best to compare the two, so hope that helps.

   - If you don't use `Context` when you pass down `props` to a deeply nexted component, you can just create a separate component for the specific `props` instead. So make sure that using `Context` is more efficient than any other methods that you can think of.

2. How to use it
   `const MyContext = React.createContext(defaultValue);`

   - This creates a `Context` object. And when React renders, the object will read the current context value from the closest matching `Provider` above it in the tree.
   - This is like an `observable` in `MobX`. The value/props that could be changed depending on events and its changes should be monitored globally.
   - The `defaultValue` argument is only used when a component does not have a matching `Provider` above it in the tree. So `defaultValue` does not have to be set/determined all the time.

   `<MyContext.Provider value={/* some value */}>`

   - Every Context object comes with a Provider React component that allows consuming components to subscribe to context changes.

   - so I think `Provider` is something like a `Redux` or `MobX` store that handles events associated with such changes.

   - `Provider` accepts a `value` props and can provide such values to many consumers. This is similar to how many components can access change of `props` or `states` in a `Redux/MobX` store.

   `Class.contextType`

   - lets you consume the nearest current value of that Context type using this.context.
   - in `MobX`, you can determine the type of `observable` when you declare it for the first time. I think this is similar to that.
   - this is associated with `lifecycle` methods, but not to sure how to use it. I think I would have to use it myself in my code to fully understand it.

   `Context.Consumer`

   - A React component that subscribes to context changes. This lets you subscribe to a context within a function component.
   - this is like an `observer` in `MobX`. `Consumer` pretty much represents a component
   - `Consumer` detects change of `props` in `Provider` and have `React` render again.
