---
title: Django - Prefetch Related
date: "2020-06-07T10:53:37.121Z"
template: "post"
draft: false
slug: "/django/prefetch-related"
category: "Django"
tags:
  - "Django"

description: "Let's try to improve efficiency when you make a query in Django"
---

According to the [official document](https://docs.djangoproject.com/en/3.0/ref/models/querysets/#prefetch-related) `preftecth_related()` returns in a single batch, related objects for each of the specified lookups.

This is somewhat similar to `select_related()` as both methods save you some time by not hitting the database whenever you access a queried object.

While `select_related()` is limited to **ForeignKey** or **OneToOneField**, `prefetch_related()` can be used for **ManyToManyField** and even used for reverse lookup for many-to-one relationship which is the opposite of **ForeignKey** which represents one-to-many relationship.

It was kinda tricky to come up with my own example, so I decided to create models provided in the official document, and ran some test queries. Here are the models that I have created;

```python
class Topping(models.Model):
    name = models.CharField(max_length=25)

class Pizza(models.Model):
    name     = models.CharField(max_length=25)
    toppings = models.ManyToManyField(Topping)

    def __str__(self):
        return "%s (%s)" % (
            self.name,
            ", ".join(topping.name for topping in self.toppings.all()),
        )
```

And then I ran queries to call the `__str__()` method on the Pizza class. One using `prefetch_related()` and the other without it .

When I ran `Pizza.objects.all()`,

```
before == 2020-06-07 09:31:07.965861
pizza_info == <QuerySet [<Pizza: hawaiian (pineapple, ham)>, <Pizza: combination (ham, pepperoni, vegetable)>, <Pizza: pepperoni (pepperoni, olive, cheese)>]>
after  == 2020-06-07 09:31:07.975982
```

You can see that the query took 0.01 second as it ran `self.toppings.all()` every single time in order to get the toppings related to each pizza object that was created.

But when I ran `Pizza.objects.all().prefetch_related('toppings')`,

```
before == 2020-06-07 09:31:15.091070
pizza_info == <QuerySet [<Pizza: hawaiian (pineapple, ham)>, <Pizza: combination (ham, pepperoni, vegetable)>, <Pizza: pepperoni (pepperoni, olive, cheese)>]>
after  == 2020-06-07 09:31:15.097154
```

It took 0.006 second, which is which is about 1.6 times more efficient as the single query has already retrieved all the information about toppings. Therefore, it won't have to run `self.toppings.all()` over and over again. As you an see there are only three rows in the Pizza table, so the efficiency may not seem that much different, but if you were to have a huge dataset with more than 10,000 rows in a table, it would come in handy
