---
title: Django - Relationships Explained
date: "2020-04-25T12:53:37.121Z"
template: "post"
draft: false
slug: "/django/django-relationships-explained"
category: "Django"
tags:
  - "Django"

description: "Django models: relationships like Many-to-One, Many-to-Many, and One-to-One explained"
---

### Many-to-One Relationships

In order to define a many-to-one relationship, you can use `django.db.models.ForeignKey` like below;

```python
from django.db import models

class Manufacturer(models.Model):
    company = models.CharField(max_length=25)

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    name         = models.CharField(max_length=25)
```

You just have to add the name of the model as an argument to the field. Make sure that the model you are referencing in your `ForienKey` field is delcared before. If you are creating a relationship on a model that has not yet been defined, you have to use the name of the model instead of the model object itself like below:

```python
from django.db import models

class Car(models.Model):
    manufacturer = models.ForeignKey('Manufacturer', on_delete=models.CASCADE)
    name         = models.CharField(max_length=25)

class Manufacturer(models.Model):
    company = models.CharField(max_length=25)
```

If you want to create a recursive relationship, which is an object that has a many-to-one relationship with itself, you can use `models.ForeignKey('self', on_delete=models.CASCADE)`. An example that I have found is when an employee supervies multiple employees. An employee has a many-to-one relatiponship with multiple employees.

### Many-to-Many Relationships

In order to define a many-to-one relationship, you can use `ManyToManyField` like below;

```python
from django.db import models

class Topping(models.Model):
    name = models.CharField(max_length=25)

class Pizza(models.Model):
    toppings = models.ManyToManyField(Topping)
    name     = models.CharField(max_length=25)
```

Like you would do with `ForeignKey`, you can create recursive relationships and relationships to models not yet defined.

Sometimes, you might want to use an extra table in order to describe the relationship between two models by using the `through` argument like below;

```python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name

class Group(models.Model):
    name    = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):
        return self.name

class Membership(models.Model):
    person        = models.ForeignKey(Person, on_delete=models.CASCADE)
    group         = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined   = models.DateField()
    invite_reason = models.CharField(max_length=64)
```

The `Group` model has a many-to-many relationship with the `Person` model through a model called `Membership` which has many-to-one relationships with both `Person` and `Group`

You can use `add()`, `create()`, `set()` to create relationships as long as you specify `through_defulats` like below;

```python
beatles.members.add(john, through_defaults={'date_joined': date(1960, 8, 1)})
beatles.members.create(name="George Harrison", through_defaults={'date_joined': date(1960, 8, 1)})
beatles.members.set([john, paul, ringo, george], through_defaults={'date_joined': date(1960, 8, 1)})
```

If you call `remove()`, all intermediate model instances related to the model will be removed. If you call `clear()`, all many-to-many relationships for an instance will be deleted.

f`{modelname}_set` like `membership_set` can be used in order to mane a query on `Person` model

```python
ringos_membership = ringo.membership_set.get(group=beatles)
ringos_membership.date_joined   #  datetime.date(1962, 8, 16)
```

### One-to-One Relationships

In order to define a many-to-one relationship, you can use `OneToOneField`. This is very straight-forward. Like you would do with `ForeignKey`, you can create recursive relationships and relationships to models not yet defined. This is somewhat similar to inheritance, which is inheriting a previously declared model like a normal python class.

```python
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```

In this case, the `Student` model has three fields: **name**, **age**, and **home_group**. And the `CommonInfo` class cannot be used as a normal Django model since it is an abstract base class.
