---
title: Django - Models Explained
date: "2020-04-19T22:53:37.121Z"
template: "post"
draft: false
slug: "/django/django-models-explained"
category: "Django"
tags:
  - "Django"

description: "Django models explained"
---

I am trying to go through the [official documentation](https://docs.djangoproject.com/en/3.0/) of Django and read through every aspect of it in order to build a server using Django. The first section is Models

## TL;DR

A Django model becomes a table and its rows in your database.

## Explanation

1. Each model is a Python class that subclasses `django.db.models.Model`

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```

You have just created a `Person` model with fields name `first_name` and `last_name`. If you use a database, `Person` is your table, and `first_name` and `last_name` are its rows. And an `id` is added automatically as a primary key for your convenience.

But the name of the table becomes `myapp_person` instead of `person`. If you want to want a table called `person` instead, you can write your model like this by using a `Meta` class.

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)

    class Meta:
        db_table = 'person'
```

After creating a model, you have to make migrations with `manage.py makemigrations` and then `manage.py migrate`. The first command creates migration files which declares what you have done with your `models.py` like creating a table and rows or alternating rows. And the next command applies the changes to your database.

2. Fields

Each field takes a certain set of field-specific arguments. Some fields requires certain arguments while others do not. I feel like it is not really worth memorizing those, so you can find the information [here](https://docs.djangoproject.com/en/3.0/ref/models/fields/#model-field-types)

Certain arguments may come to you as you write more codes. For example, a `CharField` requires an `max_length` argument. If you add `null=True` as an argument to a field, Django will store empty values as `NULL`. If you add `blank=True` as an argument to a field, Django will store an empty value instead of using `NULL`.

There is also an argument called `choices`, which is a sequence of 2-tuples. Check the example below;

```python
class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
```

If you make a query to a `Person` class, `person.shirt_size` would give you S, M, or an L. If you call a function like `person.get_shirt_size_display()`, yow would get Small, Medium, or Large.

You can also use enumeration classes like below;

```python
from django.db import models

class Runner(models.Model):
    MedalType = models.TextChoices('MedalType', 'GOLD, SILVER, BRONZE')
    name = models.CharField(max_length=60)
    medal = models.CharField(blank=True, choices=MedalType.choices, max_length=10)
```

There is an argument called `verbose_name`, which is automatically generated if you do not specify. I think you can use this as for your reference for you to help other developers what you were trying to do with this field.
