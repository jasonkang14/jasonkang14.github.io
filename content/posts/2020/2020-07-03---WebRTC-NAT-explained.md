---
title: WebRTC - NAT explained
date: "2020-07-03T10:53:37.121Z"
template: "post"
draft: false
slug: "/webrtc/nat-explained"
category: "webrtc"
tags:
  - "WebRTC"

description: "Network Address Translation(NAT) explained"
---

## TL;DR

Network Address Transaltion **translates** your private network information into public network information

## Explanation

Network Address Translation(NAT) is embedded into networking routing devices like firewalls and routers. It is placed between a private netowrk(Local Area Network-LAN) and the public network(the Internet). Thus translating your local information to become usable via the public network.

Incoming traffic into private ntwork is routed through a public address, which then gets bound on the NAT device. Such binding is created when an internal machine with a private IP address tries to access a public IP address.

#### WebRTC needs to pass media between two peers that might both be behind NAT devices. External packets also need to be passed to the internal network in order for peers to commnunicate with one another

Therfore, NAT traversal is requied. The word traverse means to travel across or through. Therefore, WebRTC must travel through NAT devices to reach the intneral network.

There are two ways to get **NAT Traversal** done. One is STUN - Session Traversal Utilities for NAT. And the other is TURN - Traversal Using Relays around NAT. STUN helps a peer find out his or her public IP address, and TURN--like its name stands--relays information from a peer to another. TURN is more reliable but more expensive.
