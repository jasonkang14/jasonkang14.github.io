---
title: WebRTC[05] Concluding ICE via Nomination
date: "2019-09-10T16:27:37.121Z"
template: "post"
draft: false
slug: "/posts/webrtc/concluding-ice-via-nomination"
category: "webrtc"
tags:
  - "WebRTC"

description: "Concluding ICE via nomination"
---

After making a candidate pair, the two peers must decide whether to continue or conclude ICE.

This is done through a process called nomination, which is literally nominating a particular candidate pair as the nominee for a WebRTC communication.

There are two different types of nomination: regular and aggressive.

But before talking about nomination, you must know that ICE assigns one of the peers in the role of the controlling agent and the other a controlled agent.

In regular nomination, the controlling agent lets the connectivity checks continue until one valid candidate pair for each media stream is found. If a valid candidate pair is selected, the controlling agent sends a STUN request with a flag that it will use this pair as the valid candidate pair. If the STUN request succeeds, the selected pair is nominated.

In aggressive nomination, the controlling agent does the connectivity checks with the flag that this pair will be selected. If such connectivity checks succeeds, the pair is nominated.

Aggressive nomination tends to be faster than regular nomination, but it gives less flexibility as it selects the pair which passes the connectivity checks with the flag as the nominated pair even if one of the other pairs could be a better option.
