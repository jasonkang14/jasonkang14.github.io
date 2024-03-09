---
title: Callback and Promise in JavaScript
date: "2019-08-21T22:27:37.121Z"
template: "post"
draft: false
slug: "/posts/javascript/Callback-and-Promise-in-JavaScript"
category: "javascript"
tags:
  - "JavaScript"

description: "Asynchronous Programming in JavaScript explained"
---

### Synchronous vs Asynchronous

- In synchronous programming, things happen one at a time. If there is a function that takes a while to finish, nothing will happen until that function is finished running.

- In asynchronous programming, a lot of things happen at the same time. Even if there is a function that takes a while to finish, the program continues to run.

- JavaScript is single-threaded, therefore, it can do only one thing at a time. Therefore, asynchronous programming helps JavaScript programs to do mutliple things at the same time while waiting for other functions to finish running.

## There are things like `callback`, `promise`, and `generator` that help JavaScript to run asynchronously.

### 1. Callback

- Simply put, `callback` is a function to be called later. It is used as a property of an object, or a parameter that you pass to another function.
- `callback` can access the scope of where it gets called.
- `error-first-callback`: using an error object as the first parameter of a `callback` in order to check if there is an error associated with the `callback`
  - if the error is either `null` or `undefined`, there is no error associated with the `callback` But if there is an error, you must return the error, otherwise, it will get stuck there.
  - if you have a lot of asynchronous codes, there is a higher change of bugs, and it will be difficult to manage. this is why `promise` has appeared.

### 2. Promise

- Every async function returns a `promise`
- `Promise` solves the problem of `callbacks` getting called multiple times. because when a `promise` is rejected, it calls an `errback` instead of a `callback`
- `Promise` can be either `fulfilled` or `rejected`
- a `promise` can be created like this: below is a code from [mediasoup-client](https://github.com/versatica/mediasoup-client)

```typescript
async function safeEmitAsPromise(event, ...args) {
  return new Promise((resolve, reject) => {
    this.safeEmit(event, ...args, resolve, reject);
  });
}
```

- `resolve` and `reject` are also functions. However, they do not stop the function from running. They simply takes care of the `state` of a `promise`

- `Promises` can be connected with a chain to return a different `promise` after a `promise` is fulfilled.
- if you set `timeout` to a `promise`, you can prevent a `promise` from not getting fulfilled or rejected.
  - I used this when I implemented social login. if there is no answer from a social login server, it throws an error to inform that a promise has been rejected instead of waiting forever.
- `Promise.all` returns a single Promise after multiple promises get resolved. Below example is from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

```typescript
var promise1 = Promise.resolve(3);
var promise2 = 42;
var promise3 = new Promise(function (resolve, reject) {
  setTimeout(resolve, 100, "foo");
});

Promise.all([promise1, promise2, promise3]).then(function (values) {
  console.log(values);
});
```

### 3. Generator

- A `generator` allows bi-directional communication between a caller and a function. A `generator` has synchronous properties, but it can manage asynchronous codes easily if used with a `promise`
- Read a section about a generator, but still kinda confused. needs to do more studies on it. One lesson that I got is that I don't need to create a generator luncher on my own. it is better to use either `co` or `Koa`
