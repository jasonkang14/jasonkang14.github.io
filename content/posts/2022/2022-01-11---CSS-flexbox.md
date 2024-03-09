---
title: CSS - flexbox
date: "2022-01-11T11:34:37.121Z"
template: "post"
draft: false
slug: "/css/flexbox"
category: "CSS"
tags:
  - "CSS"

description: "How the flexbox works in CSS"
---

In `flexbox`, a parent element is called a flex container, and the children are called flex items. There are a lot of properties that you can use with flexbox. But some properties are used more than others. so let's talk about the ones that are used more frequently.

The word `flex` is used here because it allows you to align and distribute space among items in a container even when their size is unknown. The main idea of the `flexbox` is to let the container change its item sizes to fill the available space.

let's think about HTML tags like this

```html
<div class="parent">
  <div class="child"></div>
  <div class="child"></div>
  <div class="child"></div>
  <div class="child"></div>
</div>
```

First, in order for you to use flexbox, you gotta declare the parent as a flex. And now let's look at some properties that you can set to your flex container

```css
.parent {
  display: flex;
}
```

1. `flex-direction` : just like the name says, it determines the direction in which your flex items are aligned.

   - There are four `flex-direction` options. `row`, `row-reverse`, `column`, and `column-reverse`. It's pretty straightforward.
   - `row` is the default. it aligns your items left to right. and `row-reverse` is row, but reversed. so it aligns your items right to left. `column` aligns your items top to bottom like a column, and `column-reverse` is column but reversed. so it aligns them bottom to top.

2. `flex-wrap`. By default, flex items will all try to fit into a single line. but you can change that and allow your items to wrap around the container.

   - there are three `flex-wrap` options. `nowrap`, `wrap`, and `wrap-reverse`.
   - let me add more flex items to help you understand this better. You can see how the width shrinks when I add more flex items.
   - If you remember, I told you in the very beginning that the main idea of the flexbox is to let the container change its item sizes. and by default, they all want to fit in one line.
   - `nowrap` is the default. it does not wrap. `wrap` allows your children elements to wrap onto multiple lines from top to bottom. `wrap-reverse` is wrap but reversed. it wraps from bottom to top.

3. `justify-content`. This defines the alignment along the `flex-direction`. there are a lot of options that you can use for this property. but the ones that are commonly used are `flex-start`, `flex-end`, `center`, and `space-between`
   - `flex-start` is the default. the items are leaning towards the start of the `flex-direction`
   - `flex-end`is the opposite of `flex-start`. the items are leaning towards the end
   - `center` aligns items at the center
   - `space-between` the first item is at the start, the last item is at the end. and the rest are distributed evenly.
4. `align-items`. this defines the alignment along the other direction. if you set your `flex-direction` to `row`, `align-items` defines alignments along the vertical line. the ones that are commonly used are `flex-start`, `flex-end`, and `center`
   - `flex-start` aligns items at the start of the line
   - `flex-end` aligns items at the end of the line
   - `center` aligns items at the center of the line, I personally use this option a lot

And there are also properties that you can set on your flex items. A lot of times, you don't have to set properties on your flex items to use flexbox. but sometimes, you gotta use these properties as well. Again, I am going to talk about the ones that are used more often.

1. `flex-grow`
   - if you remember, I talked about in the beginning how flexbox changes its sizes to fill the container. if you set an item to `flex-grow: 1` this specific item will fill the remaining spaces that are left.
2. `flex-shrink`
   - this is the opposite of `flex-grow`. `flex-shrink` declares which item to shrink first if the container is too small

If you need some visuals to understand this better, please check out the YouTube video below.

<iframe width="900" height="315" src="https://www.youtube.com/embed/JBcHff_JcXM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
