---
title: JWT Token for a Login Decorator
date: "2019-07-05T19:27:37.121Z"
template: "post"
draft: false
slug: "/posts/JWT-Token-for-a-login-decorator"
category: "JWT"
tags:
  - "Python"

description: "Implementing Authentication using JWT"
---

Using a `JWT` token for a `login decorator`is similar to checking hashed password. You just have to use a method to `decode` the token to see if a `payload` or `key` you used to `encode` your token is retrieved.

`jwt.decode(encoded_jwt, 'secret', algorithms=['HS256'])`

This is all that is to it, and it is really simple.
But the problem that I ran into while writing this was that I messed up the order of positional arguments and kept giving the `encoded_jwt` as the second argument.

I tried so hard to look this up on `StackOverflow`, but couldn't find anyone who made a stupid mistake like I did. So I fixed my code like below, and it is now working as a `login decorator`

```python
import json
import jwt
import db_settings

from django.http import JsonResponse

def login_decorator(func):

    def login_wrapper(self, request, *args, **kwargs):
        token = request.headers["Authentication"]

        team_name = "team_babKKUNG"
        key       = db_settings.LUNCHBUDDYDATABASES["jwt"]["KEY"]
        algorithm = 'HS256'

        check_auth = jwt.decode(token, key, algorithm)

        if check_auth["iss"] == team_name:
            return func(self, request, *args, **kwargs)

        else :
            return JsonResponse({"message": "승인되지 않은 사용자입니다."})

    return login_wrapper
```
