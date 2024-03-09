---
title: React Native - AndroidManifest.xml Explained [3]
date: "2020-03-01T14:53:37.121Z"
template: "post"
draft: false
slug: "/react-native/androidmanifest-xml-explained-permissions"
category: "ReactNative"
tags:
  - "React Native"

description: "AndroidManifest.xml permissions explained"
---

An Android app requests permissions to access sensitive user data or certain system features. Each permission is identified by a unique label. This is because a central design point of the Android security architecture is that no app has permission to perform amy operations that would adversly impact other apps, the operating system, or the user.

`Permission` is declared with a `<uses-permission>` tag. A default permission requested in an Android app is
`android.permission.INTERNET`

The example above is a **normal** permission thus granted automatically. However, a **dangerous** permision like `SEND_SMS` requires an explicit agreement from a user.
