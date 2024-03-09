---
title: JavaScript - this, apply, call, bind, arrow function
date: "2019-11-30T12:53:37.121Z"
template: "post"
draft: false
slug: "/posts/javascript/this-bind-apply-call"
category: "JavaScript"
tags:
  - "JavaScript"

description: "Basics of JavaScript: this, apply, call, bind, arrow function"
---

1. `this`
   - `this` is defined/determined by how a function is called.
   - Global Context: `this` refers to the global object (`window` in browser, `global` in node.js)
   - Functional Context:
     - if not in `strict mode`, `this` refers to the global object
     - if in `strict mode`, `this` is undefined

```javascript
function f2() {
  "use strict"; // see strict mode
  return this;
}

f2() === undefined; // true
```

- the above is because function `f2` is called directly. not as a method or a property of the `window` object
- To set the value of this to a particular value when calling a function, use `call()`, or `apply()`

2. `apply` / `call`

```javascript
var obj = { a: "Custom" };

var a = "Global";

function whatsThis() {
  return this.a;
}

whatsThis(); // 'Global'
whatsThis.call(obj); // 'Custom'
whatsThis.apply(obj); // 'Custom'
```

if you call a function without `call` or `apply`, `this` refers to the `window` object. However, if you call the same function with `call` or `apply`, you set `this` as the `object` with which you are calling the function.

if the value you are passing with `call` or `apply` is not an object, the browser will use the internal method `ToObject()` in order to convert it to an object.

3. `bind`

   - if you call a function using bind like `foo.bind(someObject)` this is the `someObject` regardless of how the function is being called.
   - but you cannot `bind` more than once.

```javascript
function f() {
  return this.a;
}

var g = f.bind({ a: "azerty" });
console.log(g()); // azerty

var h = g.bind({ a: "yoo" }); // bind only works once!
console.log(h()); // azerty
```

4. arrow function

   - if you use an arrow function, you don't have to bind this.
