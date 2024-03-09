---
title: Babel with React?
date: "2019-12-04T17:53:37.121Z"
template: "post"
draft: false
slug: "/posts/babel/what-with-react"
category: "Babel"
tags:
  - "React"

description: "How Babel works with React"
---

`React` uses `JSX`, which is a **XML-like syntax extension for JavaScript, which allows
HTML elements to be written in JavaSCript and to be placed in the DOM.**

If you are used to Vanilla JavaScript, you would write code like below

```
const test = document.createElement("h1");
test.innerText = "Hello, World!";
```

JSX allows you to change the above code like this

```
const test = <h1>Hello, World!</h1>
```

Babel allow such transformation by transpiling JSX into standard JavaScript code. The code above that you used in order to declare `test` is actually like below

```
const test = React.createElement("h1", null, "Hello, World!");
```

`React` incorporates `Babel` using `Webpack`. `Webpack` is **a package bundler for JavaScript which compiles modules into a single source, which is then rendered in the browser**

When you write codes with React, you `export` child components, which then get `imported` in parent components. This is also an ES6 grammar, which needs to be translated using Babel in order to be used in web browsers.

When you write components in React, you create separate files for each component. By the hierarchy declared in Webpack, Webpack creates dependency graph, compiles all the files to a single code by removing all the `import` and `export` statements using Babel, and converts JSX into standard JavaScript.
