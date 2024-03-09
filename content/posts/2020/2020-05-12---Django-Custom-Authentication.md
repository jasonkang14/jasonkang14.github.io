---
title: Django - Custom Authentication
date: "2020-05-12T21:53:37.121Z"
template: "post"
draft: false
slug: "/django/django-custom-authentication"
category: "Django"
tags:
  - "Django"

description: "How to create a custom user model and authentication method in Django"
---

### Django's official documentation about customizing authentication can be found [here](https://docs.djangoproject.com/en/3.0/topics/auth/customizing/#customizing-authentication-in-django)

In order to create custom authentication, you have to create a model. You can extend the existing **User** model from `django.contrib.auth.models` and create an one-to-one relationship with your model and the default **User** model. However, I cannot assure this because I ahve not done it myself, but I believe this would create an extra table in your database. So I have decided to substitute a custom **User** model by using **AbstractUser** from `django.contrib.auth.models` like below;

```python
class User(AbstractUser):
    nickname      = models.CharField(max_length=15, unique=True)
    phone         = models.CharField(max_length=15, default="010-1234-1234", unique=True)
    date_of_birth = models.DateField()
```

`birthday` could be a better name for the field, but I just decided to go with `date_of_birth`. Unless you create a completely new model for your `User`, the default **User** object has some primary attributes like **username**, **password**, **email**, **first_name**, and **last_name**. The model also has some other attributes like **is_superuser**, **is_staff**, and so on.

Now you have to authenticate this user via a sign up request. I decided to go with a class view because I wanted to use a `get` request for sign up to check if a user with the same username, nickname, and/or email already exists.

```python
import json

from django.http         import JsonResponse, HttpResponse
from django.views        import View
from django.contrib.auth import authenticate, login

from .models             import *                                 ## I take Align very seriously

class SignupView(View):
    def post(self, request):
        new_user_info = json.loads(request.body)
        req_username  = new_user_info['username']
        req_phone     = new_user_info['phone']
        req_email     = new_user_info['email']

        if User.objects.filter(username=req_username).exists():
            return JsonResponse({
                'message': 'USERNAME_ALREADY_EXISTS'
            }, status=409)

        elif User.objects.filter(phone=req_phone).exists():
            return JsonResponse({
                'message': 'PHONE_NUMBER_ALREADY_EXISTS'
            }, status=409)

        elif User.objects.filter(email=req_email).exists():
            return JsonResponse({
                'message': 'EMAIL_ALREADY_EXISTS'
            }, status=409)

        else:
            new_user = User.objects.create_user(
                username      = req_username,
                phone         = req_phone,
                email         = req_email,
                first_name    = new_user_info['first_name'],
                last_name     = new_user_info['last_name'],
                password      = new_user_info['password'],
                nickname      = new_user_info['nickname'],
                date_of_birth = new_user_info['dateOfBirth'],
                height        = new_user_info['height'],
                body_type     = new_user_info['bodyType'],
                occupation    = new_user_info['occupation'],
            )

            new_user = authenticate(username=req_username, password=new_user_info['password'])

            return HttpResponse(status=200)
```

I used 409 status if a user with same info already exists because `409 Conflict` is used in situations **where it is iexpected that the user might be able to resolve the conflict and resubmit the request**. Since the user can enter a new username, nickname, and/or email to resolve the **conflict** I used 409 as the status code.

After creating a user by using `User.objects.create_user`, you have to **authenticate** the user via `new_user = authenticate(username=req_username, password=new_user_info['password'])` in order to set the password. I will write about login in a later post
