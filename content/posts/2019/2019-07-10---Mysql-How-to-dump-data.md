---
title: How to import local mysql databases to AWS mysql databases
date: "2019-07-10T15:27:37.121Z"
template: "post"
draft: false
slug: "/posts/How-to-import-local-mysql-databases-to-AWS-mysql databases"
category: "Database"
tags:
  - "Database"

description: "How to import local mysql databases to AWS mysql databases"
---

As the first project is coming to close, I am trying to deploy what I have developed in local. In order to do that, I had to backup my local database in to a temporary sql file and then dump that data into AWS database.

Make sure to remove quotation marks "" when you enter commands

First make a backup `sql` file.
`mysql -u root -p "database you want to backup" > "temporary.sql"`

If you don't put `-u root -p`, it tries to login to the mysql server using your laptop name. By adding `-u root -p`, you get to use the root and its password.

And then go into your AWS mysql and create a database
`mysql -h "your aws mysql url" -u root -p`

when you are in AWS mysql server,
`CREATE DATABASE "yourdatabasename" CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci`

and then dump the backup into your AWS database
`mysql -h "your aws mysql url" -u root -p "yourdatabasename in AWS mysql" < "temporary.sql"`

Then you are done
