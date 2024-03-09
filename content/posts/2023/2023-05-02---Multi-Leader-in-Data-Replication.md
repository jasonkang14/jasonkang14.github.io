---
title: "복수리더로 데이터 복제"
date: "2023-05-02T20:35:37.121Z"
template: "post"
draft: true
slug: "/database/multi-leader-in-replication"
category: "Database"
tags:
  - "Database"

description: "데이터 복제 시 리더가 복수인 경우 단일 리더와 어떤 점이 다른가?"
---

[이전 포스트](https://jasonkang14.github.io/database/distributed-data-replication-single-leader)에서 이어진다

# Multi-Leader Replication

리더가 하나인 경우 모든 write 요청을 해당 리더가 처리해야한다. 따라서 리더와 네트워크 오류등으로 인해 연결할 수 없다면 write는 실패한다. 이를 해결하기 위해 여러개의 노드들이 write 요청을 처리할 수 있도록 하는 방안이 있다. write 후에는 단일 리더와 마찬가지로 모든 노드들에 복제된다. 

### Use Cases for Multi-Leader Replication

일반적으로 데이터센터가 하나라면 여러개의 리더를 두는 것이 불필요하다. 하지만 복수의 리더로 설정하는 것이 좋을 때가 있다. 

##### Multi-datacenter operation

replica가 여러 데이터센터에 흩어져있는 경우, 리더가 하나라면 하나의 데이터센터에 존재하고, 해당 리더는 write 요청을 받고 하나의 데이터센터에 write를 성공하면 해당 데이터를 모든 데이터센터에 뿌려줘야한다. 즉 하나의 데이터센터만 write를 요청을 처리할 수 있다. 리더가 여럿인 경우 각각의 데이터센터에 리더를 둘 수 있다. 리더는 write 요청을 처리하고, 다른 데이터센터의 리더에게 write된 데이터 정보를 전달하는 방식이다. 

