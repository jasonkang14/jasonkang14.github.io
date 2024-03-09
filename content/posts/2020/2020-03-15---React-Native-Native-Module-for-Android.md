---
title: React Native - How to make an Android native module
date: "2020-03-15T22:53:37.121Z"
template: "post"
draft: false
slug: "/react-native/how-to-make-an-android-native-module"
category: "React Native"
tags:
  - "React Native"

description: "How to make an Android native module for a React-Native project"
---

### This is following an example from the [official document](https://reactnative.dev/docs/native-modules-android) with some of my personal input

Before creating a native module for a react-native project, you have to understand how react-native works for an Android app. A general tree is defined in `AndroidManifest.xml` like I have posted. And how the app actually works or how it is composed is declared in `MainApplication.java`

There are two default `java` files in a react-native project. One is `MainApplication.java`, which I am going to use in order to add a native module, and the ohter is `MainActivity.java`, which you rarely deal with throughout your project. The only thing declared in `MainActivity.java` is the name of your project like below;

```java
package com.your-app-name;

import com.facebook.react.ReactActivity;

public class MainActivity extends ReactActivity {

  /**
   * Returns the name of the main component registered from JavaScript. This is used to schedule
   * rendering of the component.
   */
  @Override
  protected String getMainComponentName() {
    return "your-app-name";
  }
}
```

And everything else is declared in `MainApplication.java`. When you create a native module of your own, you have to pay attention to this section right here;

```java
    @Override
    protected List<ReactPackage> getPackages() {
        @SuppressWarnings("UnnecessaryLocalVariable")
        List<ReactPackage> packages = new PackageList(this).getPackages();

        // packages.add(new MyReactNativePackage());  // <-- this is where you are going to add your module
        return packages;
    }

```

Technically, you are not adding a module to `MainApplication.java`. You are adding a `package` which includes a `module` to `MainApplication.java`.

First, you create your module like this. The below example is directly from the [official website](https://reactnative.dev/docs/native-modules-android)

```java
// ToastModule.java

package com.your-app-name;

import android.widget.Toast;

import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;

import java.util.Map;
import java.util.HashMap;

public class ToastModule extends ReactContextBaseJavaModule {
// a native module is a Java class that extends ReactContextBaseJavaModule

  private static ReactApplicationContext reactContext;

  private static final String DURATION_SHORT_KEY = "SHORT";
  private static final String DURATION_LONG_KEY = "LONG";

  ToastModule(ReactApplicationContext context) {
    super(context);
    reactContext = context;
  }

  @Override
  public String getName() { // a ReactContextBaseJavaModule requires this method to be implemented
    return "ToastExample"; // this string represents the name of the NativeModule that you are creating
  }                        // therefore, you can access this module through React.NativeModules.ToastExample

  @Override
  public Map<String, Object> getConstants() {     // this is optional
    final Map<String, Object> constants = new HashMap<>();
    constants.put(DURATION_SHORT_KEY, Toast.LENGTH_SHORT);
    constants.put(DURATION_LONG_KEY, Toast.LENGTH_LONG);
    return constants;
  }

  @ReactMethod   // this decorator allows you to access this Java method with JavaScript. You will see what I mean later
  public void show(String message, int duration) {
    Toast.makeText(getReactApplicationContext(), message, duration).show();
  }
}
```

Now you have to create a `package` to register your `module`

```java
// CustomToastPackage.java

package com.your-app-name;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CustomToastPackage implements ReactPackage {

  @Override
  public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
    return Collections.emptyList();
  }

  @Override
  public List<NativeModule> createNativeModules(
                              ReactApplicationContext reactContext) {
    List<NativeModule> modules = new ArrayList<>();

    modules.add(new ToastModule(reactContext));  // Your module is added right here

    return modules;
  }

}
```

Now you add the `package` to `MainApplication.java`

```java
...
import com.your-app-name.CustomToastPackage; // <-- Add this line with your package name.
...

protected List<ReactPackage> getPackages() {
  @SuppressWarnings("UnnecessaryLocalVariable")
  List<ReactPackage> packages = new PackageList(this).getPackages();

  packages.add(new CustomToastPackage()); // <-- Add this line with your package name.
  return packages;
}
```

Now you create a JavaScript file to wrap the native module you have just created, which is `React.NativeModule.ToastExample`.

```javascript
//ToastExample.js

import { NativeModules } from "react-native";
module.exports = NativeModules.ToastExample;
```

This is not in the official document, but you can also do it like this

```javascript
import { NativeModules } from "react-native";

const { ToastExample } = NativeModules;

export default ToastExample;
```

You can all the `show` method which you have decorated with `ReactMethod` in `ToastModule.java` like this

```javascript
import ToastExample from "./ToastExample";

ToastExample.show("Awesome", ToastExample.SHORT);
```

This shows a toast with a text `Awesome` when you launch your Android app
