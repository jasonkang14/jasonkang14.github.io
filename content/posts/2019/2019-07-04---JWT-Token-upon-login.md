---
title: JWT Token Returned upon Login
date: "2019-07-04T23:27:37.121Z"
template: "post"
draft: false
slug: "/posts/JWT-Token-returned-upon-login"
category: "JWT"
tags:
  - "Python"
description: "Implementing Authentication using JWT"
---

`JWT` is a token that a server hands out to a user so that a user and server just exchanges the token instead of logging in every time you switch a page. After sending a `JWT` token to a user, you use a `login decorator` to check if he or she has an authentication to access a webpage.

Apparently, there's a `JWT` directly associated with `django`, but I used `PyJWT` instead.

so first install `pyjwt`
`pip install pyjwt`

This is the how you `encode` using `JWT`:
`encoded_jwt = jwt.encode({'some': 'payload'}, 'secret', algorithm='HS256')`

1. `payload` is some information that you want to send
2. `secret` is a key that you use when you encode
3. `algorithm` is how you want to encode your token

So simple. It is so easy to encode, but I faced so many errors while implementing this. But the most important thing is that you have to `decode` your `JWT` token before sending it to a user.

My first code was something like this:

```python
if bcrypt.checkpw(login_password.encode("utf-8"), registered_password.encode("utf-8")):
    payload = {
        "iss": "team_babKKUNG",
    }

    key = db_settings.LUNCHBUDDYDATABASES["jwt"]["KEY"]
    algorithm = 'HS256'

    jwt_token = jwt.encode(payload, key, algorithm)
    return JsonResponse({"token": jwt_token})
```

And below is the error that I got:
`Object of type bytes is not JSON serializable`,
which is very straight forward. you have to decode it like this in order to avoid the error and successfully.

```python
if bcrypt.checkpw(login_password.encode("utf-8"), registered_password.encode("utf-8")):
    payload = {
        "iss": "team_babKKUNG",
    }

    key = db_settings.LUNCHBUDDYDATABASES["jwt"]["KEY"]
    algorithm = 'HS256'

    jwt_token = jwt.encode(payload, key, algorithm)

    return JsonResponse({"token": jwt_token.decode("utf-8")})
```
