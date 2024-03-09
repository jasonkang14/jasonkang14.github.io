---
title: What is WebRTC and How Does It Work?
date: "2019-08-03T14:27:37.121Z"
template: "post"
draft: false
slug: "/posts/webrtc-explained"
category: "webrtc"
tags:
  - "WebRTC"
description: "Basics of WebRTC Explained"
---

###This is a summary of a post at [innoarchitech](https://www.innoarchitech.com/blog/what-is-webrtc-and-how-does-it-work)

#### WebRTC stands for Web Real-Time Communications

###Intro

1. A set of plugin-free APIs
2. Leverages multiple standards and protocols such as data streams, STUN/TURN servers, signaling, JSEP, ICE, SIP, SDP, NAT, UDP/TCP, network sockets, and more

###Detail

####Peer-To-Peer Communication

- audio and video communications
- each person's web browser must agree to begin communication, know how to locate one anohter, bypass security and firewall protections, and transmit all multimedia communications in real-time

- knowing how to locate and establish a network socket connection with another computer's web browser is crucial
  - you need to bidirectionally transmit multimedia data
  - difficult to make a request to another computer because it is hard to know exactly where it is.
    - you have to send a request by sending your audio/video data while receiving it from the othe rside without going through an external server

####Firewalls and NAT Traversal

- Firewall enforces a set of rules about what data packets will be allowed to enter of leave a network. some sort of a security system

- computer sits behind a firewall and Network Access Translation device (NAT)

  - which is why your computer does not have a static public IP address
  - NAT device translates private IP address from inside a firewall to public-facing IP addresses
    - NAT is needed for security
    - NAT makes your request IP to look different from your actualy IP address

- STUN(Session Traversal Utilities for NAT) and TURN(Traversal Using Relays around NAT) servers allow you to get someone else's IP to make a call by sending audio/video data
  - a request for your public-facing IP address is first made to a STUN server
    - you tell your peers to send a request to your server via your public-facing IP address

####Signaling, Sessions, and Protocols

- negotiate and establish the network session connection with your peer

  - The initial session negotiation and establishment happens using a signaling/communication protocol specialized in multimedia communications
    - something like Session Initiation Protocol(SIP)
      - the chosen signaling protocol must work with an application layer protocol called the Session Description Protocol (SDP)

- when you try to communicate with someone, you generate a set of Interactive Connectivity Establishment(ICE) protocol candidates

  - the candidates represent a given combination of IP address, port, and transport protocol to be used

  ![WebRTC exchange diagram from MDN](https://mdn.mozillademos.org/files/6119/webrtc-complete-diagram.png)

####Complete Process Summarized

- Each peer first establishes its public-facing IP address by sending a request to STUN

  - Signaling data channels are then dynamically created to detect peers
    - this channel is somewhat like a private room
    - only those who know about the room can send and receive messages
    - you need a unique idenfitier to access it
    - but some protocols do not require a channel since webRTC is flexibile and does not specify the signaling process

- support peer-to-peer negotiations and session establishment

  - once two or more peers are connected to the same channel, the peers are able to communicate and negotiate session information
    - initiating peer sends an offer using a signaling protocol and waits to receive an answer from any receivers that are connected to the given channel
    - Once the answer is received, a process occurs to determine and negotiate the best of the ICE candidates gathered by each peer.

- once the optimal ICE candidates are chosen,

  - things like all of the required metadata, network routing, and media information is agreed
  - the network socket session between the peers is then fully established and active. - local data streams and data channel endpoints are created by each peer, and multimedia data is finally transmitted both ways

- if the process of agreeing on the best ICE candidates fails, the fallback is to use a TURN server as a relay instead of using STUN
  - basically employs a server that acts as an intermediary and relays any stransmitted data between peers.
  - when TURN is used, each peer no longer needs to know how to contact and transmit data to each other.
    - they need to know what public TURN server to send and receive real-time multimedia data during a communication session
  - TURN servers need to be quite robust, have extensive bandwidth and processing capabilities, and handle potentially large amounts of data.

####WebRTC JavaScript APIs

- WebRTC and the processes described are implemented through a set of JavaScript APIs that actually produce and transmit the multimedia data being used for real-time communications.
- The primary WebRTC APIs include, Navigator.getUserMedia (capture audio and video), RTCPeerConnection (create and negotiate peer-to-peer connections), and RTCDataChannel (represents a bidirectional data channel between peers).
