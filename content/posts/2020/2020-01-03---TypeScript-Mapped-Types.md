---
title: TypeScript - Mapped Types
date: "2020-01-03T13:27:37.121Z"
template: "post"
draft: false
slug: "/posts/typescript/mapped-types"
category: "TypeScript"
tags:
  - "TypeScript"

description: "TypeScript, mapped types explained"
---

Mapped types allow you to map each property in a type in the same way. Think about the `map` function which you can use with a JavaScript array. By using the `map` function, you are doing the same task to each properties--like multiplying all of them by the factor of 3.

By `mapping` your type, you can make all propeties of type `readonly` or `optional`.

In old format, you would do something like below if you want to make each property optional or readonly;

```typescript
// optional
interface CartItem {
  totalPrice?: number;
  rewardPoints?: number;
  id?: number;
  count?: number;
}

// readonly
interface CartItem {
  readonly totalPrice: number;
  readonly rewardPoints: number;
  readonly id: number;
  readonly count: number;
}
```

With **mapped types**, you can do it like below. `T` represents Type/Interface and `P` properties

```typescript
type ReadOnly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

And use the mapped type like below;

```typescript
type CartItemPartial = Partial<CartItem>;
type ReadonlyCartItem = Readonly<CartItem>;
```

You can add members using an intersection type and a mapped type together

```typescript
type PartialWithNewMember<T> = {
    [P in keyof T]> : T[P];
} & { newMember: boolean}

type Keys = "option1" | "option2";
type Flags = { [K in Keys]: boolean };

```
