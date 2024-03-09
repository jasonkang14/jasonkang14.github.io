---
title: Miniter[05]Making an API Using Django without REST framework I
date: "2019-06-18T22:36:37.121Z"
template: "post"
draft: false
slug: "/posts/Miniter-API-sign-up"
category: "Django"
tags:
  - "Django"
description: "Making an API for the Miniter project with Django without using the REST framework."
---

Even though I finished a tutorial on djangoproject.com, it was still a struggle to even start an assignment/project on back-end because I have never done it before. One of my colleagues who had some experience with back-end development went ahead of me. Looking at his endproduct gave me an idea of what I am suppsoed to do.

#1. Virtual Environment with Miniconda
I used Miniconda to create a virtual environment to do my back-end project. Detailed instruction how to install Miniconda is [here](https://uoa-eresearch.github.io/eresearch-cookbook/recipe/2014/11/20/conda/)<br>

Virtual environment helps to keep dependencies required by different projects especially you have to use different versions of python or django for each project.

```
#create a virtual environment
conda create -n environment_name python=python_version anaconda

#activate a virtual environment
source activate environment_name

#deactivate the virtual environment
source deactivate

#remove the virtual environment if no longer needed
conda remove -n environment_name -all
```

#2. Understanding different ".py" files
I got to understan what each `.py` file does by working on this project.
`manage.py`: a command-line utility. you can run different command lines such as makemigrations, migrate, shell, and runserver (these are pretty much all I had to use until I ran into a problem, which I will explain later in this post)<br>
`settings.py`: settings/configurations for this Django project<br>
`urls.py`: the URL declaration for the project<br>
`models.py`: define each model that this Django project needs. model is like class in python which helps with creating objects<br>
`views.py`: takes a web request and returns a web response<br>

#3. Making objects
You can use either `python manage.py shell` command or create an object direclty from the `admin` page. Objects are created according to how class is defined in `models.py`I tried both ways since I am only working on the back-end side right now.

```
## models.py

def user_list(request):
    user_list = []
    for user in  User.objects.all():
        user_list.append({
            'user_text': user.user_text,
            'name': user.name,
            'date': user.date,
            'content': user.content,
            'password': user.password,
        })
    return JsonResponse(user_list, safe=False)

## views.py

def user_list(request):
    user_list = []
    for user in  User.objects.all():
        user_list.append({
            'user_text': user.user_text,
            'name': user.name,
            'date': user.date,
            'content': user.content,
            'password': user.password,
        })
    return JsonResponse(user_list, safe=False)
```

```
python manage.py shell

from api.models import User  # importing the class created/written in models.py

user = User(user_text="jkang14", name="jason", ..., content="first tweet", password="firstpassword")    # assign attributes as needed

user.save()      # save above information as an object, which gets appended to a list
```

#4. Changing attributes
I ran into a problem when I added a password attribute to the User class. I created an object before adding the attribute, so the table already had some data. When I tried to migrate after changing my model, I kept running into an error that the password attribute needs a default value. even after I set a default value(default='password'), I kept getting the error message.

I Googled--like any other developers would do--and found out that if I delete the migration diretory, I can easily migrate it again. It looked like I had succeeded, but I couldn't create a new object using either method that I described in part 3.

And that was because it was still checked as migrated even though I deleted the file. I had to clear the migration story using zsh in order to migrate again by using the command below

```
$ python manage.py migrate --fake api zero
```

This cleared the history, so I was able to migrate the new model and then create objects.
<br>
<br>
![api at the moment](https://scontent-hkg3-1.xx.fbcdn.net/v/t1.0-9/64883356_10219120716429697_6822158836451770368_n.jpg?_nc_cat=104&_nc_oc=AQm6ELBNSklVU2Qjf_pb8Tv8-4SP-k644FGRMw0rSWbhSmnofAqpiJFmDq93yChdVww&_nc_ht=scontent-hkg3-1.xx&oh=59628082c617465d129b0554cddf7ead&oe=5D855E32)
<br>
This is what I can see in my screen right now. Will work on it more tomorrow and see how it works.
