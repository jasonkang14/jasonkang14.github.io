---
title: What Is Axios and Why Do You Use it?
date: "2019-07-12T14:27:37.121Z"
template: "post"
draft: false
slug: "/posts/Axios-explained"
category: "Axios"
tags:
  - "JavaScript"

description: "Explaining Axios"
---

`Axios` is a library you can use to perform HTTP requests. Basically, you use this in order to send a request to a GraphQL or REST API. `Axios` is similar to the native `Fetch API`, but `Axios` has some advantages.

1. Automatic Transformation
   When you use the native `Fetch API`, you have to transfrom the `response` to `json` in order to use it. This means that it takes a while for you to receive a `response` from the server since you have to wait until a `response` gets transformed into `json`.

```typescript
// Fetch API
fetch(ADDRESS + "restaurant")
  .then((res) => res.json())
  .then((result) => {
    this.props.setStore(result);
  });
```

However, `Axios` converts the data to `json` automatically, so you have one less step to perform. This means that using a `response` from the server would not take as long.

```typescript
// Axios
if (storeId === undefined) {
  axios(ADDRESS + "restaurant").then((res) => {
    this.props.setStore(res);
  });
}
```

2. Built-in CSRF Protection
   CSRF stands for Cross Site Request Forgery. For example, after you log in to your bank account, a hacker may send you a link to retreive the token you used to log in to your bank account. Then, the hacker can use your token to log in to your account.

   But, `Axios` has a built-in protection against it in order to protect your bank account information in the case above, or your server as well.

3. Monitor POST Request Progress
   Apparently you can see the process of receiving a response after sending a `POST` request, but have not experienced this yet. If I get to experience this, I will make sure to make a post about it.
