---
title: How does the backend work?
date: "2022-01-23T11:34:37.121Z"
template: "post"
draft: false
slug: "/server/how-the-backend-works"
category: "SERVER"
tags:
  - "Server"

description: "How the backend works - server, application, and database"
---

Backend receives requests from a client, and then it generates and sends responses back to the client. The backend has three different parts. server, application, and database.

The server is the computer that receives requests. And this computer runs an application that contains logic about how to respond to different requests that a client sends. The database is where all the information is stored. The application accesses this database in order to generate appropriate responses for the client.

A client and a server communicate with each other using an API(Application Programming Interface). It defines how a client should send a request to a server and how the server must respond to the request. API contains things like an endpoint and data formats that the server needs from a client.

Let's say you are writing letters to your friends to ask for something. Your friend John talked about a cupcake that he tried in Chicago and a cookie in New York. And you want some information about that cupcake. Then you would write to John's address and ask him where exactly in Chicago where he got that cupcake. If you put in the wrong address or ask him about the cookie from New York, you are not gonna get the information you want.

Here, John's address is called the endpoint, the address to which a client needs to send a request. and that cupcake from Chicago is the data format. The client must use the correct endpoint along with the correct data format to receive the information he or she wants. Through the API, the application knows what kinds of information a client wants and returns the requested information.

Then where is this server, which is the computer that receives requests and passes them onto the application? When you think about a computer, probably an electronic device comes to your mind. But today, servers are usually run on clouds. Cloud is a virtual device that allows you to retrieve resources through the Internet. It is more accessible, convenient, and reliable than physical devices.

Check out this YouTube video below if you need some visuals to understand this concept.

<iframe width="800" height="315" src="https://www.youtube.com/embed/gOghS3BmaxI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
