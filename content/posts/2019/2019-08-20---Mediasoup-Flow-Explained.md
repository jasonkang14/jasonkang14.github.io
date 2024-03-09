---
title: WebRTC[03] - Mediasoup Flow Explained
date: "2019-08-20T22:27:37.121Z"
template: "post"
draft: false
slug: "/posts/webrtc/webrtc-Mediasoup-flow-explained"
category: "webrtc"
tags:
  - "WebRTC"

description: "Mediasoup Flow explained"
---

###Mediasoup is an SFU(Selective Forwarding Unit) which receives audio and video streams from endpoints and relays them to everyone else

1. Device Loading

   - retrieve the `rtpCapabilities` of a router from the server and uses the information to load device

2. Create WebRTCTransport

   - create a `sendTransport` and a `localTransport`
   - `transports` must be created both in the client-side and the server-side

3. Connect WebRTCTransport

   - this is done by calling `sendTransport.produce()` or `recvTransport.consume()` in the client side.
   - you have to connect the transport created in the server-side with the `dtlsParameter` of the device used to make a WebRTC connection.

4. Produce

   - When you call `sendTransport.produce()` in the client-side, `sendTransport.on("connect")` event is fired.
   - If the connection is successful, `sendTransport.on("produce")` event is fired, which creates a producer in the server-side.

5. Consume

   - When there is a new `producer`, participants must `consume` the other's `producer`.
   - Create a `consumer` in the server-side first, and then use `socket.io` to let the client-side know that there is a new `consumer` created in the server-side
   - The client-side then creates a `consumer` using the information from the server-side

6. General
   - When you build a WebRTC connection using `mediasoup`, `transports`, `producer`, and `consumer` must be created both in the client-side and the server-side. And the client-side and the server-side must create the three components using the same information.
