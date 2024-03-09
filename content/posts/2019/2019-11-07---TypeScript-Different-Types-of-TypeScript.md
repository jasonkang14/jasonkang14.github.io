---
title: TypeScript - Different Types of TypeScript
date: "2019-11-07T21:27:37.121Z"
template: "post"
draft: false
slug: "/posts/typescript/different-types-of-typescript"
category: "Typescript"
tags:
  - "TypeScript"

description: "Types that you can declare while using TypeScript"
---

1. Array

```typescript
const values: number[] = [1, 2, 3];
const values2: Array<number> = [1, 2, 3];
values.push("a"); // TypeError
```

You cannot push a string into an array declared as an array of numbers.

```typescript
const data: [string, number] = [msg, size];
data[0].substr(1);
data[1].substr(1); // TypeError
```

You cannot use a string method on a number.

2. null vs undefined

```typescript
let v1: undefined = undefined;
let v2: null = null;
v1 = 123; // TypeError

let v3: number | undefined = undefined;
v3 = 123;
```

You cannot assign a number to a variable of undefined type.
`undefined` and `null` can be used with other types to declare a variable as a union type, which is used with `|`.

3. Literal type

```typescript
let v1: 10 | 20 | 30;
v1 = 10;
v1 = 15; // TypeError

let v2: "police" | "firefighter";
let v2 = "doctor"; // TypeError
```

When you declare a variable with a certain value, the value becomes its literal type. `v1` can be 10, 20, or 30, and `v2` can be only either 'police' or 'firefighter'

4. any type
   any can be anything. it could be either a number or a string. Any type also could be a function. If you are trying to use TypeScript to change codes which have been written in JavaScript, it is helpful to use `any` type. But if you use it too much, it defeats the purpose of using TypeScript.

5. void vs never
   If a function does not return anything, it could be declared `void`. And if a function stops due to an exception or does not stop due to an infitie loop, it can be declared `never`

```typescript
function f1(): void {
    console.log('hello')    // this function does not return anything
}

function f2(): never {
    throw new Error('some error');   // this function stops due to an exception/error
}

function f3(): never {
    while (true) {
        ...          // this function has an infinite loop
    }
}
```

6. Object

```typescript
let v: obejct;
v = {
    name: 'abc';
};

console.log(v.prop1);   // TypeError
```

Since there is no information about the object property, it throws a TypeError. If you want to declare a type with information about properties included, you have to use interface, which will be explained later.

7. Intersection Type and Union Type
   Intersection Type is declared with `&` and Union Type is declared with `|`

```typescript
let v1: (1 | 3 | 5) & (3 | 5 | 7);
v1 = 3;
v1 = 1; // TypeError
```

`v1` can be either 3 or 5 and nothing else.

8. Giving a nickname to a type

```typescript
type Width = number | string;
let width: Width;
width = 100;
width = "100px";
```

You are assigning `number | string` to a type variable `Width`. As a variable `width` is declared with the type of `Width`, the variable can be either number or string.

9. enum type

```typescript
enum Fruit {
  Apple,
  Banana,
  Orange,
}

const v1: Fruit = Fruit.Apple;
const v2: Fruit.Apple | Fruit.Banana = Fruit.Banana;
```

You declare Fruit by using `enum` type. `v1` has the type of `Fruit` and has been assigned with the value `Fruit.Apple`, and `v2` has the type of `Fruit.Apple`. I think

```typescript
enum Fruit {
  Apple,
  Banana = 5,
  Orange,
}
console.log(Fruit.Apple, Fruit.Banana, Fruit.Orange); // 0, 5, 6
```

If you do not assign anything to an element of `enum` type, `0/zero` is automatically assigned to it. And if you assign a number to an element, the next element gets a number which is greater than the number assigned to the previous element--unless you declare it otherwise.

When you compile an `enum` type variable;

