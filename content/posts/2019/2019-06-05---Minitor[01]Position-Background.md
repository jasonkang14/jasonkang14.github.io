---
title: Miniter[01] Position, Background-
date: "2019-06-05T14:27:37.121Z"
template: "post"
draft: false
slug: "/posts/Minitor-Position-Background/"
category: "CSS"
tags:
  - "CSS"

description: "Use of position, and background- properties on HTML and CSS in attempt to make a mini-version of Twitter."
---

I joined a coding bootcamp called WeCode which is based in Seoul, South Korea.
For my first project, I am making a mini-version of Twitter with sign-up, log-in, and making tweets. The end product would look something like this.

![Final Picture of Miniter](https://scontent-hkg3-1.xx.fbcdn.net/v/t1.0-9/62421048_10219088287138985_1856569703267303424_o.jpg?_nc_cat=105&_nc_ht=scontent-hkg3-1.xx&oh=e4ac2dfda5d5da06e93602ed60cb5cb1&oe=5D7C3881)

I got to use `position` and `background-` properties to make my page.
`position` is a common property that a front-end developer uses when he or she tries to set a layout for the webpage. I had a hard time trying to understand the difference between `absolute` and `relative`. Just like any other front developers would do, I changed the property to see how they work, but it was difficult to see it right away. Below is a picture that really helped me understand the difference between `absolute` and `relative`

![`absolute` and `relative](https://scontent-hkg3-1.xx.fbcdn.net/v/t1.0-9/64437787_10219088554305664_1048970393961889792_n.jpg?_nc_cat=102&_nc_ht=scontent-hkg3-1.xx&oh=439ff1632529d25feb4a0ff639047088&oe=5D9AC986)
[source](https://medium.com/@leannezhang/difference-between-css-position-absolute-versus-relative-35f064384c6)

`absolute` : Positioning an element based on its closest positioned ancestor position. So basically you are placing your element based on its parent element and changing the layout around it.

`relative` : Positioning an element based on its current position without changing layout.

`background-` properties: I was trying to insert an image as a `background-image` instead of using an `img` tag/element.

![Background-Image Used](https://scontent-hkg3-1.xx.fbcdn.net/v/t1.0-9/62650384_10219088606706974_3332370773124841472_n.jpg?_nc_cat=106&_nc_ht=scontent-hkg3-1.xx&oh=b81d227a6e6a486754b39bf3a0163ad8&oe=5D8F2B9A)

Below is the CSS I wrote for this.

```
.blueCircle {
  background: #0099ff;
  position: absolute;
  left: 12%;
  top: 35%;
  border: 5px solid white;
  height: 100px;
  border-radius: 50%;
  width: 100px;
  background-image: url(https://scontent-hkg3-1.xx.fbcdn.net/v/t1.0-9/62072328_10219032446142995_4385081662694752256_n.jpg?_nc_cat=103&_nc_ht=scontent-hkg3-1.xx&oh=27926ae25a7eb38a36b18114643a4edc&oe=5D98939C);
  background-repeat: no-repeat;
  background-size: 100%;
  background-position: center;
}
```

`position: absolute;` was used in order to place the image based on the blue box in the image above.<br>

`background-img: url(link);` was used in order to insert the image. <br>

`background-repeat: no-repeat;` background image is repeated vertically and horizontally by default. you can set its property as `repeat-x` to have your image repeated only horizonatlly or `repeat-y` to have it vertically. `no-repeat` prevents image repetition.<br>

`backround-size: 100%;` makes the image fill the given space set by `height` and `width`. <br>`background-size: cover;` expands the image to fill the entire space. If the given space is too small compared to the size of the image, it might cut out most of the image. <br>`background-size: contain;` manipulates the size of the image to show the entire image within the given space.<br>

`background-position: center;` puts the image at the center of a given space.
