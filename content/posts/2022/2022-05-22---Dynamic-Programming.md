---
title: Dynamic Programming
date: "2022-05-22T23:34:37.121Z"
template: "post"
draft: false
slug: "/cs/dynamic-programming"
category: "CS"
tags:
  - "CS"

description: "See how dynamic programming can improve your performance"
---

When I first started programming, a senior engineer asked me how I would solve a certain problem. I do not recall the exact question, but he said the answer is dynamic programming.

From then I knew dynamic programming is a powerful tool to use in programming. And now I finally got to see what it means as I watched Introduction to Computer Science Lectures on MIT Open Courseware

The word <b>dynamic</b> means **characterized by constant change**.

So dynamic programming is a way to store - or memoize - results of subproblems that change constantly in order to prevent the computer from re-computing those values.

Let's take a look at a Fibonacci algorithm as an example.

```python
def fib(n):
    global num_calls
    num_calls += 1

    if n <= 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)
```

now let me add some `print` lines to the code.

```python
def fib(n):
    global num_calls
    num_calls += 1
    print(F"fib called with {n} number of calls == {num_calls}")

    if n <= 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)

num_calls = 0
n = 9
ans = fib(n)

print(F"fib of {n} == {ans} with num_calls == {num_calls}")
```

If you run the code above, below is the result

```
fib called with 9 number of calls == 1
fib called with 8 number of calls == 2
fib called with 7 number of calls == 3
fib called with 6 number of calls == 4
fib called with 5 number of calls == 5
fib called with 4 number of calls == 6
fib called with 3 number of calls == 7
fib called with 2 number of calls == 8
fib called with 1 number of calls == 9
fib called with 2 number of calls == 10
fib called with 3 number of calls == 11
fib called with 2 number of calls == 12
fib called with 1 number of calls == 13
fib called with 4 number of calls == 14
fib called with 3 number of calls == 15
fib called with 2 number of calls == 16
fib called with 1 number of calls == 17
fib called with 2 number of calls == 18
fib called with 5 number of calls == 19
fib called with 4 number of calls == 20
fib called with 3 number of calls == 21
fib called with 2 number of calls == 22
fib called with 1 number of calls == 23
fib called with 2 number of calls == 24
fib called with 3 number of calls == 25
fib called with 2 number of calls == 26
fib called with 1 number of calls == 27
fib called with 6 number of calls == 28
fib called with 5 number of calls == 29
fib called with 4 number of calls == 30
fib called with 3 number of calls == 31
fib called with 2 number of calls == 32
fib called with 1 number of calls == 33
fib called with 2 number of calls == 34
fib called with 3 number of calls == 35
fib called with 2 number of calls == 36
fib called with 1 number of calls == 37
fib called with 4 number of calls == 38
fib called with 3 number of calls == 39
fib called with 2 number of calls == 40
fib called with 1 number of calls == 41
fib called with 2 number of calls == 42
fib called with 7 number of calls == 43
fib called with 6 number of calls == 44
fib called with 5 number of calls == 45
fib called with 4 number of calls == 46
fib called with 3 number of calls == 47
fib called with 2 number of calls == 48
fib called with 1 number of calls == 49
fib called with 2 number of calls == 50
fib called with 3 number of calls == 51
fib called with 2 number of calls == 52
fib called with 1 number of calls == 53
fib called with 4 number of calls == 54
fib called with 3 number of calls == 55
fib called with 2 number of calls == 56
fib called with 1 number of calls == 57
fib called with 2 number of calls == 58
fib called with 5 number of calls == 59
fib called with 4 number of calls == 60
fib called with 3 number of calls == 61
fib called with 2 number of calls == 62
fib called with 1 number of calls == 63
fib called with 2 number of calls == 64
fib called with 3 number of calls == 65
fib called with 2 number of calls == 66
fib called with 1 number of calls == 67
fib of 9 == 34 with num_calls == 67
```

You can see that the fib function was called **67** times

But if you memoize the value, you can save a lot of time. Let’s look at the updated code below.

```python
def fast_fib(n, memo):
    global num_calls
    num_calls += 1
    print(F"fib called with {n} number of calls == {num_calls}")

    if not n in memo:
        memo[n] = fast_fib(n-1, memo) + fast_fib(n-2, memo)

    return memo[n]

def fib(n):
    memo = {
      0 : 0,
      1 : 1,
    }

    return fast_fib(n, memo)

num_calls = 0
n = 9
ans = fib(n)

print(F"fib of {n} == {ans} with num_calls == {num_calls}")
```

If you run the code with a memoized value, the result is like below.

```
fib called with 9 number of calls == 1
fib called with 8 number of calls == 2
fib called with 7 number of calls == 3
fib called with 6 number of calls == 4
fib called with 5 number of calls == 5
fib called with 4 number of calls == 6
fib called with 3 number of calls == 7
fib called with 2 number of calls == 8
fib called with 1 number of calls == 9
fib called with 0 number of calls == 10
fib called with 1 number of calls == 11
fib called with 2 number of calls == 12
fib called with 3 number of calls == 13
fib called with 4 number of calls == 14
fib called with 5 number of calls == 15
fib called with 6 number of calls == 16
fib called with 7 number of calls == 17
fib of 9 == 34 with num_calls == 17
```

You get the same result, but the fib — or the fast_fib — function was called **17** times, which is **about a third of the number of the times called compared to the original function.** This is because dynamic programming saves the previously calculated number in the memo.

Another important thing for you to notice is that the argument to the fib function remains the same. This is the idea of `Abstraction`. A user does not have to know exactly what is going on behind the scene, but the function returns the expected value in the most efficient way. I would like to talk about this in a later post.

If the size of data gets bigger, dynamic programming would play a bigger role than it did in the simple example that I provided.

If you want to learn more in detail, please watch this video on YouTube presented by MIT Open Courseware. Hope this helps.
