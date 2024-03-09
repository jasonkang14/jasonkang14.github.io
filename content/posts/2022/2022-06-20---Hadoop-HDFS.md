---
title: Hadoop - HDFS
date: "2022-06-20T22:53:37.121Z"
template: "post"
draft: false
slug: "/hadoop/hdfs"
category: "Hadoop"
tags:
  - "Hadoop"

description: "Hadoop에서 HDFS가 하는 역할"
---

# TL;DR

### HDFS는 대규모의 데이터를 분산 저장하여 read 효율을 향상시킨다

글또에 참여하면서 자연스럽게 평소에 관심있던 데이터 엔지니어링 분야의 스터디에 참여하게 되었다. 데이터 분야에서 근무중인 다른 엔지니어 분들을 통해 많이 배우고 있다. 매 주 한명씩 발표를 하는데, 발표를 담당했던 HDFS에 대해 이야기 해보려고 한다.

가장 큰 특징은 데이터를 `클러스터` 전반에 나누어서 저장하고. 데이터에 빠르고 안정적으로 접근할 수 있다는 것이다. 이렇게 나누어진 데이터를 `block`이라고 칭하고, 한 `block`은 128MB이다. 또한 여러개의 컴퓨터에서 동시에 특정 데이터에 접근할 수 있다.

![HDFS 저장방식](https://i.imgur.com/m8rGbKK.png)

HDFS의 기본구조는 아래 그림과 같이 1개의 Name Node와 여러개의 Data Node로 구성되어 있다.

![HDFS Architecture](https://i.imgur.com/8o8HG7B.png)

`data node`는 실제 `block`들이 저장되는 공간이고, `name node`는 분산되어 저장된 각 `block`들이 어떤 `data node`에 저장되어있는지 기억하는 역할을 한다. HDFS는 데이터가 들어오면 원본만 저장하는 것이아니라, 복사본도 저장을 하는데, `name node`는 그 복사본들의 위치까지 저장한다. 또한 언제 데이터의 edit log도 `name node`에 저장된다.

따라서 `client node`가 `HDFS`에서 파일을 읽기 위해서는 `block`의 모든 정보를 가지고 있는 `name node`에 먼저 접근한다. `name node`로부터 내가 접근하고자 하는 `block`이 어떤 `data node`에 있는지 확인하고, 해당 `data node`에 접근해서 원하는 `block`을 읽어낸다.

![reading a file from HDFS](https://i.imgur.com/dXbyswZ.png)

`HDFS`에 파일을 쓸 때도 역시 `name node`에 먼저 접근한다. `name node`는 `client node`가 작성하고자 하는 데이터의 Entry를 특정 `data node`에 생성하고, 해당 정보를 `client node`에 전달한다. `client node`는 `name node`로 부터 전달받은 `data node`에 write를 시작하고, 해당 block은 data node로 분산되어 저장된다. block의 위치에 관한 정보와 edit log는 `name node`에 저장된다.

`name node`가 하나인 이유는, `name node`가 여러개인 경우에 문제가 발생할 수 있기 때문이다. 만일 여러 `client node`들이 동시에 다양한 `name node`들에게 `block`의 정보를 요청하는 경우, 각 `client`들이 같은 `block`에 접근한다 하더라도 다른 정보를 받아올 수 있다.

그렇다면 모든 정보를 가지고 있는 `name node`가 고장난다면 어떻게 해야할까? `HDFS`는 `name node`가 고장날 것을 대비해 이런저런 장치들이 있다.

1. meta data backup
   - local disk나 file system에 name node가 가지고 있는 Edit log를 저장한다.
   - name node가 죽게되면, 해당 disk에서 edit log를 불러온다.
   - 디스크에 접근하는 것은 오래걸리기 때문에 정보를 조금 유실할 수는 있지만, 완전히 잃어버리는 것보다는 낫기 때문이 에런 방식을 사용한다

여기저 저장하는 메타데이터에 대해 조금 더 알아보자면, hadoop은 원래 디스크 말고 메모리에서 파일명, 디렉토리, 블록크기, 소유자, 파일속성 등 모든 메타데이터를 관리한다. 이 메타데이터는 `fsimage`와 `edits`로 나누어지는데, `fsimage`가 파일명, 디렉토리, 블록크기, 소유자, 파일속성과 같이 메모리 상에서 관리되는 메타데이터 내의 파일 이미지이다. socket의 health check와 유사한 `check point`라는 시점에 name node의 로컬 파일시트템에 생성되고, data node 블록 정보는 포함하지 않는다. `edits`는 로컬 파일 시스템에 생성되는 `edit log`이다.


2. secondary name node

   - 완전히 동일한 name node는 아니고 name node가 가지고있는 `edit log`의 copy를 저장한다
   - name node가 죽으면 바로 대체할 hot standby는 아니지만, name node가 하나 더 있는 방식이기 때문에 meta data를 backup하는 첫번째 방식보다는 복구가 빠르다는 이점이 있다.
   - secondary name node는 기존의 name node와는 다른 클러스터에서 돌아가고, meta data에서 설명한 `fsimage`와 `edits`를 주기적으로 name node에서 받아와서 저장한다.

![secondary name node](https://i.imgur.com/2tH3Fbj.png)

3. High Availability
   - 이 단어는 여기저기서 자주 사용된다. hot standby라고 보면 된다
   - 동시에 두개를 켜놓고 잘 돌던게 죽으면 바로 대체한다
   - hadoop 생태계의 다른 프로그램(?)인 `Zookeeper`가 어떤 name node가 메인이고, 어떤 name node가 hot standby인지를 파악한다.

![High Availability](https://i.imgur.com/nsul5YQ.png)

앞으로 조금씩 hadoop에 관한 포스팅을 해보도록 하겠다. 회사 인프라팀에서 쿠버네티스를 구축하는 중인데, 요즘 하둡 스터디 한다고 하니까 긍정적인 반응을 보이셨다. 공부 잘 해서 회사에서도 적용할 수 있었으면 좋겠다.

참고자료

- [udemy 강의](https://www.udemy.com/course/best-hadoop/learn/lecture/28318946?start=30#overview)
