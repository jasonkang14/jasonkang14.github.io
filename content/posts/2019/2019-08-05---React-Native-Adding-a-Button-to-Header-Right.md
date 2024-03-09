---
title: React Native - Header Right Button
date: "2019-08-05T22:27:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Native-Header-Right-Button"
category: "React Native"
tags:
  - "React Native"

description: "How to use a header right button in React Native"
---

`React Native` comes with a default header, which has a back button which allows you to go back to the previous page. You can also add a button to the right side of the header by using `headerRight` property like below;

```typescript
static navigationOptions = ({ navigation }) => ({
    headerTitleStyle: {
    ...
    headerRight: (
      <TouchableOpacity
        onPress={navigation.getParam('handleClick')}
        style={{ marginRight: 17 }}
      >
        <Text
          style={[
            styles.headerRightBtn,
            navigation.getParam('checkInput')
              ? styles.applyGreen
              : styles.applyGray
          ]}
        >
          완료
        </Text>
      </TouchableOpacity>
    )
  });
```

As you can see, a method that gets called `onPress` is not a typical `this.whateverYouWouldLikeToCall`. Instead, I used `navigation` property and used `getParam`. This is because `navigationOptions` is `static`, which I will talk about in a later post.

####TL;DR
I used `navigation` in order to call a method when the header right button is pressed.
In order for the button to `getParam`, you have to `setParam` in `componentDidUpdate()` like below

```typescript
componentDidMount() {
    this.props.navigation.setParams({
      handleClick: this._makePost
    });
  }
```

So when the header right button fires `handleClick` via `getParam`, a method called `_makePost` is executed.
