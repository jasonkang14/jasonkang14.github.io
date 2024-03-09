---
title: Miniter[04] Flexbox
date: "2019-06-08T20:27:37.121Z"
template: "post"
draft: false
slug: "/posts/Minitor-Flexbox"
category: "CSS"
tags:
  - "CSS"

description: "Use of flexbox in attempt to make a mini-version of Twitter."
---

####definition by MDN : the `flex` CSS property sets how a flex item will grow or shrink to fit the space available in its flex container.

I used `flex` property in order to display two boxes horizontally in my Miniter.
![Miniter Main Page](https://scontent-hkg3-1.xx.fbcdn.net/v/t1.0-9/64529894_10219110777621233_7758357828900749312_o.jpg?_nc_cat=106&_nc_oc=AQlqJ6of7W1GWUBQ9ng_r7agwRYXVLGnDhZKibk1joLU8G6JN2IlFQspmlZq072PHpk&_nc_ht=scontent-hkg3-1.xx&oh=adce80d702b42423ec384ba8d22b7d5d&oe=5DC6A56E)

The two `div`boxes were displayed vertically by default, so I set the display of the body as `flex` like below.

```
//HTML

body {
  display: flex;
  background-color: #e6e6e6;
}
```

You have to set the display of the parent element `flex` in order to have the child elements displayed horizontally. You can also use a property called `flex-direction` in order to display your image in different ways. The properties are very straight forward.

```
.flex-container {
  display: flex;
  flex-direction: column, column-reverse, row, or row-reverse;
}
```

<br>

`flex-wrap` property specified whether you want the flex items wrapped or not. It would help you understand better if you try the examples from W3Schools in the link below.
[Flex-Wrap](https://www.w3schools.com/css/css3_flexbox.asp#flex-wrap)
<br><br>

`justify-content` property is used to align the flex items. I think this would be very helpful when you are trying to ditribute child elements evenly. It would help you understand better if you try the examples from W3Schools in the link below.
[Justify-Content](https://www.w3schools.com/css/css3_flexbox.asp#justify-content)
