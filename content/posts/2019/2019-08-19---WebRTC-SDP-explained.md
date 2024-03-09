---
title: WebRTC[02] - SDP
date: "2019-08-19T22:27:37.121Z"
template: "post"
draft: false
slug: "/posts/webrtc/webrtc-SDP-explained"
category: "webrtc"
tags:
  - "WebRTC"

description: "SDP explained"
---

##SDP stands for Session Description Protocol.

SDP is information created via `createOffer()` and/or `createAnswer()`. SDP is used to provide a callee with information about caller and vice versa.

1. describes multimedia sessions, which can contain audio, video, whiteboard, fax, modem, and other streams.
2. Provides a general purpose, standard representation to describe various aspects of multimedia session such as media capabilities, transport addresses and related matadata in a transport manner
3. Used for session announcement, session invitation and parameter negotiation
4. Used in the context of Session Initiation Protocol, Real-time Transport Protocol, and Real-time Streaming Protocol applications.

To be honest, all you need to know about `SDP` is that this is information that a caller and a callee must exchange in order to let each other know who he or she is.
