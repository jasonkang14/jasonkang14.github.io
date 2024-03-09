---
title: React Native with Styled Component
date: "2020-05-20T21:53:37.121Z"
template: "post"
draft: false
slug: "/react-native/how-to-set-up-styled-component"
category: "REACTNATIVE"
tags:
  - "React Native"

description: "How to set up a React Native project with Styled Components"
---

[Styled-Components](https://styled-components.com/) is now a pretty big deal as CSS in JS has caught more attentions from web developers worldwide. I find this pretty helpful especially I might have to build a web page using React and an app using React Native. If I were to use the conventional `StyleSheet` for my React Native project, I would have to create a completely new css file in order to handle all the designs for my React project. So I decided to use **styled-components** in order to save myself some time.

##1. Init your react-native project

`npx react-native init AwesomeTSProject --template react-native-template-typescript`

This is what you would do if you were to init your project with the most recent react-native version. But I had to use 0.60.2 for my project. In that case you can do it like this.

`npx react-native init AwesomeTSProject --version 0.60.2 --template react-native-template-typescript`

##2. Install Styled Components
You have to install both the regular one and the one that supports type if you were to use TypeScript
`yarn add styled-components @types/styled-components`

##3. Implement your style using styled-components

The regular `styled-components` library supports a web project like React. In order to apply styled-components for your React Native project, you have to import your components from `styled-components/native` like below;

```typescript
import styled from "styled-components/native";

const StyledView = styled.View`
  ${(props) =>
    props.class === "mainOtherText" &&
    `
      padding-top: 104
      padding-bottom: 22
      padding-right:${props.paddingRight}
      padding-left:${props.paddingLeft}
    `}
  ${(props) =>
    props.class === undefined &&
    `
      flex-direction: ${props.flexDirection || "column"}
      background-color: ${props.backgroundColor || "transparent"}
      margin-vertical: ${props.marginVertical || 0}
      margin-horizontal: ${props.marginHorizontal || 0};
      padding-vertical: ${props.paddingVertical || 0};
      padding-horizontal: ${props.paddingHorizontal || 0};
      padding-top: ${props.paddingTop || 0}
      padding-bottom: ${props.paddingBottom || 0}
    `}
`;
```

I have actually integrated TypeScript into my project. I will write about using TypeScript with styled-components in a later post.
