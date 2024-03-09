---
title: Login Decorator
date: "2019-07-11T20:27:37.121Z"
template: "post"
draft: false
slug: "/posts/How-to-implement-a-login-decorator"
category: "Python"
tags:
  - "Python"
description: "How to implement a login decorator"
---

The definition of decorator is a function that must execute before executing a function. Therefore, a login decorator is used where user authorization is necessary. For example, when you look up a registered-member-only event, you must be logged in, which is supposed to be done via a login decorator.

The code is like below;

```python
import json
import jwt

from django.http          import JsonResponse
from .models              import Account
from db_settings          import jwt_key
from lunch_buddy.settings import JWT_ALGORITHM
def login_decorator(func):

    def login_wrapper(self, request, *args, **kwargs):
        token = request.headers["Authorization"]

        check_auth = jwt.decode(token, jwt_key, JWT_ALGORITHM)

        if Account.objects.filter(pk = check_auth["id"]).exists():
            request.user = Account.objects.get(pk = check_auth["id"])
            return func(self, request, *args, **kwargs)

        return JsonResponse({
            "error_code": "INVALID_LOGIN"
        }, status=400)

    return login_wrapper
```

Decorators could be "nested" into multiple layers depending on how you want to use it. In this case, I did not need much nesting.

The basic logic of a login decorator is checking whether the user is a registered user or not by using a token, which is supposed to be given out by a server you have developed.

1. Receive a token from `request headers`
2. `Decode` using a `jwt` method
3. Check if he or she has been registered.

One thing that I find very clever is the use of a `primary key` to distinguish different users as you save the data to the database. As you use the token to see if a user with such `primary key` exists, you give the user authorizaion/authentication to access a page which requires a login.
