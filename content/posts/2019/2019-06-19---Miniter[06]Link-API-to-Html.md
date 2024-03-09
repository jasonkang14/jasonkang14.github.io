---
title: Miniter[06]Making an API Using Django without REST framework II
date: "2019-06-19T22:36:37.121Z"
template: "post"
draft: false
slug: "/posts/Miniter-Link-API-to-HTML"
category: "Django"
tags:
  - "Django"

description: "Making an API for the Miniter project with Django without using the REST framework."
---

Ran into so many errors while doing this assignment. Let me try to walk you through it.

#1. CSRF Failed: CSRF token is incorrect or missing
CSRF stands for Cross Site Request Forgery, which is an attack that tricks a web browser into executing an unwanted action. CSRF token first appeared during the second generation of the web development where front-end and back-end developments were done on the same server.

Anti-CSRF tokens were developed for a security reason that an attacker, who knows the form of a web, could request something to the server and do something that a web developer did not intend. Therefore, CSRF protection ensures that only the form that the web developer of the webpage created may send requests to the server.

However, the protection is no longer needed since front-end and back-end developments are done separately. You can disable CSRF validation by disabling csrf section in `MIDDLEWARE` in `settings.py` like below.

```
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'api.middleware.DisableCsrfCheck',
    #'django.middleware.csrf.CsrfViewMiddleware',       ## to disable CSRF validation
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

#2. CORS Error
CORS stands for Cross-Origin Resource Sharing, which is a standard that allows a server to relax the same-origin policy. The same-origin policy is a security mechanism that restricts how a document/script loaded from one origin can interact with a resource from another origin. CORS allows some cross-origin requests while rejecting others.

Cross-Site Request Forgery is an attempt to take advantage of the browser's cookie storage system. A browser would store relevant session cookie when someone signs into a web application. And everytime he or she revisits the web the API will recognized the stored session cookie upon further HTTP requests.

The problem is that the browser automatically includes any relevant cookies stored for a domain when another request is made to that exact domain. Therefore, when an attacker sends a request to the API, the browser includes the relevant cookies. The attacker gains authenticated access to the website.

The easiest way to solve the problem would be installing the CORS plugin on your browser. However, this does not fix everything because this would mean that your code only works on computers which have the plugin installed. Therefore, it is the best to fix the problem on the back-end side in order to ensure that anyone who requests such API may not have a problem.

It is simple. You just have to add a package that allows CORS.

```
## settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'api',              ## name of the app
    'corsheaders',      ## added to allow CORS
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'api.middleware.DisableCsrfCheck',
    #'django.middleware.csrf.CsrfViewMiddleware',         ## added to disable CSRF validation
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'corsheaders.middleware.CorsMiddleware',              ## added to allow CORS
]


CORS_ORIGIN_ALLOW_ALL=True       # added to the bottom of settings.py
```
