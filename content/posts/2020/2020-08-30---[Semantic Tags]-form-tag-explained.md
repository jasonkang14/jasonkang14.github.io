---
title: Semantic Tags - Form tag explained
date: "2020-08-30T10:53:37.121Z"
template: "post"
draft: false
slug: "/html/semantic-tags-form-tag-explained"
category: "HTML"
tags:
  - "HTML"

description: "Why do I have to use a form tag when div can just do everything"
---

Semantic tags allow others to understand the structure of your html without looking at detailed script codes in order to ensure that **HTML is used for structure and CSS is used for the style**

Semantic tags indicate what the tags are intended to do. If you use a **div** tag instead of a **form** as a parent of your **input** tags, it is difficult for both users and browsers to understand what the tag is meant to do. **input** tags are usually used as parts of a form for you to **submit** some data. However, if you use a **div** tag instead of a **form** tag as their parents, it is difficult to understand what the **div** tag is for until **input** tags are seen.

With the programming aspects put aside, the biggest difference that I have found is that when I use a **form** tag, the page refreshes `onSubmit`. In order for me to display a message when I receive a `status==200` messsage, I had to `event.preventDefault()` like below.

```typescript
const onSubmit = (event: React.MouseEvent<HTMLButtonElement, MouseEvent>) => {
  event.preventDefault();
  dispatch(requestEmailLogin(form));
};

useEffect =
  (() => {
    if (appState === LOGIN_FAIL) {
      setShowLoginFailModal(true);
    }
  },
  [appState]);
```

Another problem that I ran into was that when I used a **button** tag instead of an **input** tag with `type="submit"`, I got a following error:<br>
`"Form submission canceled because the form is not connected"`

And this was because when there are more than one **button** tags, the browser fails to see which **button** tag is `connected` with the **form** tag. Therefore, you have to explicitly declare the type of the **button** tag or use an **input** tag with `type="submit"` in order to let the browser know which button is connected with the form.

In my next post, I will write about the importance of using **section** tags
