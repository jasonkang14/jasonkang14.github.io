---
title: Advanced CSS [02] Animating Underlines
date: "2019-06-30T20:56:37.121Z"
template: "post"
draft: false
slug: "/posts/AdvancedCSS/Animating Underlines"
category: "SASS"
tags:
  - "CSS"

description: "Practicing Advanced CSS using SASS"
---

Tried to show underline on two buttons upon mouse hover. <br>
CSS codes were written using `SASS`<br>

```css
//main.sass
$font-stack:    Helvetica, sans-serif

Button
  border: none
  height: 40px
  width: 700px
  font: 25px $font-stack
  text-decoration: none
  background-image: linear-gradient(to right, transparent 20%, currentColor 21%)
  background-position: 0% 100%
  background-repeat: no-repeat
  background-size: 0% 3px
  transition: background-size 0.3s
  position: relative

  &:hover, &:focus
    background-size: 100% 3px



.btn_wrap
  position: relative
  top: 50%
  left: 30%
  display: flex
```

A nice thing about using `SASS` is that I don't need to use curly brackets or semicolons. It kinda feels like `Python`. You can just use indentation to use different effects.

In the example above,

```
Button
    &:hover
```

is equivalent to

```
Button:hover
```

using `SASS` makes the code a lot more efficient by elminating the need to write a parent component over and over again. if you change the extension to `scss`, you can use curly brackets and semicolons to improve readability if you prefer that way.

An underline is bascially a `background-image` which is initially hidden, but appears upon `hover`.

1. `linear-gradient` was used to generate an image, which is a solid line in this case. The syntax for `linear-gradient` is like below:

```
background-image: linear-gradient(direction, color-stop1, color-stop2, ...);
```

`transparent` and `currentColor` are straight-forward. By adding `transparnt`, it makes the line appear to start slightly left of the zero position.

2. `background-position` places the underline in the bottom left corner. the first value represents the horizontal position at 0% and the second value represents the vertical position at 100%.

3. `background-repeat` prevents multiple instances of the lines filling the entire background of the button.

4. `background`size` makes the underline zero pixels wide and three pixels tall

5. `transition` on `background-size` so that any change to the property will take `0.3 seconds` to complete

6. on `hover`, the width of the underline becomes 100%, which creates a full underline, of which animation the `transition` takes care of
