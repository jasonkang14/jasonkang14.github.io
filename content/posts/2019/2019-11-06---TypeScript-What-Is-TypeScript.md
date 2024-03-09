---
title: TypeScript - What Is TypeScript?
date: "2019-11-06T21:27:37.121Z"
template: "post"
draft: false
slug: "/posts/typescript/what-is-typescript"
category: "Typescript"
tags:
  - "TypeScript"

description: "Introducing TypeScript"
---

TypeScript is a Statically Typed Language which includes all the aspects/properties/functions of JavaScript. TypeScript is developed by Microsoft, and you can use it with `React.js` and `Redux`.

Statically Typed Language has a steep learning curve since you have to consider the type of a variable when you declare it. However, it is more efficient if you are working on a pretty big project.

Statically Typed Lanauge is more efficient, or more productive, because all the codes are linked with one another by type. Therefore, it is easy to move around related codes, which also makes refacrtoring easier.

Statically Typed Langauge allows you to face erros during compile-time, which means that you can find your error as your codes become executable, before they start running. So you can find the error as you type your code.

Check out the code below;

```typescript
let v1 = 123;
v1 = "abc"; // throws an error
```

The code above throws an error since 'abc' is not a number. This is the advantage of TypeScript as you find your error during compile-time.

You can declare a variable with different type options like below as well;

```typescript
let v1: number | string = 123;
v1 = "abc";
```

In the code above, `v1` is either a number or a string, so you can assign 'abc' to `v1`, and it will not throw an error
