---
title: How to create a hash key for an android project
date: "2019-09-09T22:27:37.121Z"
template: "post"
draft: false
slug: "/posts/android/how-to-create-a-hash-key-for-an-android-project"
category: "Android"
tags:
  - "Android"

description: "How to create a hash key for an android project"
---

I have been building my project using React Native which allows me to build a native app for both Android and iOS. However, Android requires hash key especially when you try to use SDK to implement things like social login.

The command is really simple.

`keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android | openssl sha1 -binary | openssl base64`

This will generate a line with a hash key which I will not print here for a security purpose. You just have to add the value to the field where the hash key is required
