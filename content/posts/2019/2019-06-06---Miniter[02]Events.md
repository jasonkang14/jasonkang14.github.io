---
title: Miniter[02] Events
date: "2019-06-06T11:50:32.169Z"
template: "post"
draft: false
slug: "/posts/Miniter-Events/"
category: "DOM"
tags:
  - "HTML"

description: "How to set events in attempt to make a mini-version of Twitter."
---

I had to make some events such as `keyup` and `click` for my Miniter. <br>
I used `addEventListner` on certain elements in order to give events to them so that they may do what I want them to do.

#1. keyup, keydown, onKeyUp, onKeyDown
Change the color of a button from gray to blue if all cateroies are filled.<br>
Below is the code I wrote to accomplish this

```
// HTML
<!DOCTYPE html>
<html lang="en">

<body>
  <div class="container">
    <img class="logo" src="https://upload.wikimedia.org/wikipedia/fr/thumb/c/c8/Twitter_Bird.svg/944px-Twitter_Bird.svg.png">
    <header><b>Log in to Miniter</b></header>
    <form>
      <div class="input-wrap">
        <input type="text" placeholder="Enter ID" class="enterId" autocomplete="username">
      </div>
      <div class="input-wrap">
        <input type="password" placeholder="Password" class="enterPassword" autocomplete="current-password">
      </div>
      <div class="input-wrap">
        <button class="login-btn"><b>Log in</b></button>
      </div>
    </form>
    <div class="input-wrap">
      <a href="./signup.html">Sign Up for Miniter</a>
    </div>
  </div>
  <script src="js/login.js"></script>
</body>
</html>

```

```
// CSS
.login-btn {
  border: none;
  border-radius: 30px;
  color: white;
  font-size: 20px;
  text-align: center;
  background-color: gray;
}

button:hover, a:hover {
  cursor: pointer;
  opacity: 0.7;
}
```

```
// JavaScript
const elLoginBtn = document.querySelector('.login-btn');
const elInputId = document.querySelector('.enterId');
const elInputPassword = document.querySelector('.enterPassword');

const changeBtnColor = () => {
  if(elInputId.value !== "" && elInputPassword.value !== "") {
    elLoginBtn.style.backgroundColor = "#0099ff";
  } else {
    elLoginBtn.style.backgroundColor = "gray";
  }
};

elInputId.addEventListener('keyup', changeBtnColor);
elInputPassword.addEventListener('keyup', changeBtnColor);
```

I used `document.querySelector()` instead of `document.getElementById()` or `document.getElementsByClassName()` for the consistency purpose.

I also used `keyup` instead of `keydown` since `keyup` triggers an event when you release a key that just pressed. <br>
I tried using `keydown` first because `keydown` triggers an event when you press down a key.<br> However, it required me to press an extra key in order to trigger the event. I believe that is because when `keydown`is used, the function `changeBtnColor` is executed before a key gets inserted as a value.

#2. click, onClick
Added a `click` event to generate a tweet.

```
const makeTweetList = (obj) => {
  let tweet =
  `
    <span class="user">${obj.user}</span>
    <span class="date">${obj.date}</span>
    <div class="content">${obj.contents}</div>
  `
  return tweet;
};

const makeNewTweet = () => {
  count++;
  if (elTextarea.value === "") {
    alert("write your tweet");
    event.preventDefault();
    return;
  }

  const newTweet = document.createElement('li');
  newTweet.className = "tweet";
  let tweetObj = {
    user: elUserName.innerHTML,
    date: getDate(),
    contents: elTextarea.value
  }

  newTweet.innerHTML = makeTweetList(tweetObj);
  elTweetList.insertBefore(newTweet, elTweetList.childNodes[0]);
}

elMakeBtn.addEventListener('click', makeNewTweet);
```

I decided to use `backtick` to create HTML elements and put them directly into the HTML using DOM as shown in the function `makeTwitList(obj)`. <br>
The function `makeTwitList(obj)` is called from function `makeNewTweet()` after receiving input values upon `click`. <br>
`makeNewTweet()` function stores input values into an object and sends the object to `makeTwitList(obj)` as a parameter so that the callback function may require data from the object.

Then the returned value gets assigned as `innerHTML` of a `newTweet` element. `insertBefore()`was used so that a new tweet may get stacked upon old tweets.

#3. event.stopPropagation()
I was trying my best--ok I admit not my utmost best-- to make it look similar to the actual Twitter. So I added a feature to enlarge the textarea when someone clicks the textarea to make a new tweet and return it to its original size if elsewhere is clicked. Below is my code

```
//HTML

<div class="enterNewTweet">
  <textarea placeholder="What's happening?" class="newMessage"></textarea>
  <div class="toolBar">
    <button class="make-btn">Tweet</button>
  </div>
</div>
```

```
//CSS

textarea {
  border-radius: 5px;
  border: 3px solid #99d6ff;
  height: 50px;
  width: 90%;
  margin: 15px auto;
  padding: 10px;
  resize: none;
  font-size: 20px;
}

textarea::placeholder {
  font-size: 20px;
}
```

```
//JavaScript

const enlargeTextarea = () => {
  event.stopPropagation();
  elTextarea.style.height = "150px";
  elToolBar.style.display = "block";
}

elTextarea.addEventListener('click', enlargeTextarea);

document.addEventListener('click', () => {
  elTextarea.style.height = "50px";
  elToolBar.style.display = "none";
});
```

I added a `click` event to `document` to change the size if any areas rather than the textarea was clicked. And this was a problem `document` includes `textarea` as well. <br>This is where `event.stopPropagation()` comes in. According to MDN, `event.stopPropagation()` prevents further propagation of the current event in the capturing and bubbling phase.<br>
This means that when `textarea` is clicked, it stops the current event, which is the event set on `document`, so that the event on `textarea` may take place.
