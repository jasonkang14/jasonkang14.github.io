---
title: React[02] Remaking Miniter Using React II - Components and Props
date: "2019-06-23T14:56:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Remaking-Miniter-Using-React-part-two"
category: "React"
tags:
  - "React"

description: "Remaking Miniter Using React"
---

React app consists of multiple `components` and they transfer imformation in the format of `props` (`props` stands for properties). To be exact, a parent component transfers its infomration to its children as `props`

#1. Function and Class
you can write a function to define a component

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

you can also use an ES6 class to define a component

```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

From my own understanding, it is better to use an ES6 class when you define a component, because it allows you to use `constructor()` method to set `state`, which is a topic to discuss later

#2. Composing Components
Components can refer to other components in their input, which allows us to re-use the same component for different purposes. For example, a single button component can be used for log-in, sign-up, and main-tweet pages.

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

In the code above, you can see that the `App` component refers to `Welcome` component.<br>
You can maximize the use of children components to simplify a parent component.

```
// parnet component
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

The above parent component is really long. But you can simplify it by using children component.

```
// child component called Avatar
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />

  );
}

// parent component re-written
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