```typescript
var Fruit;
(function (Fruit) {
  Fruit[(Fruit["Apple"] = 0)] = "Apple";
  Fruit[(Fruit["Banana"] = 5)] = "Banana";
  Fruit[(Fruit["Orange"] = 6)] = "Orange";
})(Fruit || (Fruit = {}));
console.log(Fruit.Apple, Fruit.Banana, Fruit.Orange); // 0, 5, 6
```

the `enum` type exists as an object, and each element is mapped bi-directionally with the key and value.

If you use it for run-time;

```typescript
enum Fruit {
  Apple,
  Banana = 5,
  Orange,
}

console.log(Fruit.Banana); // 5
console.log(Fruit["Banana"]); // 5
console.log(Fruit[5]); // Banana     << bi-directional mapping
```

But if you assign a string to an element of an `enum` type, it is uni-directionally mapped since the same string could be assigned to different elements.

10. Function type

In order to declare a function, you need types of parameters and returns. You can declare types of parameters and returns using a colon(:)

```typescript
function getInfoText(name: string, age: number): string {
  const nameText = name.substr(0, 10);
  const ageText = age >= 35 ? "senior" : "junior";
  return `name: ${nameText}, age: ${ageText}`;
}

const v1: string = getInfoText("mike", 23);
const v2: string = getInfoText("mile", "23"); // TypeError
const v3: number = getInfoText("mike", 23); // TypeError
```

the types of parameters are declared inside parenthesis, and the type of return is declared right before the curly brackets;

You can declare the above function like this as well;

```typescript
const getInfoText: (name: string, age: number) => string = function (name, age) {
    ...
}
```

You can declare an optional parameter using a question mark like below

```typescript
function getInfoText(name: string, age: number, language?: string): string {
  const nameText = name.substr(0, 10);
  const ageText = age >= 35 ? "senior" : "junior";
  const languageTet = language ? language.substr(0, 10) : "";
  return `name: ${nameText}, age: ${ageText}, lanauge: ${languageText}`;
}
getInfoText("mike", 23, "ko");
getInfoText("mile", "23"); //
getInfoText("mike", 23, 123); // TypeError
```

The second case does not throw a TypeError since `language` is an optional variable. However, the third case throws a TypeError since the type of `language` has to be a string if it is used.

You can also assign `undefined` buy using `union` type.

```typescript
function getInfoText(
    name: string,
    language: string | undefined,
    age: number,
): string {
    ...
}

getInfoText('mike', undefined, 23);
```

This does not throw a TypeError, but its usability and readability is extremely low. You can pre-assign a value to a parameter like you would do in Python. I won't write about that here

You can assign `this` type of a function as the first parameter of a function like below

```typescript
function getParam(this: string, index: number): string {
  const params = this.splt(","); // TypeError
}
```

The code above throws a TypeError since the type of this has been declared as string. If you do not declare the type of `this`, it would not have thrown a TypeError. And `index` here is the first parameter since `this` type is not a parameter.

You use `interface` when you add a method to a primitive type like below;

```typescript
interface String {
  getParam(this: string, index: number): string;
}

String.prototype.getParam = getParam;
console.log("asdf, 1234, ok ".gerParam(1));
```

You are adding `getParam` method to a primitive type `String` by using `interface`

11. Function Overload: declaring multiple types at once

Since JavaScript is a Dynamically Typed Language, one functioncan have different parameter types and return types. In TypeScript, you can use function overload to declare multiple types within a single function.

```typescript
function add(x: number | string, y: number | string): number | string {
  if (typeof x === "number" && typeof y === "number") {
    return x + y;
  } else {
    const result = Number(x) + Number(y);
    return result.toString();
  }
}

const v1: number = add(1, 2); // TypeError
console.log(add(1, "2"));
```

`const v1` throws a TypeError even though both parameters and return are numbers. This is because the type of the function was not declared specifically. This is how you are supposed to declare a function using `function overload`

```typescript
functino add(x: number, y: number): number;
function add(x: string, y: string): string;
function add(x: number | string, y: number | string): number | string {
    ...
}
```

Basically you are declaring a function with all the possible options
