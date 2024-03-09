---
title: React[01] Remaking Miniter Using React I - JSX
date: "2019-06-22T22:56:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Remaking-Miniter-Using-React-part-one"
category: "React"
tags:
  - "React"

description: "Remaking Miniter Using React"
---

Decided to dig deeper into the front-end development. Used `React` to re-do the miniter project which I had done using DOM
#1. JSX format
Simply put, it's using HTML elements as a string like this.

```
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

Any JavaScript expression can go inside the curly braces in JSX. For the longest time, I was wondering why every single example was rendinring the element to document.getElementById('root') and found out that HTML components written in React becomes children to that element. like below:

```
<!doctype html>
<html lang="en>
  <head>...</head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root>...</div>
      <!--
        ... has code written in React
      -->
  <script>...</script>
 </body>
</html>
```

You can just close the element directly like this. without a separate closing tag

```
<button name="input-btn" />
```

#2. JSX as an object
if you write something like this,

```
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

this is same as this

```
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
