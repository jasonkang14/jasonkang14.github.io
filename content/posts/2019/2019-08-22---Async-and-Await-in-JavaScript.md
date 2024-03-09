---
title: Async/Await in JavaScript
date: "2019-08-22T18:27:37.121Z"
template: "post"
draft: false
slug: "/posts/javascript/Async-and-Await-in-JavaScript"
category: "javascript"
tags:
  - "JavaScript"

description: "Async/Await in JavaScript Explained"
---

`Async/Await` is a syntax to use `promise`. I find it easier to use than `then`, and your code looks somewhat more straight-forward if you use `async/await`.

if you don't `await`, your `async` function will return a `promise` instead of the value you expected by calling a function/method. And you can only `await` inside an `async` function because it is used for asynchronous programming.

```typescript
const getTrack = async () => {
  const audioStream = await mediaDevices.getUserMedia({ audio: true });
  const audioTrack = audioStream.getAudioTracks()[0];
};
```

in the code above, I am `awaiting` for the result of `mediaDevices.getUserMedia({ audio: true })` in order to use the return value of the method. If I don't `await` for it, I get a promise instead of the value that I want. And if I were to use `console.log` to check the value assigned to `audioStream`, it will be undefined.

If you use the conventional `.then` method, the code above is equivalent to this:

```typescript
const getTrack = () => {
  mediaDevifes.getUserMedia({ audio: true }).then((audioStream) => {
    const audioTrack = audioStream.getAudioTracks()[0];
  });
};
```

If you were to only think about `.then`, `async/await` could be just merely a change of syntax, but `async/await` is a lot more useful since it allows you to use a `promise`.
