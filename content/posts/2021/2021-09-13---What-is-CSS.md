---
title: What is CSS?
date: "2021-09-13T23:34:37.121Z"
template: "post"
draft: false
slug: "/css/what-is-css"
category: "CSS"
tags:
  - "CSS"

description: "The role of CSS in web development"
---

In the previous video, I talked about what HTML is. This video is about CSS. **CSS stands for Cascading Style Sheet.** Just like what I did with HTML, I will break down each term to help you understand better.

This time I am going backward. A sheet is a piece of paper. So a style sheet is a piece of paper that has some styles on it.

Let's think about the word style. what is it? According to the Webster Dictionary, **the word style means to shape or design something so that it looks attractive.**

It's the same with a webpage. A style sheet is a way to make a webpage look more attractive. Different colors, fonts, or even where to put certain images are defined in a style sheet.

Then what does cascade mean? According to MDN, **which I discussed in the previous video about HTML,** **the cascade is an algorithm that defines how to combine property values originating from different sources**. What do you mean by different sources?

Let's say you have HTML elements that look like this

```html
<div>Code On div</div>
<div id="div-with-id">Code On div with id</div>
<div class="div-with-class">Code On div with class</div>
<div class="div-with-class" id="div-with-id">Code On div with id and class</div>
```

If you look at the elements above, you can tell that they are all div elements. One with an id called `div-with-id` and another with a class called `div-with-class`. And the last one has both. The id and class are usually used to apply styles.

Now you have declared three different styles. The first one says that the color of all the div elements must be red.

```css
div {
  color: red;
}
```

The second one says that the color of all the elements with the id of `div-with-id` must be orange.

```css
#div-with-id {
  color: orange;
}
```

And the third one says the color of all the elements with the class of `div-with-class` must be blue.

```css
.div-with-class {
  color: blue;
}
```

The first three elements are pretty obvious, but what about the last one that has both the id and class?

if you apply the styles it looks like below;

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d4be966-53c3-4a47-9ce1-761227810557/_2021-06-21__3.45.46.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d4be966-53c3-4a47-9ce1-761227810557/_2021-06-21__3.45.46.png)

Now you have three different styles that would be applied to the HTML element, **but an element cannot have multiple colors. It can only have one.** This is where the cascading comes in. There is this cascading mechanism that allows a browser to determine which color to choose.

Simply put, the Cascading Style Sheet is a stylesheet that cascades. which means that **it defines how to choose styles for each HTML element to make it look more attractive.** The more attractive a webpage is, the more likely it would you hook you up to stay on the webpage.

If you need some visuals to understand this concept, please check out the YouTube video below.

<iframe width="900" height="315" src="https://www.youtube.com/embed/wQ6swEYqsZk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
