---
title: Object Oriented Programming
date: "2022-04-25T13:34:37.121Z"
template: "post"
draft: false
slug: "/cs/object-oriented-programming"
category: "CS"
tags:
  - "CS"

description: "What is OOP?"
---

# TL;DR

### Object Oriented Programming is creating your own data types for your program.

If you are a software engineer or a software developer, you are very likely to have heard about Object-Oriented Programming. It sounds kinda fancy, but it’s not really that difficult. Let’s break this word down. if you look into the terminology, it really helps you understand what it means.

Just like the name says, Object-Oriented Programming means that your program is organized with objects. In order to understand what an `object` is, you should understand what a `class` is.

A class is a way to define data. Let’s think about a car. We are trying to define what a car is. A car has a lot of properties. When you think about a car, you can think of its manufacturer, its model, its color, or types like a sedan or an SUV. These properties are called `class attributes`.

A car is also capable of doing something. It can start, stop, or drive. And these are called `class methods`. And if you use JavaScript, you can declare a class called car like this

```javascript
class Car {
  constructor(manufacturer, model, color, type) {
    this.manufacturer = manufacturer;
    this.model = model;
    this.color = color;
    this.type = type;
  }

  start() {}

  stop() {}

  drive() {}
}
```

I have just defined a data called `Car`. Then what is an `object`? An object is an instance of a class created with specific data. In English, an object is a real example of a class. Let’s think about a tesla model X. It’s a car. And if you were to define it using the class that we have declared, it would look like this

```javascript
const modelX = new Car("tesla", "X", "white", "SUV");
```

In the example that I just gave, the `modelX` is an object. Object-Oriented Programming is a way to program using classes and objects like we just did by defining own data types.

I called a class a data type because data types like Number are also classes. When you think about the number 3, it just looks like a number. But in programming, **Number is a class, and the number 3 is an object.**

Just like the modelX object we declared, the number 3 also has attributes and methods. For example, you can change the number 3 into a string by calling the `toString()` method like this. `String` is a data type that represents a sequence of characters.

```javascript
const three = Number(3);
const threeStr = three.toString();
console.log(threeStr); // '3'
console.log(typeof threeStr); // 'string'
```
