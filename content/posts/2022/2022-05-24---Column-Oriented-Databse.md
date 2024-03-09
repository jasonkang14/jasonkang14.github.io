---
title: Column Oriented Database
date: "2022-05-24T03:34:37.121Z"
template: "post"
draft: false
slug: "/sql/column-oriented-database"
category: "Database"
tags:
  - "Database"

description: "How a column-oriented database is different from a row-oriented database"
---

# TL;DR

### A column-oriented database is faster at reading data than a row-oriented database

When I think about reading some data from my database, I usually think about how to access a row. However, I have come across the concept of a column-oriented database, which means that a table is organized by its columns rather than rows. Interesting. and I’ve never heard of it, so I decided to dig into this a little bit.

Databases like PostgreSQL, MySQL, and MariaDB are row-oriented databases, which is probably why I usually think about accessing rows in a table. Databases like BigQuery are known to be column-oriented. I have been told that BigQuery is fast, so I guess column-oriented databases are fast.

Let’s look into a row-oriented database first. and let’s think about a table that looks like this

| Name  |   City   | Age |
| :---: | :------: | :-: |
| John  | New York | 33  |
| James |  Seoul   | 27  |
| David |  London  | 27  |

The data above is stored in a disk like below
![Row data stored in a disk](https://i.imgur.com/UKrKiWq.png)

Let’s say you would like to insert some data into the table like this. It won’t be that big of a problem because it would look like this.

| Name  |   City   | Age |
| :---: | :------: | :-: |
| John  | New York | 33  |
| James |  Seoul   | 27  |
| David |  London  | 27  |
| Paul  | Chicago  | 22  |

And if you were to add that row into a disk, you just gotta put it at the end of your original data.

![A row added into a row-oriented database](https://i.imgur.com/IyM39lI.png)

Reading from a row-oriented database isn’t too bad. but it actually allocates more memory when you try to aggregate some data. Let’s say you want to get the average age of all the people from the table. Then you would have to access the entire row. And it gets worse if your disk is too small so that a disk can hold only one row. That means that you have to read from every single disk to calculate the age

![a row-oriented database split into several disks](https://i.imgur.com/oR7An7Z.png)

---

Now let’s talk about a column-oriented database. The very first table you saw would look like this in a disk.

![Row data stored in a disk](https://i.imgur.com/UKrKiWq.png)

This means that adding data to the table could be inefficient because it would look like this.

![data added into a disk from a column-oriented database](https://i.imgur.com/SZsfZDi.png)

But it’s actually not a problem if your disk holds only one column. Instead of accessing every single disk that you did when the row-oriented case, you just have to access only one disk like this.

![a column-oriented database split into several disks](https://i.imgur.com/eiZTuze.png)

And in this case, if you were to add some data, you just have to append new data at the end of each disk.

Reading some specific person’s data from a column-oriented database is also. fast because the columns are indexed.

![Data added to a column-oriented database](https://i.imgur.com/UdkJ6Zp.png)
