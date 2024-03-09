---
title: React Native - AndroidManifest.xml Explained [1]
date: "2020-02-20T14:53:37.121Z"
template: "post"
draft: false
slug: "/react-native/androidmanifest-xml-explained-package-name"
category: "ReactNative"
tags:
  - "React Native"

description: "AndroidManifest.xml package explained"
---

# TL;DR

`AndroidManifest.xml` describes essential information about an app to the Android build tools

# Explanation

What a `AndroidManifest.xml` looks like

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.voting_frontend">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
      android:name=".MainApplication"
      android:label="@string/app_name"
      android:icon="@mipmap/ic_launcher"
      android:roundIcon="@mipmap/ic_launcher"
      android:allowBackup="false"
      android:usesCleartextTraffic="true"
      android:theme="@style/AppTheme">
      <activity
        android:name=".MainActivity"
        android:label="@string/app_name"
        android:configChanges="keyboard|keyboardHidden|orientation|screenSize"
        android:windowSoftInputMode="adjustResize">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
      </activity>
      <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
    </application>

</manifest>
```

1. `package="com.voting_frontend"` is the app's package name. The Android build tools use this to tdetermine the location of code when building an Android project. When packaging the app, the build tools replace this package name with the application ID from the Gradle build files. Google Play Store cannot publish two different apps with a same package name.

   - The Android build tools use the app name to create a `R.java` class like `com.voting_frontend.R`
   - The package name is also used for `.MainActivity` like `com.voting_frontend.MainActivity`
   - so if you change the package name, it will throw off Android build tools when creating the APK
   - The `package` value is replaced with the `applicationId` value in `android/app/build.gradle`

2. `Components` like `activitiy`, `service`, `broadcast receiver`, and `content provider`. Each component must define basic properties such as the name of its `Kotlin` or `Java` class. The above example defines `MainApplication.java` and `MainActivity.java`

3. The `permissions` that the app needs in order to access protected parts of the system

4. The `hardware` and `software` features the app requires
