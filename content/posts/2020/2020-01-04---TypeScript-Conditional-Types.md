---
title: TypeScript - Conditional Types
date: "2020-01-04T13:27:37.121Z"
template: "post"
draft: false
slug: "/posts/typescript/conditional-types"
category: "TypeScript"
tags:
  - "TypeScript"

description: "TypeScript, conditional types explained"
---

**This is a summary of the [official document](https://typesriptlang.org)**

Conditional types are very self-explanatory. Depending on its condition, the type can be either one or the other.

`T extends U ? X: Y`

A conditional type T is either X, or, Y, or deferred depending on one or more type variables. The type of T depends on whether the type system has enough information to conclude that T is always assignable to U.

Look at the example below;

```typescript
declare function f<T extends boolean>(x: T): T extends true ? string : number;

let x = f(Math.random() < 0.5);
```

If I were to examine the code above, type T could be either `string`, `number`, or deferred. And it depends on whether the type system has enough information to conclude that T is always assignable to `true` in this case.

if `Math.random()` is a number less than 0.5, the type T is `number`.
