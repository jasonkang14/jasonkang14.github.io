---
title: Django - Select Related
date: "2020-06-04T15:53:37.121Z"
template: "post"
draft: false
slug: "/django/select-related"
category: "Django"
tags:
  - "Django"

description: "Let's try to improve efficiency when you make a query in Django"
---

### TL;DR

`select_related()` allows you to access data without accesing the database multiple times.

### explanation

`select_related()` is helpful when there is a `ForeignKey` relationship. Esepecially if there are nested foreignkey relationships between tables, it will save you a lot of time.

I created models in `models.py` for a testing purpose like this

```python
class State(models.Model):
    name = models.CharField(max_length=25)

class City(models.Model):
    state = models.ForeignKey(State, on_delete=models.CASCADE)
    name  = models.CharField(max_length=25)

class Street(models.Model):
    city = models.ForeignKey(City, on_delete=models.CASCADE)
    name = models.CharField(max_length=25)
```

And then created two views. One using `select_related()` and the other not using it

```python
class SelectRelatedView(View):
    def get(self, request):
        print("select_related")
        street = Street.objects.select_related('city__state').get(pk=1)
        print(F"street      == {street} {datetime.datetime.now()}")
        c = street.city
        print(f"street.city == {street.city} {datetime.datetime.now()}")
        s = c.state
        print(f"city.state  == {c.state} {datetime.datetime.now()}")

        return HttpResponse(status=200)

class NormalCityView(View):
    def get(self, request):
        print("normal")
        street = Street.objects.get(pk=1)
        print(F"street      == {street} {datetime.datetime.now()}")
        c = street.city
        print(f"street.city == {street.city} {datetime.datetime.now()}")
        s = c.state
        print(f"city.state  == {c.state} {datetime.datetime.now()}")

        return HttpResponse(status=200)
```

And here is the time difference

```
select_related
street      == Street object (1) 2020-06-04 15:25:55.039457
street.city == City object (1) 2020-06-04 15:25:55.039495
city.state  == State object (1) 2020-06-04 15:25:55.039506
[04/Jun/2020 15:25:55] "GET /select_related/ HTTP/1.1" 200 0

normal
street      == Street object (1) 2020-06-04 15:26:04.422837
street.city == City object (1) 2020-06-04 15:26:04.425180
city.state  == State object (1) 2020-06-04 15:26:04.429000
```

You can see that when I used `select_related()`, the time difference between the last print line the the first prine like is 0.00005 second. However, when I did not use `select_related()`, the time difference is 0.006 second. Therefore, if I were to use simple math, `select_related()` is **120** times more efficient.
