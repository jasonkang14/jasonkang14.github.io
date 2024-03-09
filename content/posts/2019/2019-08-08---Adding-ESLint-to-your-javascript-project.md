---
title: JavaScript - Adding ESLint
date: "2019-08-08T13:27:37.121Z"
template: "post"
draft: false
slug: "/posts/Adding-ESLint-to-Your-JavaScript-Project"
category: "javascript"
tags:
  - "JavaScript"

description: "How to add ESLint to your JavaScript Project"
---

I am using `VS Code` as an editor when I work on a `JavaScirpt` project. You must first install `ESLint` Plugin from Extensions.

However, when you install `ESLint` using the plugin, it does not really do much. It throws a lot of erros with red and green underlines that don't really make sense. I have just found out a very easy way to add `ESLint` to your project using command lines from your terminal

First, install `ESLint` <br>
`npm install -g eslint` <br>

then you gotta `init` <br>
`eslint --init` <br>

then your terminal will ask some questions. you have to answer them by using arrows--or j and k if you are familiar with vim--to choose your `ESLint` options

![ESLint questions answered](https://scontent-icn1-1.xx.fbcdn.net/v/t1.0-9/69036743_10219574424732121_8346729998689763328_o.jpg?_nc_cat=104&_nc_oc=AQlGHZSxPCwUYvh0A6vl5c82DOBPdNhCtjc-gnIvobnUH9k8hr2SfOavYSB_qrpM8-c&_nc_ht=scontent-icn1-1.xx&oh=6ea0750cc536dbe38dc60a85d4a310c0&oe=5DE9F755)

After answering all the questions, it even asks you if you want to install all the necessary dependencies. You just gotta enter `Y`, and all the dependencies will be installed. And it automatically creates an `.eslintrc.js` file like this

```typescript
module.exports = {
  env: {
    browser: true,
    es6: true,
  },
  extends: ["airbnb"],
  globals: {
    Atomics: "readonly",
    SharedArrayBuffer: "readonly",
  },
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 2018,
    sourceType: "module",
  },
  plugins: ["react"],
  rules: {},
};
```
