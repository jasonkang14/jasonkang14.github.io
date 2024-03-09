---
title: Babel - What is Babel and what does it do?
date: "2019-12-03T14:53:37.121Z"
template: "post"
draft: false
slug: "/posts/babel/what-is-babel"
category: "Babel"
tags:
  - "Babel"

description: "Basics of Babel and its role in web development"
---

# TL;DR

Babel allows new features of JavaScript to work with old browsers.

# Explanation

According to the [official document](https://babeljs.io/docs/en/), Babel is a `toolchain that is mainly used to convert ECMAScript 2015+` code into a backwards compatible version of JavaScript in current and older browsers or environments.

For example, if a developer uses an arrow function, Babel transforms the arrow function to its old format so that the arrow function can be compatible with web browsers

```
// ES6
[1,2,3].map((n) => n+1);

// ES5 : Babel-transformed version
[1,2,3].map(function (n) {
    return n+1;
})

```
