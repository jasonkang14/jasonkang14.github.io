---
title: B+ Tree
date: "2022-06-06T13:34:37.121Z"
template: "post"
draft: false
slug: "/sql/b-plus-tree-database-indexing"
category: "Database"
tags:
  - "Database"

description: "B+ Tree 구조가 Database Indexing에 미치는 영향"
---

# TL;DR

### B+Tree 구조는 insert/update/delete에 쉽게 대응할 수 있고, Search가 빠르기 때문에 데이터베이스 인덱싱에 적합하다.

[Database indexing](https://jasonkang14.github.io/sql/database-indexing)에 대해 공부해보니, 효율성이 B+ Tree구조에서 온다고한다.

인덱싱이 왜 빠른지는 이해했으니, B+ Tree구조를 적용한 이유에 대해서 공부하고자 한다. B+ Tree를 학습하기 전에, B Tree 를 먼저 보자.

B Tree는 이런 구조로 이루어져 있다.
![B Tree Structure](https://i.imgur.com/zGU03Yj.png)

Tree 구조라는 이름 때문에 root, branch, leaf 등의 이름을 쓰는 것 같다. 각 박스는 node라고 불린다. binary search tree와 유사한 구조인데, Binary tree와 다르게 하나의 노드가 2개가 넘는 자식(?) node를 가질 수 있다. B Tree의 장점은 leaf node에 있는 항목들을 검색할 때, 모두 같은 시간 내에 찾을 수 있다는 점이고, 단순 리스트를 검색하는 것 과 비교했을 때 훨씬 빠르다 `log(n) vs n`

하지만 B Tree 의 문제점은, update가 빈번하게 발생하는 테이블의 경우 Tree의 균형이 깨져서 효율이 떨어지기 때문에 주기적으로 인덱싱을 업데이트 해줘야하는 단점이 있다 또한 모든 값들이 순서대로 저장되기 때문에, 인덱싱을 업데이트 할 때 비효율적이다.

---

B+Tree는 B Tree의 확장형 개념인데, 가장 큰 차이점은 leaf nodes 들이 Linked Lists로 연결되어 있고, branch node에는 key만 담아두고 모든 값들은 Leaf node에 저장된다.

leaf node를 제외하고는 데이터를 가지고 있지 않아서 메모리에 여유가 있고, 하나의 node에 B Tree보다 더 많은 값들을 담을 수 있기 때문에, tree의 높이가 낮아지는 장점이 있다.

또한 데이터베이스 전체를 풀스캔 하는 경우에는값을 가지고 있는 leaf node만 보면 된다. 그리고 leaf node들이 linked list로 이루어지기 때문에, B Tree에서 겪는 균형이 깨진다던지, 인덱싱을 업데이트 해줘야 한다던지의 문제가 없다.

두가지를 비교한다면,

![Compare B+ Tree with B Tree](https://i.imgur.com/Mh6KdDb.png)

참고
https://www.guru99.com/introduction-b-plus-tree.html
https://www.geeksforgeeks.org/difference-between-b-tree-and-b-tree/
