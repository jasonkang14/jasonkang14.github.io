---
title: WebRTC[04] ICE Candidate Exchange
date: "2019-09-08T19:27:37.121Z"
template: "post"
draft: false
slug: "/posts/webrtc/ice-candidate-exchange"
category: "webrtc"
tags:
  - "WebRTC"

description: "How ICE candidate exchange works in WebRTC flow"
---

Simply put, ICE candidate exchange is simply a negotiation between to peers to see which ICE candidates--or an ICE candidate pair--to use in order to establish a WebRTC connection.

An ICE candidate is an object that looks like this

```typescript
{
  candidate: "candidate:2322976989 1 tcp 1518280447 IPADDRESS PORT typ host tcptype passive generation 0 ufrag UXEY network-id 1 network-cost 10";
  sdpMLineIndex: 0;
  sdpMid: "audio";
}
```

Before negotiating ICE candidates, a peer must gather candidate addresses where candidate is a transport address, which is a combination of IP address and port for a particular transport protocol.

There are three types of candidates: host, server-reflexive, and relayed. There is also a peer-reflexive candidate, but I will talk about it in a later post.

Host candidates are obtained from physical or logical netowrk interfaces such as public internet or a private network. Host candidates can be easily understood as the one for which its IP address is the actual, direct IP address of a remote peer. like something you would retrieve from a local computer.

Server-reflexive candidates are obtained from either a STUN or TURN server. The IP of a server-reflexive candidate indicates an intermediary address to represent the candidate's peer anonymously.

Relayed candidates only from a TURN server. Their IP addresses are addresses the TURN server uses to forward the media between two peers.

ICE candidate exchange requires connectivity checks through which the two peers decide wich ICE candidate to use to communicate with each other. Each peer creates a list of candidates based on their priorities and share the lists and performs connectivity checks as they go down the list.

The steps could be summarized like below;

1.  Sort the candidate pairs in priority order.

2.  Send checks on each candidate pair in priority order.

3.  Acknowledge checks received from the other agent.

When the checks are complete, the two candidates form a candidate pair through wich the two peers communicate by sending media through the pair.
