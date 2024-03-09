---
title: STUN and NAT
date: "2022-04-22T18:53:37.121Z"
template: "post"
draft: false
slug: "/webrtc/stun-and-nat"
category: "WebRTC"
tags:
  - "WebRTC"

description: "STUN 서버가 실패하는 이유는 무엇인가에 대해 알아본다"
---

# TL;DR

### NAT의 종류에 따라 STUN 서버가 제대로 작동하지 않을 수 있다.

WebRTC 처음 공부할 때 내렸던 결론은 STUN은 가끔 안 될 때가 있고, TURN은 왠만하면 된다 였다. 회사에 인턴 분들이 오시면서 WebRTC과제를 제공하는데, ICE server 설명 자료를 준비하다보니 왜 STUN이 안되는지 궁금해져서 찾아봤다.

[STUN](https://www.ietf.org/rfc/rfc3489.txt)의 기본적인 원리는, 본인의 Public IP를 모르는 client에게 Public IP 주소를 알려줌으로써, 이 Public IP를 통해 [RTCPeerConnection](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection) 들이 소통할 수 있게 하는 것이다.

따라서, STUN 서버로 통신할 수 없다는 것은 STUN server가 알려주는 Public IP의 주소가 제대로 작동하지 않는다는 것이다. 즉 클라이언트의 Private IP와 매핑된 Public IP로 통신이 불가능하다는 것이다.

이걸 이해하기 위해서 NAT의 종류에 대해 찾아봤는데, 총 4가지가 있다고 한다. 순서대로 알아본다.

1. Full Cone NAT

   - 같은 private ip와 port를 가진 host는 같은 public ip와 port에 매핑된다
   - 따라서 외부에서 ip와 port에 상관없이 매핑된 public ip와 port를 통해 host에 요청을 보낼 수 있다.

```bash
   {NAT internal side}   |    {NAT external side}  |  {Remote host}
                         |                         |
1. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, INT_PORT) -> (REM_ADDR, REM_PORT) ]
2. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, INT_PORT) <- (   *    ,    *    ) ]
```

2. Restricted Cone NAT
   - Full Cone NAT와 다르게 기존에 통신하던 IP가 아니라면 host로 패킷이 도달하지 않는다.
   - Port Restricted Cone NAT과 구분하기 위해 Address Restricted Cone NAT이라고도 한다 -> IP 주소를 제한한다는 뜻

```bash
    {NAT internal side}  |    {NAT external side}  |  {Remote host}
                         |                         |
1. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, INT_PORT) -> (REM_ADDR, REM_PORT) ]
2. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, INT_PORT) <- (REM_ADDR,    *    ) ]
```

3. Port Restricted Cone NAT

   - Restricted Cone NAT에서 한 단계 더 나아가 por 번호도 제한한다.
   - 기존에 통신하던 IP라고 하더라고 port 번호가 다르다면 통신할 수 없다.

```bash
    {NAT internal side}  |    {NAT external side}  |  {Remote host}
                         |                         |
1. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, INT_PORT) -> (REM_ADDR, REM_PORT) ]
2. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, INT_PORT) <- (REM_ADDR, REM_PORT) ]
```

4. Symmetric NAT

   - Port Restricted Cone NAT과 유사한데, 한 단계 더 나아가 outbound transmission마다 external side에서 새로운 port를 부여한다.
   - internal side와 external side의 매핑이 지속적으로 바뀐다는 것이다.

```bash
    {NAT internal side}  |    {NAT external side}  |  {Remote host}
                         |                         |
1. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, EXT_PORT1) -> (REM_ADDR, REM_PORT1) ]
2. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, EXT_PORT1) <- (REM_ADDR, REM_PORT1) ]
...
3. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, EXT_PORT2) -> (REM_ADDR, REM_PORT2) ]
4. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, EXT_PORT2) <- (REM_ADDR, REM_PORT2) ]
```

STUN이 실패하는 가장 큰 이유가 바로 Symmetric NAT 때문이다. RTCPeerConnection이 다른 목적지로 패킷을 적용하면, external side port가 계속 바뀐다. 즉 STUN 서버로 요청을 보낼 때 사용된 external port가 다른 RTCPeerConnection과 소통하기 위해 사용된 external port와 다르기 때문이다. [RFC3489](https://datatracker.ietf.org/doc/html/rfc3489)에 따르면 STUN은 Full Cone NAT에서만 사용 가능하다. 그리고 Restricted Cone Nat과 Port Restricted Cone NAT의 경우 application 상태에 따라 가능하지만, Symmetric NAT의 경우에는 사용 가능한 public IP 주소를 return하지 않는다고 나와있다.

이를 해결하기 위해 TURN이 사용되는데, TURN은 P2P통신은 아니고, RTCPeerConnection이 전달하는 패킷을 relay--전달 또는 배달--하는 역할을 한다. 다음에는 TURN에 대해 조금 더 자세히 공부해봐야겠다.

참고자료

1. https://doc-kurento.readthedocs.io/en/stable/knowledge/nat.html
2. https://www.ietf.org/rfc/rfc3489
