---
title: Babel - How to Make a Babel Plugin
date: "2019-12-05T17:27:37.121Z"
template: "post"
draft: false
slug: "/posts/babel/how-to-make-a-babel-plugin"
category: "Babel"
tags:
  - "Babel"

description: "How to make a babel plugin"
---

Babel provides APIs so that anyone can create a preset and plugin of their own.

Babel plugin changes a code based on AST(Abstract Syntax Tree). Therefore, you have to know the structure of AST in order to create your own plugin

you can found the full code on [github](https://github.com/jasonkang14/babel_practice/tree/master/test-babel-custom-plugin)

1. Basic structure of a babel plugin

   - exports a function with a `types` parameter
   - uses the `types` parameter to make a node
   - the `types` node is used to check the tyoe of the AST node
   - nothing happens if you return an empty object.

2. A babel plugin to remove console.log's

   - this is the structure of the AST of `console.log('asdf')`;
   - console.log code starts with an `ExpressionStatement` node
   - a code that calls a function or a method is a `CallExpression` node
   - a `MemberExpression` node inside the `CallExpression` node calls a method
   - the `MemberExpression` node has the information about the object and the method

3. A babel plugin to add console.log's
   - this is the structure of the AST of a simple function
   - a code that declares the function is a `FunctionDeclaration` node
   - the name of the function is the value of `id`
   - all the nodes of the code of the function is inside the body of a `BlockStatement` node
   - We are trying to add a `console.log` at the very beginning of the array
