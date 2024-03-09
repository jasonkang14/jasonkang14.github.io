---
title: List vs Tuple
date: "2022-06-01T13:34:37.121Z"
template: "post"
draft: false
slug: "/cs/list-vs-tuple"
category: "CS"
tags:
  - "CS"

description: "Data Structure - list vs tuple"
---

# TL;DR

### The major difference between the two is that tuples are immutable while lists are mutable. 

I started to watch MITOpenCourseWork in order to study some basics of computer science. Today, I got to look into the difference between list and tuple.

Lists and tuples are similar in ways that they are both

1. collections of data
2. heterogeneous — you can store whichever data you want
3. iterable
4. accessible using index

Let's take a look using some code

---

```python
test_list = ["apple", "banana", "orange", "kiwi", "watermelon"]
test_tuple = ("apple", "banana", "orange", "kiwi", "watermelon")

test_list[3] = "mango"
print(test_list)

["apple", "banana", "orange", "mango", "watermelon"]
test_tuple[3] = 7

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

You can see that the third index of the list has been changed to 7 but it failed to change an item of the tuple. This is because tuples are immutable.

I have also learned that **Python allocates more memory to lists** than tuples because **the size of lists can be manipulated.** Let’s go back to the tuple and the list.

```python
import sys
print(sys.getsizeof(test_list))   # 120
print(sys.getsizeof(test_tuple))  # 80
```

You can also see that tuples are more memory-efficient than lists. and because of that, iterations over tuples are a lot faster than over lists if they are of the same size.

```python
python -mtimeit '["apple", "banana", "orange", "kiwi", "watermelon"]'
>> 10000000 loops, best of 3: 0.096 usec per loop

python -mtimeit '("apple", "banana", "orange", "kiwi", "watermelon")'
>> 100000000 loops, best of 3: 0.0115 usec per loop
```

It may not be the best comparison, but you can see that iteration over the tuple is approximately **8 times faster.**

---

## Conclusion

1. Lists and tuples have similarities but their nature makes tuples more efficient in terms of memory.
2. However, it looks like it’s more of a personal choice in terms of which one to use
