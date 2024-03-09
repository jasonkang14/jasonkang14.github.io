---
title: Django - Query Set API
date: "2020-06-02T15:53:37.121Z"
template: "post"
draft: false
slug: "/django/query-set-api"
category: "Django"
tags:
  - "Django"

description: "Methods you can use to make queries in Django"
---

### exclude()

very straight forward. you can exclude multiple properties using a comma like below
`Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3), headline='Hello')`

### order_by()

getting a query set in an ascending or a descending order. `order_by(id)` will give you a query set in an ascending order, and `order_by(-id)` in which you use a negative sign will give you a query set in a descending order.

You can add multiple parameters using a comma like below as well.

`order_by(-id, headline)`

In the case above, your query set will be organized in a descending order of id, and then in an ascending order of headline afterwards.

You can also use a double underscore **(\_\_)** if you want to query by joining a table

There are also expressions like `asc()` and `desc()` of which roles are very straight forward. These two methods are helpful because they allow you to have `null` values at the beginning of your query set or at the end of your query set by using **nulls_first** or **nulls_last** arguments.

### reverse() and distinct()

straight forward. **reverse()** does not take any argument, but **distinct()** does.

### values()

probaly the one that is used the most. you can get a queryset of dictionaries. arguments are optional in which the entire row will be retrieved. But you can also add arguments if you want specific columns from the queryset. You can use a comma if you want to get multiple values from the query set.

### values_lst()

you get a query set of tuples instead of dictionaries. if you add the `flat=True` argument, you get a dictionary in which all the tuples are concated. if you add the `named=True` argument, you get named tuples instead.
