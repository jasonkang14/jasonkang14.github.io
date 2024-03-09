---
title: Request to Server using MobX
date: "2019-08-02T18:27:37.121Z"
template: "post"
draft: false
slug: "/posts/Request-to-Server-using-MobX"
category: "MobX"
tags:
  - "React"

description: "How to send a request to the server via MobX"
---

State like login must be handled globally since your login information should be accessible in every single page that requires permissions. So when you send a request to a server for login, you have to do it globally. `MobX` is the way to go.

I have already posted about how to set up your `React Native` project using `MobX`. And this post is about how you send a request.

First, you have to create a `Store` which changes a state and detects such changes.

```typescript
import { observable, action, runInAction } from "mobx";
import { LoginManager, AccessToken } from 'react-native-fbsdk';
import { API_URL } from "../../config";
import axios from "axios";

class UserStore {
    @observable user = [];
    @observable facebookLoginStatus = "";
    @observable state = "pending"

    const store = new UserStore();
    export default store;

```

`observable` in `MobX` is data of which change could be observable. When Facebook login is successful, his or her user information is going to be stored in `@observable user`

And you implement the Facebook login by using the code provided by Facebook Github. I used a custom button, so my code is like below.

```typescript
    @action
    facebookLogin() {
        LoginManager.logInWithPermissions(["public_profile"]).then(
            action ((result) => {
              if (result.isCancelled) {
                console.log("Login cancelled");
              } else {
                AccessToken.getCurrentAccessToken().then(
                   action ((data) => {
                    this.facebookToken = data.accessToken;
                    const headers = {
                            "Content-Type": "application/json",
                            "Accept": "application/json"
                        }
                    const body = JSON.stringify(this.facebookToken)

                    axios.post(`${API_URL}account/facebooklogin`, body, { headers }). then(
                        action ((response) => {
                        if (response.data.code === 0) {
                            this.state = "done";
                            this.facebookLoginStatus="fail";
                            this.user = response.data.user_info;
                        } else {
                            this.state= "done";
                            this.facebookLoginStatus="success";
                            this.user = response.data.user_info;
                        }
                    }))
                  })
                )
                console.log(
                  "Login success with permissions: " +
                    result.grantedPermissions.toString()
                );
              }
            }),
            action ((error) => {
              console.log("Login fail with error: " + error);
            })
        );
    }

}
```

`action` is anything that can modify a `state`. Since I am going to change the state called `user`, I am calling this request an `action`. I used `async/await` like above since the instruction was very clear in the [MobX website](https://mobx.js.org/best/actions.html).

And when you send a request to a server using a token from Facebook, your server will return the access token from your server, which you have to store in your storage.

One thing `React Native` is different from `React` is that you have to store the token by using `async-storage` library. I will talk about this in the next post
