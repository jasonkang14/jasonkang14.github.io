---
title: React Native Facebook Login (iOS only)
date: "2019-08-01T14:27:37.121Z"
template: "post"
draft: false
slug: "/posts/React-Native-Facebook-Login-Front-End"
category: "React Native"
tags:
  - "React Native"

description: "How to implement Facebook Login using React Native."
---

I struggled a lot to implement Facebook Login using React Native. I followed three lectures on YouTube and saw it working in the video, but it didn't work on my screen. I followed the instruction given on [facebook developer](https://developers.facebook.com/) very closely as well, but it did not work. After Googling different errors, I finally made it. So I am writing this blog hoping that you won't have to go through what I had to go through.

---

If you have not already, start your react-native project by following the [react native document](https://facebook.github.io/react-native/docs/getting-started).

`react-native init project`

Now you have to install `react-native-fbsdk`, but <b>instead of using the command given in Facebook Github, use the following</b>

`npm install --save https://github.com/facebook/react-native-fbsdk.git`

If you install by `npm install --save react-native-fbsdk`, you will face an error as you add a line to `AppDelegate.m` later, and it will not get resolved.

Now you just have to follow the instruction given in the [link](https://developers.facebook.com/docs/facebook-login/ios).

1. `Create a New App` by clicking the blue button. Then you will get an `App ID`, which you will need to use later.

   ![first step image](https://scontent-gmp1-1.xx.fbcdn.net/v/t1.0-9/67658681_10219516192556353_8573416276628602880_o.jpg?_nc_cat=104&_nc_oc=AQnePz3W-UU2cpr7BW133hsH5KWArc5p8Ap_iKtWN82zavzs22deZRrzktuewYXEX_0&_nc_ht=scontent-gmp1-1.xx&oh=a87660d355748673fbe9f78aac296f09&oe=5DCF60C2)

2. Set up Your Development Environment
   Facebook for Developers give two options, but I would recommend that you use `SDK:Cocoapods` because there would be less chance for an error.
   And please follow my instruction for this specific step.

   - First, install `cocoapods` in your `ios` directory
     `cd ios && sudo gem install cocoapods`

   and then create a Podfile
   `pod init`

   `open PodFile` and add following lines into your Podfile

   ```
   pod 'FBSDKCoreKit'
   pod 'FBSDKLoginKit'
   pod 'FBSDKShareKit'
   ```

   then install the dependencies
   `pod install`

I would recommend you to open your XCode and build your project after each step and check if there is an error.

3. Register and Configure Your App with Facebook

   Open your XCode in your `ios` directory. Make sure you open `yourprojectname.xcworkspace` of which color is white instead of the other one. I used the other file so many times, and it never worked.

   Copy your `Bundle Identifier` from step 3 of the first image, and then add it to the Facebook for Developers page like the second image.

   ![first image](https://scontent-gmp1-1.xx.fbcdn.net/v/t1.0-9/67744521_10219518396571452_630729407845105664_o.jpg?_nc_cat=104&_nc_oc=AQnMR8heOu39t9CIRsT9MSU9iahkUvkrqpcw5bBhhlLQEUg3N5UBGLWBcxHIIQQx2JE&_nc_ht=scontent-gmp1-1.xx&oh=8ae4a328aeac6e7db2cb04f6626c77de&oe=5DE61F9D)

   ![second image](https://scontent-gmp1-1.xx.fbcdn.net/v/t1.0-9/67740893_10219518396411448_6746216577638596608_o.jpg?_nc_cat=104&_nc_oc=AQly76cQX7bMaNYZfvrAFwm970_ww5mg9ydtOdjH5R2qckUg3ydDHZLukxEdEvLCvto&_nc_ht=scontent-gmp1-1.xx&oh=65cf70420eb12f4891548594ba22df04&oe=5DE6FD67)

   Then enable the single sign on like the image below;

   ![third image](https://scontent-gmp1-1.xx.fbcdn.net/v/t1.0-9/67930407_10219518407251719_4004078549840429056_n.jpg?_nc_cat=109&_nc_eui2=AeEIib0lD_vMZ6ne5JxFg5o47E7WevsHPx67TN08ohNukCVDFYeLWYYpQhprN2fy9vpkHtkc34qAMWKglQJaz6wPk5jHsaGVUzM-vux3ezEXrA&_nc_oc=AQkv2J7J1_zmrFd5nxAtL6fWHAGupYQQ8W6i-y1srl59IC22mpKWwH6paF58DQVg4N8&_nc_ht=scontent-gmp1-1.xx&oh=5ba492d6820eaab3e003ca567a7287c4&oe=5DD24ED6)

4. Follow the instruction given in 4a. 4b has already been done when you `react-native init`, so you don't have to worry about it

5. Connect Your App Delegate
   Add `#import <FBSDKCoreKit/FBSDKCoreKit.h>` to the top of your `AppDelegate.m` and then copy the code given in Facebook for Developers and paste it to your `AppDelegate.m` like below

```swift
@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:self launchOptions:launchOptions];
  RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                   moduleName:@"sideProject"
                                            initialProperties:nil];
  [[FBSDKApplicationDelegate sharedInstance] application:application
                           didFinishLaunchingWithOptions:launchOptions];

  rootView.backgroundColor = [[UIColor alloc] initWithRed:1.0f green:1.0f blue:1.0f alpha:1];

  self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
  UIViewController *rootViewController = [UIViewController new];
  rootViewController.view = rootView;
  self.window.rootViewController = rootViewController;
  [self.window makeKeyAndVisible];
  return YES;
}

- (BOOL)application:(UIApplication *)application
            openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {

  BOOL handled = [[FBSDKApplicationDelegate sharedInstance] application:application
                                                                openURL:url
                                                      sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]
                                                             annotation:options[UIApplicationOpenURLOptionsAnnotationKey]
                  ];


  // Add any custom logic here.
  return handled;
}

```

6. Ignore the rest of the instruction given in the link and copy the code from Facebook Github. Follow the [Usage](https://github.com/facebook/react-native-fbsdk#usage) section
