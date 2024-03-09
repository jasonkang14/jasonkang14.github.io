---
title: Django - Models Meta Options Part II
date: "2020-05-02T18:53:37.121Z"
template: "post"
draft: false
slug: "/django/django-models-meta-class-part-two"
category: "Django"
tags:
  - "Django"

description: "Django models: what you can do with Meta options"
---

#### 9. permissions

Extra permissions to enter into the permissions table when creating a model. Add, change, delete, and view permissions are automatically created for each model. This has to be a tuple like (`permission_code`, `human_readable_permission_name`)

#### 10. proxy

if `proxy=True`, this model is a proxy model. I will write about a proxy model later.

#### 11. constraints

List of constratins you want to define on the model. like an age limit.

```python
from django.db import models

class Customer(models.Model):
    age = models.IntegerField()

    class Meta:
        constraints = [
            models.CheckConstraint(check=models.Q(age__gte=18), name='age_gte_18'),
        ]
```

I will write about constraints in a later post as well.
