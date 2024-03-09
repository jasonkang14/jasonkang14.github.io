---
title: Babel - How to Make a Babel Plugin
date: "2019-12-07T23:27:37.121Z"
template: "post"
draft: false
slug: "/posts/babel/babel-with-typescript"
category: "Babel"
tags:
  - "TypeScript"

description: "Using Babel with TypeScript"
---

Babel 7 supports TypeScript. Since I am pretty new to web development, I did not fully understand why people were struggling to use Babel with TypeScript. According to what I have found online, Babel did not support type check, which is a key feature of TypeScript. And TypeScript itself could do both compiling and type checking. But now Babel 7 allows TypeScript to compile faster by allowsing the TypeScript compiler to only check types wihtout emitting files

Install all the necessary plugins:
`npm install --save-dev @babel/preset-typescript @babel/preset-env @babel/plugin-proposal-class-properties @babel/plugin-proposal-object-rest-spread`

Set up `.babelrc` like below

```
{
    "presets": [
        "@babel/env",
        "@babel/preset-typescript"
    ],
    "plugins": [
        "@babel/proposal-class-properties",
        "@babel/proposal-object-rest-spread"
    ]
}
```

Create a `tsconfig.json`

```
{
  "compilerOptions": {
    // Target latest version of ECMAScript.
    "target": "esnext",
    // Search under node_modules for non-relative imports.
    "moduleResolution": "node",
    // Process & infer types from .js files.
    "allowJs": true,
    // Don't emit; allow Babel to transform files.
    "noEmit": true,
    // Enable strictest settings like strictNullChecks & noImplicitAny.
    "strict": true,
    // Disallow features that require cross-file information for emit.
    "isolatedModules": true,
    // Import non-ES modules as default imports.
    "esModuleInterop": true
  },
  "include": [
    "src"
  ]
}
```
