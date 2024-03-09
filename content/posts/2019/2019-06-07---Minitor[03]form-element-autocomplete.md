---
title: Miniter[03] <form>, autocomplete
date: "2019-06-07T18:27:37.121Z"
template: "post"
draft: false
slug: "/posts/Minitor-form-element-autocomplete/"
category: "HTML"
tags:
  - "HTML"

description: "Use of the <form> element and autocomplete in attempt to make a mini-version of Twitter."
---

#1. `form` element
When I created `type` and `password` input tags, Chrome Developer Console threw me a message. My code still worked, but it was giving me an advice.

- [DOM] Password field is not contained in a form: (More info: https://goo.gl/9p2vKq) ;

So I can tell that password field, which was an `input` tag that I used was suppsoed to be contained in a form, which I had never heard of. So I decided to follow the link to see what's up.
This is what it says on the link.

- Make sure that each authentication process (registration, login or change password) is grouped together in a single form element.

- Don’t combine multiple processes (like registration and login), or skip the form element.

Basically, it tells you to use the `form` element, but doesn't tell you why. So I Googled further. The most plausible answer that I found is this.

- Why do we put `input` elements inside of `form` elements? For the same reason we put `li` tags inside of `ul` and `ol` elements - it's where they belong. It's semantically correct, and helps to define the markup.

By using the `form` element, we are labeling the code correctly by making them semantically correct.
[Semantically Correct Explained](https://stackoverflow.com/questions/1294493/what-does-semantically-correct-mean)
#2. `autocomplete`
While making the input fields, I ran into another another advice.

- [DOM] Input elements should have autocomplete attributes (suggested: "current-password"): (More info: https://goo.gl/9p2vKq)

This time, the link supported its argument. It helps the user not to fill in wrong information rather than to fill in right information by providing users with what inputs are expected.

- Autocomplete attributes help password managers to infer the purpose of a field in a form, saving them from accidentally saving or autofilling the wrong data.
- Further explanation is in this link [Autocomplete Explained](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#autofilling-form-controls%3A-the-autocomplete-attribute)

So with the "advice" that I had received from the Chrome Developmen Console, I wrote down my HTML like below.

```
<form>
      <div class="input-wrap">
        <input type="text" placeholder="Enter ID"
        class="enterId" autocomplete="username">
      </div>
      <div class="input-wrap">
        <input type="password" placeholder="Password"
        class="enterPassword" autocomplete="current-password">
      </div>
      <div class="input-wrap">
        <button class="login-btn"><b>Log in</b></button>
      </div>
    </form>
```

#3. event.stopPropagation()
