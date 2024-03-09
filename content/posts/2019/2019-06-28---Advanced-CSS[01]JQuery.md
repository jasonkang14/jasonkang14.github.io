---
title: Advanced CSS [01] Changing Font Color/Weight/Size upon Mousemove
date: "2019-06-28T20:56:37.121Z"
template: "post"
draft: false
slug: "/posts/Advanced-CSS-Change-Font-Mousemove"
category: "CSS"
tags:
  - "CSS"

description: "Practicing Advanced CSS using jQuery"
---

Check the demonstration on YouTube by clicking the CSS logo : <br><br>
<a href ="https://youtu.be/AYuYFC4slP8"><img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d5/CSS3_logo_and_wordmark.svg/1200px-CSS3_logo_and_wordmark.svg.png" style="width:100px;height:100px" alt="Advanced CSS"></a>

The HTML code looks like this:

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Page Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" type="text/css" media="screen" href="main.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
</head>
<body>
    <link href="https://fonts.googleapis.com/css?family=Libre+Franklin:100,300,500,700,900&display=swap" rel="stylesheet">
    <div class="text-wrap width-left">
        <span class="circle"></span>
        <br>
        <div class="text">
            Eleanor se plaignait sans cesse qu’elle n’aimait pas ses jambes et sa grand-mère lui disait toujours en soupirant : « Si jeunesse savait, si     vieillesse pouvait » .
        </div>
    </div>
    <script src="main.js"></script>
</body>
</html>
```

`<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>`
This line was added to `head` in order to use `jQuery`, and the first line in the `body` is the font that I retrieved from `Google Font`.
I used a `span` with a class in order to show the mouse location as it moves around the page.

I used jQuery to apply `mousemove` events to the text like below;

```
$('.text-wrap').mousemove(e => {
    const wrapWidth = $('.text-wrap').width();
    const step = wrapWidth/5;
    const nowX = e.pageX - $('.text-wrap').offset().left;
    $('.circle').css({'left': nowX, 'top': e.pageY});

    if (nowX < step) {
        $('.text-wrap').attr('class', 'text-wrap step-1');
    } else if (nowX >= step && nowX < step*2) {
        $('.text-wrap').attr('class', 'text-wrap step-2');
    } else if (nowX >= step*2 && nowX < step*3) {
        $('.text-wrap').attr('class', 'text-wrap step-3');
    } else if (nowX >= step*3 && nowX < step*4) {
        $('.text-wrap').attr('class', 'text-wrap step-4');
    } else if (nowX >= step*4 && nowX < step*5) {
        $('.text-wrap').attr('class', 'text-wrap step-5');
    }
})
```

I separated the entire page into five sections becuase I wanted to use five different font sizes. Since the element has some padding by default, `offset().left` was subtracted from the mouse position.

The circle, which is supposed to move along witht he mouse, has its position assigned to it. The horizontal position is assigned by accounting the `offset().left` of the page.

The CSS code looks like below;

```
  .step-1 {
    font-weight: 100;
    color: red;
  }

  .step-2 {
    font-weight: 300;
    color: orange;
  }

  .step-3 {
    font-weight: 500;
    color: green;
  }

  .step-4 {
    font-weight: 700;
    color: blue;
  }

  .step-5 {
    font-weight: 900;
    color: purple;
  }

  * {
      cursor: none;
  }
```

`cursor: none;` was applied to show the mousemovent with the circle only.
