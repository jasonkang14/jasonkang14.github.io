---
title: Terminology from Interviews
date: "2019-08-09T23:27:37.121Z"
template: "post"
draft: false
slug: "/posts/Terminology-from-Interviews"
category: "terminology"
tags:
  - "Interview"

description: "Some important terminology in web development based on interview questions"
---

I have been doing a lot of interviews these past couple weeks--maybe 10--as I have been searching for a job. I have been told that terminology--knowing exact words to use in different situations--is important at [WeCode](https://wecode.co.kr/), but didn't fully understand it until I went through interviews. Below are some examples written in no speicic order, just based off what I recall.

1. populate

- to add record to a database. It can be data for testing an application's functionality or the actual data during implementation.

2. sanitize

- to remove malicious data from user input, such as form submissions
- to clean user input to avoid code-conflicts (duplicate ids for instance), security issues (xss codes etc), or other issues that might arise from non-standardized input & human error/deviance.

3. stale

- old data that is no longer fresh. something that has not been updated to represent current information

4. pure

- of which return value is only determined by its input values, without observable side effects.

5. side effect

- the modification of some kind of state outside its local environment something like a mutable data structure or variable, using IO, throwing an exception or halts an erro
- does not have to be hidden or unexpected, and I was told that sometime side effects are used on purpose in order to implement certain things of which examples that I do not recall

6. declarative

- a programming paradigm in which the programmer defines what needs to be accomplished by the program without defining how it needs to be implemented.

7. imperative

- opposite of declarative. you have to define how it needs to be implemented.

8. memoize

- caching information to return the output when a program receives an input that it has experienced

9. stateless

- no record of previous interactions and each interaction request has to be handled based entirely on information that comes with it.
- opposite of stateful, which keeps the records of previous interactions

10. RESTful: Definition from [RESTFULAPI.net](https://restfulapi.net/)

- Representational State Transfer
- Characteristics
  - stateless: as described above, each request from client to server must contain all the necessary information to understand the request as no information from previous requests is stored
  - cacheable: data within a response to a request must be labeled as cacheable or non-cacheable. If cacheable, a client cache is given the right to reuse that response data for later
  - Uniform interface: four interface constraints:
    - identification of resources
    - manipulation of resources through representations
    - self-descriptive messages
    - hypermedia as the engine of application state
  - to be honest, I have no idea what the fourth constraint means

11. compile

- the process of creating an executable program from code written in a compiled programming language
- allows a computer to run and understand a program without the need of the programming software used to create the program
