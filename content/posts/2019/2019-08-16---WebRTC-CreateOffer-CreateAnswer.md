---
title: WebRTC Flow[01] - createOffer() and createAnswer()
date: "2019-08-16T17:27:37.121Z"
template: "post"
draft: false
slug: "/posts/webrtc/webrtc-create-offer-create-answer"
category: "webrtc"
tags:
  - "WebRTC"

description: "WebRTC flow explained"
---

##This post is about using WebRTC in React Native.

make sure to install `react-native-webrtc` versions 1.75 or above <br>
`npm install --save react-native-webrtc`

And then you have to import. <br>
`import { RTCPeerConnection } from 'react-native-webrtc'`

Now create a caller

`const caller = new RTCPeerConnection()`

Now the caller has to `createOffer()` with `offerOptions`.
`offerOptions` represent which source of data you are going to offer: either audio or video, or could be both.

```typescript
const offerOptions = {
  offerToReceiveAudio: 1,
  offerToReceiveVideo: 0,
  voiceActivityDetection: true,
};
```

Using the created `offer`, the caller must set his/her `localDescription`

```typescript
caller.createOffer(offerOptions)
  .then(async (desc) => {
    await caller.setLocalDescription(desc);
```

Then the caller sends this `localDescription` to a callee, which is another `RTCPeerConnection`. <br>
`const callee = new RTCPeerConnection()`<br>

Then the callee uses the `localDescription` from the caller to set the callee's `remoteDescription`. After setting the `remoteDescription`, the callee now `createAnswer()` and uses the answer to set the callee's `localDescription`.

The callee then sends the `localDescription` to the caller so that the caller can use the callee's `localDescription` to set the caller's `remoteDescription`

```typescript
callee.setRemoteDescription(desc)
  .then(() => {
    return callee.createAnswer()
      .then(async (desc) => {
        await callee.setLocalDescription(desc)
```

The caller receives the callee's `localDescription` and uses the `localDescription` to set the caller's `remoteDescription`

`caller.setRemoteDescription(desc)`

Then the caller and callee needs to change their Interactive Connectivity Establishment(ICE) information in order to build connection. It is managed using an eventhandler, which I will talk about in a later post.
