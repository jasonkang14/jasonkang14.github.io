---
title: Django - Views, Class vs Function
date: "2020-05-11T23:53:37.121Z"
template: "post"
draft: false
slug: "/django/django-views-function-vs-class-comparison"
category: "Django"
tags:
  - "Django"

description: "Explaining the difference between function-based view and class-based view"
---

## Function-Based View

Django originally had only funcion-based view, which takes a `request` object as an argument and returns a `response` object. I will write about those objects in a later post.

The returned `response` object is usually a `render` object, which is a built-in django shortcut which can return a django template with some messages. The advantage of using a function-based view is that you know exactly what you are getting since you have to write out every single case that you are dealing with. However, it is also the downside of a function-based view is that you have to write code over and over again in order to handle every single exception. For a single login view, you might have to write out 30 lines to handle a wrong username, a wrong password, and a wrong username and wrong password, and if you have more than two fields for your login for some reason, the view would have to be a lot longer to handle every possible case.

## Class-Based View

Class-based views were added in order to make views more concise. Since they are more concise, you now have to have a better understanding of underlying inheritance structure as the class inherits `django.views.View`

Class-based views are Python classes, which has to classmethods of `get` and `post`, or `patch` and `delete` can be also used depending on the type of API you are trying to create. And it is also easier to reuse your code when you are using class-based views. And this is the reason why class-based views were created in the first place. **Class-based views were created in order to prevent developers from repeating their codes over and over again**.

This image shows what type of views to use;
![which view to choose](https://i.imgur.com/Z9iTIBA.jpg)

### Pros and Cons

Function-based views might have longer codes, but they are more straight-foward compared to class-based views. Class-based views might be more difficult to read since the codes would not be as explicit, however, you can reuse your code over and over again especially if you are using built-in generic class based views.

Generic Class-Based Views seem to be the best option if you can fully understand the structure of the code. I will write about it in a later post.
