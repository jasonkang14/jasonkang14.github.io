---
title: React[06]Remaking Miniter Using React VI - Forms
date: "2019-06-27T19:56:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Remaking-Miniter-Using-React-part-six"
category: "React"
tags:
  - "React"

description: "Remaking Miniter Using React"
---

I used three HTML form elemements for this proejct.

#1. Textarea
A textarea component looks like this;

```
import React from 'react';

const Textarea = props => {
    return (
        <textarea
            value={props.value}
            className={props.className}
            placeholder={props.placeholder}
            onChange={props.handleInput}
        />
    );
}

export default Textarea;
```

`value` is to handle inputs to the text area <br>
`className` is to apply CSS<br>
`placeholder` is for placeholder<br>
`onChange` is to use `setState()` method to change the state of `value`<br>
Since Textarea does not need a constructor, I used a function format in order to define the component. Passing props allows me to omit `this`, which normally comes in front of `props`, making the code slightly more efficient.

The parent component of the textarea code is below;

```
handleInputChange = (event) => {
        this.setState ({
            value: event.target.value
        })
    }

...

<Textarea
    className="newMessage"
    placeholder="What's happening?"
    value={this.state.value}
    handleInput={this.handleInputChange}
/>
```

When an input is made into the textarea, it runs `handleInputchange()` and changes the state of `value`, which will be used as contents for a tweet.

#2. Input
This is used for id and password section.

```
import React from 'react';

const Input = props => {
    return (
        <input
            className={props.className}
            type={props.type}
            name={props.name}
            placeholder={props.placeholder}
            autoComplete={props.autoComplete}
            onChange={props.changeInput}
        />
    );
}

export default Input;
```

the only thing that is different is `type` since an `input`element can be of different types. <br>
also added `autoComplete` upon the advice of Chrome brower.

```
<form>
    <Input
    type="text"
    placeholder="Enter ID"
    className="enterId"
    name="userId"
    autoComplete="username"
    changeInput={this.handleChange}
    />

    <Input
    type="password"
    name="password"
    placeholder="Password"
    className="enterPassword"
    autocomplete="current-password"
    changeInput={this.handleChange}
    />
</form>
```

very similar to `textarea`, it takes an `onChange` attribute to run `handleChange()` so that input value to the `input` elements can be used to change the `state` of value

#3. Button

```
import React from 'react';

const Button = props => {
    return (
        <button
            className={props.className}
            name={props.name}
            onClick={props.btnClicked}
        >
            {props.innerHTML}
        </button>
    );
}

export default Button;
```

`button` elements have an `onClick` event to take care of it.

```
<Button
    className="make-btn"
    name={`${this.state.mode === "generate" ? "makeTweetBtn" : "updateTweetBtn"}`}
    innerHTML={`${this.state.mode === "generate" ? "Tweet" : "Update"}`}
    btnClicked={this.state.mode === "generate" ? this.generateNewTweet : this.updateTweet}
/>
```

Each tweet also has an `edit` button. Therefore, when the `edit` button is clicked, it changes the generate button into an edit button, which is why the conditional operator was used.
