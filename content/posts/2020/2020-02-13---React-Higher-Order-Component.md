---
title: React - Higher Order Component
date: "2020-02-13T14:53:37.121Z"
template: "post"
draft: false
slug: "/react/higher-order-component"
category: "React"
tags:
  - "React"

description: "React higher order component explained"
---

# TL;DR

A higher-order component is a function that takes a component and returns a new component.

# Explanation

A HOC is a wrapper that wraps a component and returns another component. The wrapper will allow you to use the same code in multiple components so that you don't have to rewrite same codes--or similar codes--over and over again.

For example, if you have a component that deals with a blog post with comments and another component with a video with subscriptions, you can reuse your codes for comments for both components without having to write a code for comments for both components. You can reuse your code for comments by wrapping such components. Because comments and subscriptions would have a similar structure.

```javascript
const BlogWithComments = withComments(CommentList, (DataSource) =>
  DataSource.getComments()
);

const VideoWithSubscription = withSubscription(Video, (DataSource, props) =>
  DataSource.getVideo(props.id)
);
```

According to the [official website](https://reactjs.org/docs/higher-order-components.html), libraries that allow you to maintain global states like `Redux` and `MobX` are somewhat higher order components. You wrap your components with `connect()` or `provider()` so that you can reuse your codes in `store` in various components.

React also provides `Context API` so that you don't have to use a third-party library. I will post about `Context-API` after using it in my personal project later on.
