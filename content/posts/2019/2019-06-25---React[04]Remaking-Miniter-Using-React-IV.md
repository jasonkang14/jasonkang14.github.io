---
title: React[04]Remaking Miniter Using React IV - Handling Events
date: "2019-06-25T17:56:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Remaking-Miniter-Using-React-part-four"
category: "React"
tags:
  - "React"

description: "Remaking Miniter Using React"
---

React event handlers are slighlty different from HTML event handlers

#1. React events are named using camelCase and pass a function as the event handler using JSX

notice the difference below

```
//HTML
<button onclick="activateLasers()">
  Activate Lasers
</button>

//React
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

#2. Call event.preventDefault() explicitly to prevent default behavior

```
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```

#3. binding `this`
since event handlers are added as a method on a class, binding is required in order to make `this`work in callback functions. Otherwise, when a callback function receives `this`, the `this` will be `window` instead of the class to which the method belongs.

```
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(state => ({
      isToggleOn: !state.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

if you build a habit of writing the method in an ES6 format, binding is not necessary.

```
//ES6 example I
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}

//ES6 exmample II
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // This syntax ensures `this` is bound within handleClick
    return (
      <button onClick={(e) => this.handleClick(e)}>
        Click me
      </button>
    );
  }
}
```

However, the problem with the above syntax is that a different callback is created each time `LoggingButton` renders. If this callback is passed as a prop to lower components, it might do an extra re-rendering, which would decrease the efficiency of your code. So binding is recommended.

#4. Passing arguments to event handlers
Either way is fine

```
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```
