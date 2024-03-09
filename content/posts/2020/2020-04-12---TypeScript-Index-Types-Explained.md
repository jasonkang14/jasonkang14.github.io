---
title: TypeScript - Index Types
date: "2020-04-12T19:53:37.121Z"
template: "post"
draft: false
slug: "/typescript/index-types-explained"
category: "Typescript"
tags:
  - "TypeScript"

description: "Explaning Index Types in TypeScript"
---

It is easier to understand by looking at the code directly.

```typescript
function pluck<T, K extends keyof T>(o: T, propertyNames: K[]): T[K][] {
  return propertyNames.map((n) => o[n]);
}

interface Car {
  manufacturer: string;
  model: string;
  year: number;
}

let taxi: Car = {
  manufacturer: "Toyota",
  model: "Camry",
  year: 2014,
};

let makeAndModel: string[] = pluck(taxi, ["manufacturer", "model"]);

let modelYear = pluck(taxi, ["model", "year"]);
```

An object called `taxi` has an interface called `Car`. And there is a function `pluck`, which checks if the object has items in an array as its property.

If you look at `(o: T, propertyNames: K[])` this section right here, the first argument is an object, which is `taxi` in this case, and the second argument is an array of strings, which are properties of the object.

If you add another property like `owner: string` to the Car interface, the type `keyof Car`, or `keyof T` in this case, is automatically updated as well.

The second operator is `T[K]`, which is the **indexed access operator**. Here `T` represents the object, and `K` represents `keyof T`, so the `pluck` function can be re-written like below;

```typescript
function pluck<T, K extends keyof T>(o: T, propertyName: K): T[K] {
  return o[propertyName]; // o[propertyName] is of type T[K]
}
```
