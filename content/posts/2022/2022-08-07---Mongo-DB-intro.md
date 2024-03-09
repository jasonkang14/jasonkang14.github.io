---
title: Mongo DB
date: "2022-08-07T21:41:37.121Z"
template: "post"
draft: false
slug: "/database/explaing-mongo-db"
category: "Database"
tags:
  - "Database"

description: "NoSQL중 MongoDB에 대해 알아본다"
---

MongoDB는 Cassandra DB와 같은 카테고리에 속한 NoSQL 데이터베이스이다. 차이가 있다면, CAP Theorem에서 CassandraDB는 consistency를 포기하고 availability를 우선시한다면, MongoDB는 availability를 포기하고 consistency를 우선시한다. 

`consistency`는 데이터가 일관성이 있어야 한다는 뜻이다. 어떤 node에 접근하던지 상관없이 query를 시도하면 같은 데이터를 받아올 수 있어야 한다는 것을 뜻한다. CassandraDB의 경우 해당 데이터가 저장되지 않은 node에 read 요청을 보내면 내가 원하는 데이터를 불러올 수 없는 문제(?)가 있었다. 인스타그램을 예로 들면, 데이터가 복제되지 않은 cluster에 read요청을 보내는 것이 아니라면, 포스트를 작성하고 나서, 내가 작성한 포스트를 확인하기 까지 시간이 소요된다.

반면 MongoDB는 Relication Set들로 이루어졌기 때문에 때문에, 어떤 클러스터에 요청을 보내더라도 read 할 수 있다는 장점이 있다. 하지만 모든 node들이 서로 소통하며 별도의 master node를 갖고 있지 않은 Cassandra DB와 달리, MongoDB는 master node가 하나이기 때문에 availability가 떨어지는 단점이 있다. 

그럼 왜 이렇게 될 수밖에 없는지 MongoDB의 구조에 대해 살펴본다. 

Mongo DB는 `relication set`들로 이루어져 있다. 단어에서 유추할 수 있듯 MongoDB는 데이터를 복제해서 저장한다. 하나의 replication set은 하나의 `primary node`(다른 데이터베이스에서 master node의 역할)와 여러 `secondary node`로 이루어져 있다. 

![mongo-db-replication-set](https://i.imgur.com/kV7WrP1.png)

그림에서 보이는 것과 같이 client application들은 `primary node`로만 write할 수 있고 read도 우선적으로 `primary node`를 통해서만 가능하다. 

`replication set`에서는 `primary node`로 write된 데이터들이 `secondary node`들로 복제된다. 데이터들이 `secondary node`들로 복제되기 때문에, write 요청은 `primary node`로만 보낼 수 있지만, `secondary node`들로도 read 요청은 보낼 수 있다(추가 설정 필요). 따라서 `primary node`가 죽더라도 read는 가능하다.

이러한 구조가 MongoDB의 consistency를 보장한다. 

하지만 이 `replication set` 구조에서 node를 항상 홀수개로 올려야 하는데, 그 이유는 MongoDB의 node들이 **투표를 해서 과반수로** 어떤 node가 `primary node` 역할을 할지 정하기 때문이다. 처음 `primary node`역할을 하는 node가 죽으면, `primary node`의 역할을 할 node가 새로 선정되는 방식이다. 이로인해 실제 데이터를 저장하지는 않지만 투표에만 참여하는 node를 올리는 방식도 가능하다.

![mongo-db-primary-node-election](https://i.imgur.com/ro6si6k.png)

이 `primary node`가 죽고 새로운 `primary node`가 선출될 때까지 write 기능을 담당할 수 없기 때문에 availability가 떨어지는 것이다. 

CAP Theorem 기준 partition tolerance도 보장하는 MongoDB는  분산저장(Sharding)이 가능하다. MongoDB의 분산 클러스터는 `shard`, `mongos`, `config server` 3개 구조로 이루어져있다.

![mongo-db-sharding](https://i.imgur.com/XUIjx1j.png)

각각의 `shard`는 하나의 `primary node`와 짝수개의 `secondary node`를 가진 `replication set`으로 이루어져 있다. 그리고 하나의 MongoDB 데이터베이스는 하나의 `primary shard`를 가지고 있는데, 이 `primary shard`는 분산되지 않은 `collection`(관계형 데이터베이스의 테이블에 해당)들을 모두 가지고 있다. 

![mongo-db-primary-shard](https://i.imgur.com/uiFXO6A.png)

Shard A가 모든 `collection`들의 정보를 가진 primary shard인듯하다. 새로운 데이터베이스가 생성될 때, `mongos`가 가장 적은 데이터를 가지고 있는 `shard`를 `primary shard`로 선택한다. 

그렇다면 이 `mongos`는 무엇인가? `mongos`는 shard가 어떻게 되어있는지, 즉 어떤 `shard`에 어떤 데이터가 저장되어있는지에 대한 정보를 가지고 있는 router이다. 따라서 클라이언트가 MongoDB로 요청을 보내면, `mongos`가 그 request를 해당 데이터를 가지고 있는 `shard`로 redirect한다. 

`config server`는 MongoDB 분산 클러스터의 모든 metadata를 가지고 있는 `replacation set`이다. 위 그림에서 알 수 있듯이 하나의 `primary node`와 짝수개의 `secondary node`로 이루어져 있다. 

클라이언트의 요청을 redirect하는 `mongos`는 이 `config server`의 데이터를 cache해서 가지고 있다. 그리고 이 데이터를 가지고 클라이언트의 요청을 routing한다. 또한 `mongos`가 `config server`의 데이터를 cache하고 있기 때문에, `config server`가 죽더라도 MongoDB는 정상적으로 작동한다. 

CassandraDB와 MongoDB를 비교하는 표를 작성하려고 했는데, [MongoDB 문서](https://www.mongodb.com/compare/cassandra-vs-mongodb)에 잘 정리되어 있어 링크로 대신한다. 

#### 참고문헌
[MongoDB](https://www.mongodb.com/docs/upcoming/core/replica-set-primary/)