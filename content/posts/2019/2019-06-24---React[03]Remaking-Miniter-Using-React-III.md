---
title: React[03]Remaking Miniter Using React III - State and Lifecycle
date: "2019-06-24T22:56:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Remaking-Miniter-Using-React-part-three"
category: "React"
tags:
  - "React"

description: "Remaking Miniter Using React"
---

`state` is a way to store and update data or information that a component has. <br>
As mentioned in a previous post, a `component` must be defined as a `class component` in order to use `state`

#1. Adding Local State to a Class
Use a `class constructor` in order to assign initial `this.state` to a component.

```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

Class components should always call the base constructor with `props`. by passing `props` to the base constructor, it -- but why?

#2. Adding Lifecycle Methods to a Class
Using the example of the `Clock` component writeen in section 1, <br>
`mounting`: set up a timer whenever the `Clock` is rendered to the DOM for the first time <br>
`unmounting`: clear the timer whenever the DOM produced by the `Clock` is removed

this is the order of how React inserts components into the DOM: <br>

- constructor() -> componentWillMount() -> render() -> componentDidMount()

Like below, you can do `mounting` and `unmounting` by using `componentDidMount()` and `componentWillUnmount()` respectively.

```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

#### `compononetWillMount()`

- called before the `render()` method
- only called once in a life of a component
- therefore, no access to the DOM

#### `compononetDidMount()`

- called after the `render()` method
- use this method if your initialization relies on the DOM (different from `componentWillMount()`)
- able to set the state with `this.setState()`, which will trigger a re-render, therefore, displayed on the browser
- `fetch` data from a server

#### `componentWillUnmount()`

- called right before React unmounts and destroys components
- can't set state before unmounting
- remove `event listeners` added in `componentDidMount()`
- cancelling active network requests
- cleaning up DOM elements created in `componentDidMount()`
