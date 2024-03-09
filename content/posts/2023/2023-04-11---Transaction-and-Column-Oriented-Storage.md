---
title: "Column-Oriented Storage"
date: "2023-04-11T20:35:37.121Z"
template: "post"
draft: false
slug: "/database/column-oriented-storage"
category: "Database"
tags:
  - "Database"

description: "Colum-Oriented가 read효율을 극대화하는 방법"
---

[이전편](https://jasonkang14.github.io/database/index-to-improve-read)에서 계속된다

# Transaction Processing or Analytics

Transaction은 흔히 말하는 ACID(Atomicity, Consistency, Isolation, Durability)와는 관계없다. Transaction은 그것보다는 사용자들이 low-latency로 read/write를 할 수 있게 하는 것에 가깝다.

요즘 빅데이터나 데이터 분석 등이 부상하면서, 큰 규모의 데이터를 읽어와서 분석하는 경우가 발생한다. 이를 Online Analytic Processing(OLAP)라 칭한다. 이와 반대로 Online Transaction Processing(OLTP)는 어플리케이션과 사용자의 interaction을 통해 데이터베이스가 업데이트되고, read/update/write등이 일어나는 것을 뜻한다. 데이터 분석에 비해서 읽어오는 양이 적다.

처음에는 목적에 상관없이 OLAP, OLTP모두 같은 데이터베이스를 사용했지만, SQL이 발달하면서, 목적에 따라 다른 데이터베이스를 사용하기 시작했다. 데이터분석을 위해 사용되는 데이터베이스를 `data warehouse`라 칭한다.

### Data Warehousing

서비스 규모가 커지고, 사용자들이 많아지면서 low-latency를 통한 read/write의 필요성이 더 대두되기 시작했다. 따라서 많은 회사들이 OLTP형식의 데이터베이스를 HA 상태로 운영한다. 해당 데이터베이스에 분석을 위해 ad hoc query들을 날리면, 데이터 분석용 쿼리가 더 비용이 클 가능성이 높기 때문에, 사용자들의 서비스 사용에 문제를 야기할 수 있다. 

이로인해 데이터 분석용으로 data warehouse를 따로 구축하는 것이다. 사용자들이 접근하는 OLTP작업에 영향을 미치지 않는 상태로 OLAP query들을 날릴 수 있는 별도의 데이터 저상소를 구축한다. OLAP 데이터베이스에 적재되는 데이터는 OLTP 데이터베이스에서 추출된다. 추출된 데이터는 분석하기 편리한 상태로 처리되어 data warehouse에 적재되는데, 이 과정을 Extract-Transform-Load(ETL)이라 부른다. 

작은 서비스의 경우 OLAP를 따로 구축할 필요성을 느끼지 못했을 수 있다. 하지만 OLAP는 분석에 용이하게 데이터를 처리하면서 색인 등을 활용한 검색 최적화가 가능하다. OLTP 데이터베이스와 data warehouse는 겉으로 보기엔 비슷해보이지만, 쿼리의 방식이나 목적이 다르기때문에 내부적으로는 다르게 구현되어있다. 따라서 다른 구조로 이루어진다. 

### Stars and Snowflakes: Schemas for Analytics

서비스 운영에는 필요에 따라 적합한 data model을 선정해야하지만, 데이터 분석에는 대부분 `star schema`를 사용한다. 일반적으로 `fact table`을 중심으로 구성되고, 해당 테이블의 row에는 timestamp와 함께 그 시간에 발생한 이벤트가 같이 기록된다. 

`fact`는 flexibility를 제공하기위해 이벤트 단위로 기록된다. 일반적인 관계형 데이터베이스와 유사하게 fact table의 컬럼은 해당 테이블에 해당하는 attribute와 다른 테이블과의 관계를 나타내는 foreign key로 이루어져 있다. fact table과 연결된 다른 테이블들은 `dimension table`이라고 칭한다. fact table의 row는 어떤 이벤트가 발생했는지를 기록하고, dimension table의 row는 이 이벤트의 육하원칙에 해당하는 정보들을 저장한다.

timestamp를 fact table이 아니라 dimension table에 넣는 경우도 있는데, 이를 통해 확장성을 가져갈 수 있기 때문이다. 예를들면 date정보를 인코딩해서, 휴일과 평일 등을 구분하는 쿼리를 날리거나 하는데 사용된다. 

star schema이외에 `snowflake schema`도 많이 사용된다. star schema의 dimension table을 `subdimension table`로 한 번 더 쪼개는 방향이다. 

# Column-Oriented Storage

[Column-Oriented](https://jasonkang14.github.io/sql/column-oriented-database)에 관해서는 예전에 공부하고 글을 썼던적이 있다. 요약하면 데이터를 column 단위로 저장해서, 한 번 쿼리를 날릴 때 여러 디스크를 조회하지 않아서 더 빠르게 read를 가능하게 하는 것이다. 예를들면 

```sql
SELECT age FROM users;
```

라는 쿼리를 할 때, `age`들끼리 묶어서 저장해서 Read효율을 높이는 것이다. 약간 indexing이 필요 없는 느낌? 이를 통해 random I/O를 줄일 수 있는 것 같다. 

### Column Compression 

데이터를 압축하면 disk throughput도 줄일 수 있다. 압축을 위해 `bitmap encoding`을 많이 사용하는데, n개의 고유한 값들(편의상 cardinality라고 칭하겠음)을 n개의 bitmap으로 인코딩한다. 예를들면 1부터 20까지의 숫자가 랜덤하게 있다고 할 때, 14를 나타내는 bitmap은 14를 1로, 14를 제외한 나머지 값들을 0으로 규정한다. cardinality가 낮으면, row한개에 하나의 bit를 넣을 수 있다. 하지만 높다면 bitmap에 0이 엄청 많을거기 때문에, run-length 인코딩을 추가해서 더 압축할 수 있다. 

##### Memory bandwidth and vectorized processing

read할 데이터가 많다면, 디스크에서 메모리로 값을 불러올 때의 bandwidth가 바틀넥이 될 수 있다. 뿐만아니라, 메인 메모리에서 CPU 캐시로 값을 효율적으로 옮기는 것도 고민해야한다. column-oriented storage는 bandwidth측면에서도 유리하고, CPU cycle 사용에서도 유리하다. 컬럼 데이터를 압축해서 CPU L1 cache에 넣으면, function call없이 데이터를 계속 불러올 수 있다. 

### Sort Order in Column Storage

컬럼 스토어에서는 row가 저장된 순서가 중요하지 않다. 그냥 들어오는 순서대로 저장하면 된다. 하지만 인덱싱을 사용해서 순서를 정해줄수도 있다. row가 들어운 순서가 중요하지는 않지만 column이 저장되는 순서는 중요하다. 순서가 섞이면 안되는 이유는, 각 컬럼의 순서들이 해당 값들이 같은 Row에 속한다는 것을 보장해주기 때문이다. 예를 들면 데이터가 날짜를 중심으로 쿼리되는 경우가 많다면, date_key를 사용해서 순서를 정할 수 있다. 일반적인 RDB의 multi index를 사용해서 인덱스별 순서를 주는 것처럼, sort_key도 순서를 줄 수 있다. 

순서를 정하는 것이 필수는 아니지만, 데이터 압축에서 장점이 있다. 위에서 언급한 bitmap encoding을 생각해보면, 고유값의 수가 적게되면 결국 하나의 row에 bitmap을 다 넣을 수 없다는 문제가 생긴다. run-length encoding작업이 들어가면 가능하겠지만, 가급적이면 sorting을 통해 first sort_key가 겹치지 않도록 배정하면서 압축 효율을 향상시키는 편이 좋다. 

##### Several different sort orders

쿼리의 종류에 따라 sort order가 다르니, 데이터를 필요에 따라 다양한 방법으로 sorting하는 방법이다. 처음에 C-Store에서 소개돼서 Vertica에도 적용된 방식이다. Single point of failure를 방지하기 위해 데이터는 여러 머신들에 복제되어 저장된다. 따라서 복제된 데이터를 저장할 때, 필요에 따른 sort order에 맞게 값을 저장하는 것이다. 

여러 sort order를 갖는 것은 RDB에서 secondary index를 여러개 갖는것과 유사하다. 하지만 row-oriented의 경우에는 모든 row를 한곳에 저장하고, secondary index는 해당 row에 pointer만 가지고 있지만, column-oriented에서는 pointer는 없이 실제 값만 존재한다. 

### Writing to Column-Oriented Storage

column oriented의 경우 위에서 언급한 read에서의 장점이 있지만, write가 어렵다는 단점이 있다. sort가 추가된 데이터베이스에 row를 추가하려고 한다면, 전체 컬럼파일을 다시 write해야하는 문제가 있다. row가 column내의 position에 따라 인식이 되기 때문이다. 

LSM-Tree를 활용하면 이를 일부 해결할 수 있는데, write가 필요한 값들을 sort해서 memory에 저장해뒀다가 디스크에 밀어넣는 식이다. in-memory store는 row-oriented건 column-oriented건 상관없다. write가 충분히 쌓이면 컬럼 파일로 합쳐지고, bulk형태로 disk로 write되는 방식이다. 따라서 query는 디스크와 memory에 최근에 write된 값들을 모두 확인해서 결과를 하나로 합쳐야한다. 

### Aggregation: Data Cubes and Materialized Views

모든 data warehouse가 column-oriented인 것은 아니다. 기존의 row-oriented들도 사용될 수 있는데, ad hoc query등에 column-oriented가 확실한 강점을 보이니 최근 상승세를 보이는 것이다. 

`materialized aggregates`도 data warehouse의 중요한 포인트 중 하나이다. data warehouse는 COUNT, SUM, AVG, MIN등의 aggregation 함수를 사용할 수 있다. 만약 여러 쿼리들이 해당 함수들을 사용한다면, raw데이터를 계속 불러오면서 낭비가 생길 수 있다. 이를 방지하기 위해 COUNT, SUM등의 데이터를 cache해두는 것이다. 

`materialized view`는 이런 캐시를 생성하는 한가지 방법이다. 관계형 데이터 모델에서는 standard view라고 불린다. 쿼리 결과로 이루어진 table처럼 보이는 객체를 뜻한다. 차이가 있다면 materialized view는 디스크에 기록된 쿼리 결과들의 실제 복제본들이고, virtual view는 쿼리를 작성하기위한 shortcut이라는 것이다. virtual view에서 read를 시도하면 SQL engine이 view의 쿼리에 접근해서 해당 쿼리를 실행한다. 

만약 데이터가 변경된다면 materialized view도 변경되어야 한다. 왜냐면 데이터가 변경되면 더이상 정규화된 값이 아니기 때문이다. 데이터베이스가 이를 자동화 할수도 있지만 그럴경우 write비용이 증가하는 문제가 있다. `data cube` 또는 `OLAP cube`라고 불리는 것이 materialized view로 많이 사용된다. 위에서 언급한 다양한 dimension table들을 엮어두는 표 정도로 이해하면 된다. 
