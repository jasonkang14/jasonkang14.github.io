---
title: React[05]Remaking Miniter Using React V - List and Keys
date: "2019-06-26T20:56:37.121Z"
template: "post"
draft: false
slug: "/posts/React Remaking-Miniter-Using-React-part-five"
category: "React"
tags:
  - "React"

description: "Remaking Miniter Using React"
---

A React list is like a JavaScript array, but it is recommended--not required--to provide a key to each element of a list. The key provided to each elemenet must be unique to the element. React executes the code whether a unique key is assigned to each element or not, but it keep throwing a warning if it is not.

#1. Keys
According to the React official document, key helps React identify which items have changed and are added/removed. Which is why keys have to be unique in order to distinguish which items have been affected by a change.

```
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

I have said that React will still execute your code even if a list doesn't have a key assigned to it, and that is because React automatically assigns an index as a key to each element, but there are some negative impacts to it, which is why you gotta get some unique keys to each and every item. Details can be found [here](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318)

It is recommended to use `shortid`, which generates short non-sequential url-friendly unique keys like below

```
var shortid = require('shortid');
function createNewTodo(text) {
  return {
    completed: false,
    id: shortid.generate(),
    text
  }
}
```

#2. Keys with `map()` method
to display tweets for my miniter project, I made each tweet as an object and nested it into an array. Instead of requiring `shortid`, I decided to use the indexes of the array with a prefix like below.

```
{
  this.state.tweetArr.map((el, idx) =>
          <DisplayTweets
              key={`tweet-${idx}`}
              tweet={el}
          />

  )
}
```

if I were to connect this with an API, it would be better to require `shortid`, but for the sake of this specific project, I decided to go with this.
