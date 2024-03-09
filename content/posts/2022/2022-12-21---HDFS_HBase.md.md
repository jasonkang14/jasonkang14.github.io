---
title: 하둡 HBase
date: "2022-12-21T19:35:37.121Z"
template: "post"
draft: false
slug: "/hadoop/what-is-hbase"
category: "Hadoop"
tags:
  - "Hadoop"

description: "하둡에서 Column Oriented Database가 필요한 이유"
---

`HBase`는 `HDFS`위에서 돌아가는 column-oriented 분산 데이터베이스이다. 큰 규모의 데이터셋을 실시간으로 read/write를 할 때 좋다.

일반적으로 데이터베이스들은 partition이나 복제를 통해 큰 데이터를 관리하려고 한다. 하지만 join, complex query등의 문제가 있고, foreign key constraint 문제가 생기면 구동하기 매우 비싸지는 단점이 있다.

`HBase`는 이를 해결하기 위해 node를 추가하는 방식으로 linear scalability를 추구한다. 관계형이 아니고, SQL을 지원하지는 않지만, 일반적인 관계형 DB에서 할 수 없는 저렴한 하드웨어에서 대용량 데이터 처리가 가능하다.

# Concepts

### Datamodel

application은 데이터를 레이블된 테이블에 저장한다. 테이블은 row와 column으로 이루어져있고, row와 column이 만나는 곳을 cell이라고 부른다. cell에는 version 정보가 포함되어있고, `HBase`는 insert time에 timestamp를 version 정보에 포함시킨다. cell의 내용은 array of bytes이다. `HBase`가 데이터를 저장하는 방식은 아래 그림에 잘 나타나있다.

![hbase-data-model](https://i.imgur.com/CLpjEQO.png)

table row key 또한 bytearray이기 때문에 이론상으로는 row key로 아무 값이나 사용할 수 있다. string이 될 수도 있고, serilized된 자료구조일수도 있다. row들은 key(primary key)로 sorting 된다.

column들은 column family로 그룹화 된다. 일반적으로 `family:detail` format으로 공통된 prefix를 가진다. 물리적으로는 같은 family member들은 file system에 같이 저장된다. 따라서 tuning과 specification을 위해서 column family 멤버들은 같은 패턴이나 사이즈를 갖기를 권장한다.

테이블들은 `HBase`에 의해 수평적으로 `region`이라는 단위로 partition 된다. 각 region은 테이블 row의 subset으로 구성되어 있다. region이 너무 커지면 두개로 나눠지는데 split이 일어나기 전까지는 region은 하나의 서버에서 지속적으로 hosting된다.

### Implementation

master node와 regionserver worker들로 이루어져 있다. 다른 시스템들과 유사하게 master node가 모든 정보를 가지고 있고, regionserver worker들이 실제로 데이터를 저장하는 느낌이다.

master는 최초 설치와, regionserver들에 region들을 할당하는 역할을 하고, regionserver에서 fail이 발생하면 회복하는 역할을 담당한다. regionserver들은 region들을 가지고 있고, client의 read/write request를 수행한다. 또한 region split이 일어나는 경우 master에게 내용을 전달한다.

![hbase-cluster-members](https://i.imgur.com/ntLx8oY.png)

`HBase`는 `ZooKeeper`에 의존한다. `ZooKeeper`가 `HBase` meta table이나 현대 master의 위치를 저장한다. 만약 region을 assign 할 때 문제가 발생하면, `ZooKeeper`가 이또한 해결한다.

regionserver worker node들은 HBase의 `conf/reginservers`라는 파일에 저장되어 있다. HBase는 하둡의 filesystem API를 사용해서 데이터를 제공한다. 일반적으로 HBase는 HDFS위에서 storage용도로 사용되고, HBase는 local filesystem에 write한다.

HBase가 구동될 때는 `hbase:meta`라는 catalog table에 현재 regionserver들의 리스트, 상태, 그리고 클러스터내에서의 위치정보들을 기록한다. row는 sort되어있기 때문에, 자료를 찾을 때 인덱싱이 적용된 개념과 유사하다고 보면 된다.

처음으로 HBase를 사용하려고 하면, `ZooKeeper` cluster와 연결해서, `hbase:meta`의 위치를 가져온다. 클라이언트는 그 정보를 바탕으로 `hbase:meta` region에 lookup을 시도해서 space region과 그 위치 정보를 가져온다. 정보를 가져오면 hosting regionserver와 바로 소통해서 원하는 데이터를 가져올 수 있다.

계속해서 `base:meta`정보를 확인하는 것을 방지하기 위해서, 클라이언트는 lookup을 할 때마다 해당 정보를 cache한다. 같은 곳에서 데이터를 가져와야 한다면 굳이 `hbase:meta`에 lookup을 시도하지 않고, cache된 정보를 바탕으로 regionserver와 직접 소통한다. 만약 region이 이동한다면, `hbase:meta`를 다시 조회하고, 그래도 실패한다면 `ZooKeeper`와 다시 소통해서 정보를 가져온다.

write를 하는 경우에는 regionserver에서 일어난 작업들이 `commit log`에 append되고, 그 이후에 in-memory `memstore`에 저장된다. `memstore가` 가득차면, 내용들이 filesystem으로 이동된다. 이 Commit log는 HDFS에 저장되기 때문에 regionserver가 죽더라도 정보가 계속 남아있는다는 장점이 있다. regionserver가 죽는 경우는 ZooKeeper에 있는 znode가 죽는 경우가 대부분인데, 죽은 regionserver의 commit log를 region단위로 나눈다.그리고 다시 살아나면 죽었던 시점부터 write를 이어서 한다.

read를 할 때는 `memstore`에 먼저 접근한다. `memstore`에 충분한 정보가 있으면 거기서 끝나고, 추가로 read해야 한다면 query를 실시한다.

설치는 Apache 공홈을 추천하고, 책에는 `MapReduce`등을 포함한 예제코드가 있다.

# RDBMS와 비교

구현 방법과 탄생 배경은 다르지만, 같은 문제를 해결할 수 있다는 점에서 많이 비교된다.

`HBase`는 분산 column-oriented 데이터 storage 시스템이다. HDFS위에서 구동되면서 random read/write access를 제공한다. `HBase`는 다양한 방향으로 scalability를 고려해서 만들어졌다. row, column모든 방향으로 확장이 가능하고, 분산 시스템이기 때문에 paritition, replication을 통해서 저렴한 하드웨어에서도 구동이 가능하다는 장점이 있다. 테이블 스키마는 물리적 storage와 유사하기 때문에, 효율적인 직렬화를 통해서 데이터를 저장하고, 불러올 수 있다. 이 효율을 극대화 하는 것은 개발자의 능력이다.

반면 `RDBMS`는 fixed schema이고, row-oriented 데이터 베이스이다. ACID를 기본 원칙으로 하고 SQL query engine을 제공한다는 장점이 있다. consistency, integrity, abstraction에 장점이 있고, 복잡한 쿼리를 SQL 언어를 통해 수행할 수 있다. 또한 매우 간단하게 secondary index를 추가할 수 있고, join이 가능하며, 다양한 연산들이 가능하다는 장점이 있다.

일반적으로 small-to-medium 사이즈의 프로젝트에서는 MySQL이나 PostgreSQL같은 RDBMS로 충분히 구현이 가능하다. 하지만 데이터 사이즈가 엄청 크거나, read/write를 병렬적으로 하는 경우가 잦다면, `HBase`가 대안이 될 수 있다.
