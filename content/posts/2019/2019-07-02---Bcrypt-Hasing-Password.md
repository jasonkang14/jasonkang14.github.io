---
title: Bcrypt[hashpw()] Signup, how to hash password securely
date: "2019-07-02T21:27:37.121Z"
template: "post"
draft: false
slug: "/posts/Bcrypt-Signup-Function-using-hashpw"
category: "Brcypt"
tags:
  - "Django"

description: "Implementing Signup using Django and Bcrypt"
---

I've been using Django to make API for a project, which was very straight forward as long as I follow the tutorial from the official website. And signup was also fairly easy.

First, install bcrypt:
`pip install bcrypt`

And using bcrypt, you gotta encode the entered password into `bytes` type,and then hash the password using a salt.

```python
else :
    password = bytes(new_account_info["password"], "utf-8")
    hashed = bcrypt.hashpw(password, bcrypt.gensalt())

    new_account = Account(
        user_id = new_account_info["user_id"],
        password = hashed.decode("utf-8"),
    )

    new_account.save()
```

You can use a randomly generated salt using `bcrypt.gensalt()`, which gets added to the password entered by the user. Then it gets hashed by using `bcrypt.hashpw()`, which gets decoded again before getting saved to the database.

I saved the decoded password to the databse, which doesn't really seem to make sense becuase the database probably should have encoded password, which sounds more secure. However, if I save encoded password to the database, it causes problem when the server checks the password upon login,which will be discussed in the next post.
