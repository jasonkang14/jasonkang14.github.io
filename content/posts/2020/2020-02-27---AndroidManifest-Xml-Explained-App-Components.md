---
title: React Native - AndroidManifest.xml Explained [2]
date: "2020-02-27T14:53:37.121Z"
template: "post"
draft: false
slug: "/react-native/androidmanifest-xml-explained-app-components"
category: "ReactNative"
tags:
  - "React Native"

description: "AndroidManifest.xml app components explained"
---

For each app component like `activity` and `service` a corresponding XML element must be declared in the manifest file

# Activities

### TL;DR

An entry point for interacting with the user. It represents a single screen with a user interface

### Explanation

An email app can have two different activities: one to read emails, and the other to write an email. The two activities work together to form a cohesive user experience, but each one is independent of the others. Therefore, each activity must be declared separately. You can also add an ativity to access a camera in order to attach a picture or a video.

What to consider when making an activity

1. Keep track of the current screen to ensure the system runs the process that is hosting the current activity
2. Know that stopped activities contain data the user may return to -> store the data
3. Help the app handle kill an activity to allow users to return to activities with their previous state restored
4. Provide a way for apps to implement user flows betweeen each other

# Services

### TL;DR

A general-purpose entry point for keeping an app running in the background

### Explanation

Since a `service` tag is used to run an Android app in the background, a `service` tag does not provide a user inteface. For example, a `service` tag may play some music in the background while the user is in a different app. Another component like an `activity` can start the `service` and let it run or bind to it in order to interact with it.

Two semantics services tell can tell tye system about how to manage an app:

1. Started services tell the system to keep them running until their work is completed. Could be used to sync some data in the background or play music even after the user leaves the app
   - Music playback is something a user is directly aware of
   - A regular background service is something tthat a user is unaware of

2) Bound services run because another app or the system has said that it wants to make use of the service. It is like providing an API for an app to use in the background.

# Activating Components

`Activities` and `services` are activated by an asynchronous message called an `intent`, which binds individual components to each other at runtime. `Intent` defines the action to perform and may specify the URI of the data to act on. An `intent` may convey a request for an `activity` to show an image or to open a webpage. You can also start an `activity` to receive a result in which case the `activity` also returns the reuslt in an `intent`

# AndroidManifest.xml

If a `component` is declared in `AndroidManifest.xml` without a specific package name but uses a `.` instead, it assumes that the app's package name must be used. An `Intent` object is defined with `<intent-filter>` element in `AndroidManifext.xml`
