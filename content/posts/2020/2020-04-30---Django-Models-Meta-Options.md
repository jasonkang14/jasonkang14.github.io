---
title: Django - Models Meta Options
date: "2020-04-30T12:53:37.121Z"
template: "post"
draft: false
slug: "/django/django-models-meta-class"
category: "Django"
tags:
  - "Django"

description: "Django models: what you can do with Meta options"
---

According to the [official document](https://docs.djangoproject.com/en/3.0/topics/db/models/#meta-options), the `Meta` class in a Django model is **anything that is not a field**. One example I provided was `db_table`, which allows you to decide the name of the table instead of using the default `myapp_modelname`.

Below are some examples;

#### 1. abstract

If you set `abstract=True`, this model will be an **abstract base class**, which can be used as a model to inherit.

#### 2. app_label

Can be used when the model is declared outside an app declared in `INSTALLED_APPS` in `settings.py`. However, I don't see when I would use this.

#### 3. db_tablespace

Default is `DEFAULT_TABLESPACE` setting in `settings.py`, which is an empty string, but you can change it according to your needs. I will write a post about talespace later.

#### 4. default_related_name

The default value for this is **<model_name>\_set** as in `membership_set` in the post where through relationship was declared. This option also sets the `related_query_name`.

#### 5. get_latest_by

The name of a field or a list of field names in a model. This is usually `DateField`, `DateTimeField`, or `IntegerField`, which can be ordered numerically. This specifies the default fields to use in `manager`'s `latest()` and `earliest()` methods.

```python
# Latest by ascending order_date.
get_latest_by = "order_date"

# Latest by priority descending, order_date ascending.
get_latest_by = ['-priority', 'order_date']
```

#### 6. managed

The default is `True`, however, if set `False`, django does not perform any table creation or deletion operations in this model. This is useful when this model represents an existing table or a database view that has been created by some other means. I think this would come in handy when you are migrating a project build with PHP to Django.

#### 7. order_with_respect_to

Declaring that you would like to order your jmodel with respect to aa field, ususally a `ForeignKey` like the example below;

```python
from django.db import models

class Question(models.Model):
    text = models.TextField()
    # ...

class Answer(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    # ...

    class Meta:
        order_with_respect_to = 'question'
```

#### 8. ordering

The default ordering for the example. Either a tuple or list of strings and?or query expressions. `ordering = ['pub_date']`, or `ordering = ['-pub_date']` if you ant a descending order. You can also use query expressions like below;

```python
ordering = [F('author').asc(nulls_last=True)]
```

You want to order it by a model called `author` in an ascending order while making null values sort last.
