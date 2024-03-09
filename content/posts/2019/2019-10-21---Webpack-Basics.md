---
title: Webpack - Basics
date: "2019-10-21T21:27:37.121Z"
template: "post"
draft: false
slug: "/posts/webpack/basics"
category: "Webpack"
tags:
  - "Webpack"

description: "Webpack basics"
---

Webpack is a module bundler-- a tool that takes pieces of JavaScript and their dependencies and bundles them into a single file.

Module represents each resource file, and a bundle represents the final product after running webpack. One bundle is made of many modules.

Webpack is required since a single-page-application uses a single HTML file which consists of a lot of JavaScript files. Those modules are bundled into a bundle using webpack.

you can found the full code on [github](https://github.com/jasonkang14/babel_practice/tree/master/test-babel-custom-plugin)

1. Running Webpack
   When you run Webpack, a `dist` directory is created where a `main.js` bundle file is created. Below is the basic structure of the bundle

   - the entire bundle file is an IIFE(Immediately Invoked Function Expression)
   - runtime code is what takes care of modules. If you use multiple `entry` files in `webpack.config.js`, a bundle file created by each `entry` file has a webpack runtime code.
   - code that I wrote is used as a parameter of the IIFE

2. Using a Loader
   A loader is a function that takes a module as an imput and transforms it into a format that you desire. Image files, CSS files, CSV files, or any other files can be modules.

   - there are different types of loaders like `babel-loader`, `file-loader`, `style-loader` and so on. You have to choose the right type of loaders to deal with the file that you are trying to use.
   - if you add an image file to a bundle, you can reduce the number of times that a browser requests for the file by using `url-loader`
     - `url-loader` provides a fallback option which allows other loaders to take care of the file if the file is smaller than its limit

3. Using Plugins
   - `html-webpack-plugin` is used to update the HTML file automatically when contents are updated.
   - `clean-webpack-plugin` cleans the `dist` directory each time webpack is ran.
   - `DefinePlugin` is used to replace strings in modules.
   - `ProvidePlugin` automatically adds previously configured modules
