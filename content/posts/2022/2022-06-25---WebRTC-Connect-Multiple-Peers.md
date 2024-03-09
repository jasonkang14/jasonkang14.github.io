---
title: WebRTC - Connecting Multiple Peers
date: "2022-06-25T14:26:37.121Z"
template: "post"
draft: false
slug: "/webrtc/connecting-multiple-peers"
category: "WebRTC"
tags:
  - "WebRTC"

description: "Multipoint Control Unit(MCU)와 Selective Forwarding Unit(SFU)에 대해 알아본다"
---

# TL;DR

### WebRTC에서 여러명을 연결시키는 방법은, Mesh, MCU, SFU가 있고. 이중에 가장 적합한 것을 사용하면 된다.

처음 WebRTC는 1:1로 등장했다. 1:1은 [RTCPeerConnection](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection)를 [ICE server](https://developer.mozilla.org/en-US/docs/Web/API/RTCIceServer) 정보를 담아 생성하고나서 두 `PeerConnections`가 Signaling Server를 통해 `Session Description Protocol(SDP)`와 `ICE` 정보를 주고받으면 완료된다.

Peer-to-Peer connection이라서 signaling server 에는 큰 부담이 없다. 따라서 서버는 처음 SDP를 주고받고난 후에는 아무런 역할도 하지 않는다. 최근 회사에서 새로운 피쳐를 개발하면서 여러명이 WebRTC로 연결되는 경우에 대해 알아보아야 했고. 1:1 처럼 클라이언트들간의 소통으로 처리하는 것 보다 서버를 거쳐서 통신하는 편이 좋겠다는 결론을 내렸다. 여러명을 WebRTC로 연결시키는 방법은 3가지가 있는데, 각각의 장단점에 대해 알아보도록 한다.

1. Mesh

   - 1:1 연결과 유사하게 모든 `RTCPeerConnection`을 1:1로 연결시키는 것이다.
     ![webrtc-mesh](https://i.imgur.com/z9NOhzf.png)

1:1 연결과 유사하게 signaling server는 처음 SDP교환만 담당하기 때문에, 서버에 부하는 그렇게 크지 않다. 또한 별도의 서버를 거치지 않고(TURN server가 Relay 해주지 않는다면) 직접 연결되기 때문에 실시간성이 보장된다.

하지만 이렇게 연결할 경우, 사용자가 많아졌을 때 클라이언트의 부하가 급격하게 증가한다. 클라이언트는 본인의 영상/음성 stream을 다양한 사용자에게 모두 보내줘야하고, 모두의 stream을 다시 받아와야한다. N명의 사용자가 있다고 할 때, uplink N-1개, downlink N-1개, 1인당 총 2(N-1)개의 링크를 유지해야하는 부담이 있다.

2. MCU(Multipoint Control Unit)
   - 서버가 모든 stream을 받고, 각각의 영상/음성 stream을 하나로 mix해서 클라이언트에게 보내주는 방식
     ![webrtc-mcu](https://i.imgur.com/L4OaCg4.png)

서버가 모든 stream을 받은 후, 처리해서 모든 client들에게 뿌려주기 때문에, 각각의 `RTCPeerConnection`이 1:1로 연결되는 Mesh와 비교했을 때, 각 클라이언트가 1개의 uplink와 1개의 downlink만 유지한다. 관리해야 할 stream의 갯수가 고정된다는 장점이 있다. 따라서 client에 부하가 걸리지 않는다는 장점이 있다.

하지만 Mesh의 케이스와 반대로, 이런 경우에는 서버가 stream들을 mixing과 transcoding하는 과정을 거치기 때문에, 서버에 오버헤드가 걸릴 수 있는 단점이 있다. 또한 같은 이유로 서버에서 가공하는 시간이 필요하기 때문에 Mesh에 비해 실시간성이 떨어지는 단점이 있다.

3. SFU(Selective Forwarding Unit)

- 서버가 모든 stream을 받고, remote peers 들의 steam을 사용자들에게 전달한다.
  ![webrtc-sfu](https://i.imgur.com/EkQnjdD.png)

서버에 uplink stream을 보낸다는 측면에서는 MCU와 동일하다. 하지만 차이가 있다면 MCU가 모든 stream을 하나로 합치는 과정을 거친다면, SFU는 stream을 forwarding 하기만 한다. 따라서 서버에서 별도의 데이터 가공을 하지 않기 때문에, 서버에 오버헤드가 감소하고 MCU와 비교했을 때 실시간성이 유지된다.

하지만 클라이언트 입장에서는 downlink stream의 숫자가 늘어나기 때문에, 클라이언트에 조금 부하가 생길 수 있다.

결국 뭐가 맞고 틀린 문제가 아니라 서비스 환경에 따라 적절히 잘 선택해야한다. 다자간 영상연결이라면 각각의 stream을 따로 처리하는 편이 좋을테니 SFU방식으로 처리해야 할 것 같고. 실제로 Google Meet이나 ZOOM 모두 SFU 방식을 사용한다고 한다. 하지만 굳이 모든 영상 stream을 핸들링 할 필요 없는 인터넷 강의와 같은 경우에는 MCU로 처리해도 무방할 것 같다. 화면은 강사의 화면 하나만 있고 학생들은 음성으로 질문만 하면 되기 때문.

참고자료

1. https://webrtc.ventures/2020/12/webrtc-media-servers-sfus-vs-mcus/
2. https://bloggeek.me/webrtc-multiparty-video-alternatives/
