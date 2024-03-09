---
title: JavaScript - hoisting
date: "2019-11-29T17:27:37.121Z"
template: "post"
draft: false
slug: "/posts/javascript/hoisting"
category: "JavaScript"
tags:
  - "JavaScript"

description: "JavaScript hoisting"
---

With JavaScript hoisting, or since JavaScript hoist variables, variables can be used before declaration.

So even if you call a function, before it is declared, you can still get the same result as when you call the function after its declaration.

```javascript
sayHello(); // "hello"

function sayHello() {
  console.log("hello");
}

sayHello(); // "hello"
```

So even if you call a function before its declaration, you won't get undefined.
JavaScript hoist declartaion, but it won't hoist initialization.

```javascript
a = 3;
console.log(a); //  3
var a;

console.log(b); // undefined
var b = 3;
```
