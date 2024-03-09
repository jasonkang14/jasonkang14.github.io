---
title: Django - Query with Filter and Get
date: "2020-05-25T19:53:37.121Z"
template: "post"
draft: false
slug: "/django/query-with-filter-and-get"
category: "Django"
tags:
  - "Django"

description: "How to make a query in Django using filter abd get"
---

`filter` is used when there are likely to be more than one query sets. It something like this

`User.objects.filter(join_date__year=2020)`

This query will give you all the users who has signed up for your server in the year of 2020. Your query is supposed to look like below;

`User.objects.all().filter(join_date__year=2020)`

However, your **Manager** class allows your query to be shorter

You can also add a condition to `exclude` certain data like below;

`User.objects.filter(join_date__year=2020).exclude(join_date__month=5)`

By using this, you are excluding all the users who have signed up in May.

There are some methods that you can find in the django official doc. In case of filtering by number, you can use `count__gte=30`, which is equivalent to count greater than 30, and also `count__lte=30`, which would be coult less than 30.

In the case of filtering by string, you can use `headline__startswith="what"`, which is straight forward, and I believe there is also `headline__endswith="what"`.

And you would use `get` when you expect to receive only one query set. For example, if you added `unique=True` to an email field, you would query your user like this;

`User.objects.get(email='test@email.com')`
