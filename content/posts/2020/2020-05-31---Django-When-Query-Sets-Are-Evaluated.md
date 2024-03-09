---
title: Django - When Query Sets Are Evaluated
date: "2020-05-31T22:53:37.121Z"
template: "post"
draft: false
slug: "/django/when-query-sets-are-evaluated"
category: "Django"
tags:
  - "Django"

description: "Django does not access the database every single time you make a query"
---

Let's say you have this code in your django backend;

```python
q = Entry.objects.filter(headline__startswith="What")
q = q.filter(pub_date__lte=datetime.date.today())
q = q.exclude(body_text__icontains="food")
print(q)
```

It looks like you are accessing your database three times by using the code above. However, according to the offcial document of Django, Django access the database only once, at the last line `print(q)` because **the results of a queryset aren't fetched from the database until you **ask** for them**. Which means that you are asking for the data when you are trying to `print` it.

There are some methods that you can use to have Django access the database. Some common ones are default python methods like `repr()`, `len()`, `list()`, and `bool()`.

I am going to write about QuerySet APIs in a later post
