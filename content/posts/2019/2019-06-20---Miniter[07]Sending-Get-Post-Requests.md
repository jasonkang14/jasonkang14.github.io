---
title: Miniter[07]Sending Get/Post Requests
date: "2019-06-20T21:56:37.121Z"
template: "post"
draft: false
slug: "/posts/Miniter-Sending-Get-Post-Requests"
category: "JavaScript"
tags:
  - "JavaScript"

description: "Sending a post request to a locally created API to build log-in, sign-up, and making tweets."
---

There are two ways to `fetch` data from server:`get` and `post`.<br>
Thankfully the methods are very straight-forward. `get` is used in order to merely get data, and `post` is used to interact with the server by posting some data and retrieve it back.

`get` was used to display previously generated tweets on screen, and `post` was used to sign up, log in, and generate new tweets.

#1. `Get` to display previously generated tweets
Here is the code for the `get` method;

```
fetch ('http://localhost:8000/api/tweet', {mode: 'cors'})
  .then (
    (response) => {
      response.json().then((data) => {
        for (let i=data.length-1; i>=0; i--) {
          const originalTweet = document.createElement('li');
          originalTweet.innerHTML = makeTweetList(data[i]);
          elTweetList.appendChild(originalTweet);
        }
      });
    }
  )
```

1. `fetch` information from the url
2. `then` wait for a `response`
3. `then` use the `data` from the `response` in order to do something.

Here is the back-end part:

```
class Tweet_class(View):
    def get(self, request, *args, **kwargs):
        tweet_list = list(Tweet.objects.values())
        return JsonResponse(tweet_list, safe=False)
```

when the server gets a `get` request, it returns tweet_list, which is in the form of an array(or a list in python) to the front-end. Therefore, the fetched `data` looks like an array.

![fetched data](https://scontent-hkg3-1.xx.fbcdn.net/v/t1.0-9/65284788_10219135481398812_1402279772439969792_o.jpg?_nc_cat=101&_nc_oc=AQlnw2un3dSg9S6vSmjPY2OdcZkfxMePJvhGY1XZnJli6Cejciwe2DmA25wqszg89WQ&_nc_ht=scontent-hkg3-1.xx&oh=6bab252688ffa04c297e64b0b83a01b0&oe=5DC1DE49)

The retrieved data ges paired up with previously written code to display previously generated tweets.

#2. `Post` to interact with the server.
Here is how I wrote a JavaScript code for sign-up.

```
const signUp = () => {
  fetch('http://localhost:8000/api/account', {
    method: 'post',
    headers: {
      "Content-type": "application/x-www-form-urlencoded; charset=UTF-8"
    },
    body: JSON.stringify({
      "user" : elNewId.value,
      "name" : elNewName.value,
      "password" : elNewPassword.value,
      "profile" : elNewProfile.value
    })
  })

  .then(
    (response) => {
      console.log(1);
      response.json().then((data) => {
        console.log(data);
        alert("회원가입 성공");
      })
  })

  .catch(function (error) {
    console.log('Request failed', error);
  });
}
```

`signUp()` gets invoked when sign-up button is clicked. In order to sign up for Miniter,

1. `fetch` information from the url
2. `post` your data (userid, name, password, profile)
3. `then` wait for a `response`
4. `then` use the `data` from the `response` in order to do something.

When a `post` request is made from the front-end, the back-end receives data and stores it in the form of a list (or an array in JavaScript).

![sign up](https://scontent-hkg3-1.xx.fbcdn.net/v/t1.0-9/64495197_10219135524159881_7340781733786157056_n.jpg?_nc_cat=100&_nc_oc=AQnFZmh8ODTkhgdufGuYGOThYTRTOXGMbso3gVjVDtF8cXxXrVkhf7s5vYRsiel7R88&_nc_ht=scontent-hkg3-1.xx&oh=1c2a093e2bb525bcbf3447d4c5dfa806&oe=5D8C99A4)
