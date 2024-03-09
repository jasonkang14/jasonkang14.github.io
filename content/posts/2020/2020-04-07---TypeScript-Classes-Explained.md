---
title: TypeScript - Classes
date: "2020-04-07T19:53:37.121Z"
template: "post"
draft: false
slug: "/typescript/classes-explained"
category: "Typescript"
tags:
  - "TypeScript"

description: "Explaning classes in TypeScript"
---

Classes are a programming property used in Objected Oriented Programming. In TypeScript, you can use a `Class` with `Interface` like below;

```typescript
class Student {
  fullName: string;
  constructor(
    public firstName: string,
    public middleInitial: string,
    public lastName: string
  ) {
    this.fullName = firstName + " " + middleInitial + " " + lastName;
  }
}

interface Person {
  firstName: string;
  lastName: string;
}

function greeter(person: Person) {
  return "Hello, " + person.firstName + " " + person.lastName;
}

let user = new Student("Jane", "M.", "User");

document.body.textContent = greeter(user);
```

First you declare a class called `Student`. In its constructor, it takes arguments like `firstName`, `middleInitial`, and `lastName`. The use of `public` on arguments indicate that you can create properties with that name.

The `Person` interface, which is the type of an argument for the function `greeter`, declares the types of `firstName` and `lastName`. Thus ensuring the created `user` object has the types that can be used for the `greeter` function.
