---
title: React Native - Basic Setup with MobX
date: "2019-07-29T22:27:37.121Z"
template: "post"
draft: false
slug: "/posts/reactnative/basic-setup"
category: "React Native"
tags:
  - "React Native"

description: "How to set up a React Native project with MobX"
---

Before you start following my blog to set up your first `React Native` project, please check out the versions that I have installed.

```
"react-native": "^0.60.4"
"mobx": "^5.13.0",
"mobx-react": "^6.1.1",
"@babel/plugin-proposal-class-properties": "^7.5.5",
"@babel/plugin-proposal-decorators": "^7.4.4",
"@babel/plugin-transform-flow-strip-types": "^7.4.4",
```

I did not use `expo` to set up my `react native` project, so it could be a little different in the beginning, but I am sure the rest of the process is the same. First, follow the instruction given in the [react native document](https://facebook.github.io/react-native/docs/getting-started).

And then you have to install `mobx` and `mobx-react`. But honestly, I feel like you don't need to install `mobx` for a `react-native` or `react` project. Maybe you only need to install `mobx-react`. I will check on it and update this post if necessary. Regardless, install `mobx` and `mobx-react` for now.
`npm install mobx mobx-react --save`

And then you have to install `babel`. The problem that I ran into while installing setting up this project was that I have to install `@babel/plugin-proposal-decorators` instead of `babel-plugin-transform-decorators-legacy`. And you also have to install `@babel/plugin-proposal-class-properties` and `@babel/plugin-transform-flow-strip-types`.

`npm install @babel/plugin-proposal-decorators @babel/plugin-proposal-class-properties @babel/plugin-transform-flow-strip-types --save`

And then you have to change `babel.config.js` like below;

```typescript
module.exports = {
  presets: ["module:metro-react-native-babel-preset"],
  plugins: [
    ["@babel/plugin-transform-flow-strip-types"],
    ["@babel/plugin-proposal-decorators", { legacy: true }],
    ["@babel/plugin-proposal-class-properties", { loose: true }],
  ],
};
```

when you do `react-native run-ios`, you should be good to go.
