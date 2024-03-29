---
title: "인덱스를 활용한 Read 효율 개선"
date: "2023-04-05T20:35:37.121Z"
template: "post"
draft: false
slug: "/database/index-to-improve-read"
category: "Database"
tags:
  - "Database"

description: "실행할 query에 맞게 인덱스를 활용하면 read 효율을 개선할 수 있다"
---

데이터베이스는 2가지 역할을 담당한다.

1. 데이터베이스로 전달한 데이터를 저장하고
2. 데이터베이스로 요청한 데이터를 제공한다

어플리케이션 개발자가 데이터베이스가 데이터를 어떻게 저장하고 불러오는지 알아야하는 이유는, 데이터베이스 자체를 직접 개발하지 않더라도, 어플리케이션에 적합한 데이터베이스를 선택할 줄 알아야하기 때문이다. 본인의 개발로드에 적합하도록 데이터베이스를 튜닝해야 한다면, 스토리지 엔진이 어떻게 돌아가는지 어느정도 이해하고 있어야한다. 

# Data Structures That Power Your Database

아래와 같은 key-value store가 있다고 생각해보자. 

```bash
#!/bin/bash
db_set () {
  echo "$1,$2" >> database
}
db_get () {
  grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

스토리지 포맷은 매우 간단하다. 텍스트 파일(.txt)에 key와 value를 comma로 구분한 csv와 유사한 형태이다. `db_set()`을 호출할 때 마다 가장 마지막줄 뒤에 새로운 데이터를 추가하는 시스템이다. 데이터를 추가할 수는 있지만, 기존의 데이터를 삭제하거나 overwrite하는 것은 불가능하다. 

기존 데이터 맨 뒤에 값을 추가하는 append는 일반적으로 매우 효울적이기 때문에, `db_set()`은 꽤 좋은 성능을 보일것이다. 많은 데이터베이스들이 로깅을 위해 `db_set()`과 유사한 함수를 사용한다. 로깅은 기존에 있는 데이터를 유지하면서 맨 끝에 새로운 텍스트를 추가하기 때문이다. 

반면에 `db_get()`은 매우 비효율적이다. 특정 key를 사용해서 값을 찾을 때, 전체 텍스트 파일을 모두 훑어봐야 하기 때문이다. read를 효율적으로 하기 위해서는 `인덱스`가 필요하다. 인덱스에 대해 간단하게 설명하자면, 메타데이터를 다른 곳에 따로 저장해서, 어디를 read할지 미리 파악하는 것이다. read의 방식에 따라 다양한 인덱스를 활용할수도 있다. 개인적으로 인덱스에 대해서는 [이 포스트](https://jasonkang14.github.io/sql/database-indexing)에 기록한 적이 있다. 

인덱스 사용은 query 성능을 개선할 뿐 실제 데이터에는 아무런 영향을 주지 않는다. 반면에 데이터를 추가로 어딘가에 저장해야 하기 때문에 오버헤드가 발생한다. 특히 write를 할 때, 인덱스 테이블을 전반적으로 rewrite해야 하기 때문에, 위에 작성한 `db_set()`보다는 효율이 떨어질 수밖에 없다. 

이처럼 인덱스 사용은 read에는 좋지만 write에는 불리하다. 따라서 데이터베이스는 자동으로 인덱싱을 하지 않는다. 

------
데이터에 인덱싱을 적용하는 것은 개발자의 몫이다
------

개발중인 어플리케이션의 쿼리패턴을 아는 개발자가, write할 때 발생하는 오버헤드를 최소화 하면서 효율적으로 데이터를 read하기 위해 인덱싱을 적용하는 것이다. 

### Hash Indexes

key-value의 인덱스부터 시작해보자. key-value 데이터베이스는 hash map(hash table)으로 구현된 파이썬의 딕셔너리와 매우 유사하다. in-memory 데이터베이스에 hash map이 있기 때문에, 디스크에 인덱스를 적용하기 위한 목적으로 사용할 수 있다. 

데이터 스토리지가 append만 지원한다고 가정하자. 그렇다면 가장 간단한 인덱싱은 key가 매핑된 바이트의 offset을 hash map의 형태로 저장하는 것이다. 파일에 새로운 key-value 페어를 추가하면, 새로 추가된 데이터의 offset을 hash map에 저장하면 된다. read할 때는 hash map에서 offset을 찾는 방식으로 빠르게 값을 읽어낼 수 있다. 

hash map을 메모리에서 들고있기 때문에, 실제 메모리 크기보다 더 많은 데이터를 저장할 수 있다. 인덱싱이 되어있어서 디스크에서 값을 읽어올 때도 한번만 접근하면 되기 때문이다. 만약 read할 데이터가 이미 파일시스템 캐시에 있다면, 디스크에 접근하지 않고도 read가 가능하다. 

하지만 계속 append해서 디스크가 가득 차면 어떻게 할까? 이를 방지하기 위해서 hash map이 특정 사이즈가 되면, 해당 hash map에 write를 멈추고, 새로운 hash map에 write를 시작하면 된다. 그리고 중복되는 key를 제거하는 `compaction`을 통해 공간을 최적화 할 수 있다. 

compaction이 hash map의 사이즈를 축소할 수 있기 때문에, 여러 hash map들을 하나로 합칠수도 있다. 작성된 hash map은 re-write가 불가능하기 때문에, merge된 hash map은 새로운 hash map에 write 된다. 더이상 write가 되지 않는 hash map들에 대한 compaction은 background thread에서도 가능하다. background thread에서 작업이 되기 때문에, 지속적으로 다른 곳에 write가 가능하고, merge가 끝나면 해당 hash map으로 read request를 보낼 수 있다. 

인덱싱 외에도 로깅을 할 때 다양한 것들을 고려해야 한다. 

1. File Format - csv보다는 binary 포맷이 좋다. 
2. Deleting records - key와 관련된 값을 지울 때, deletion record를 데이터 파일에 추가해야한다. 특히 merge후에는 해당 값에 더이상 접근하면 안되기 때문에 deleting record를 추가해서 해당 값에 접근하지 않도록 막아야한다. 
3. Crash recovery - 데이터베이스를 재시작하면, 메모리에 있는 hash map은 날아갈 수밖에 없다. 다시 생성하면 되겠지만, 시간이 오래 걸리기 때문에 hash map을 종종 디스크에 저장하는 방식을 취할 수 있다. 
4. Partially written records - write를 하는 중간에 데이터베이스가 crash하는 경우에는 checksum을 활용해서 corrupt된 로그는 무시할 수 있다. 
5. Concurrency control - 로그는 sequential하게 작성되어야 하기 때문에, 동시성을 보장해야 한다. 

삭제하지 않고 계속 append만 하는 것이 낭비라고 생각할 수 있지만, write하는 속도 측면에서 많은 이점이 있고, 동시성 보장에도 훨씬 유리하다. 

이런 장점들이 있지만, 용량이 너무 커져서 메모리가 다 차면 문제가 발생할 수 있고, range query에 불리하다는 단점이 있다. 

### SSTables and LSM-Trees

그동안 설명했던 것은 계속 뒤에 append만 하지만, 이번에는 key 순서대로 정렬된 시퀀스에 대해 생각해보자. 정렬을 하기 때문에 sequential write가 불가능할 것처럼 보이지만 그렇지 않다. 이러한 구조를 `Sorted String Table` 또는 `SSTable`이라고 부른다. merge는 기존과 유사하게 한번만 나타난다. 

SSTable은 hash index와 비교했을 때 많은 장점이 있다. 

1. merge가 매우 빠르다. 일단 key 순으로 정렬이 되어있기 때문에, segment를 나란히 놓고 mergesort를 실행하는 방식이다. 각 segment에서 가장 앞에있는 key들을 서로 비교해서 merge한다. 만약 여러 segment에 같은 key를 가진 값이 있다고 하면, 가장 최근에 작성된 segment에 있는 key만 남긴다. 
2. 파일에서 특정 key를 찾을 때 인덱스에 의존하지 않아도 된다. sort가 되어있기 때문에 binary search느낌으로 찾아가면 된다. 메모리에 특정 인덱스를 기록한 값이 있긴 해야하지만, 많은 값을 가진 디테일한 인덱스는 필요 없다. 
3. 기본이 range scan이기 때문에, 여러 record들을 block으로 묶어서 압축해서 write가 가능하다. 용량도 아끼고, i/o bandwidth도 줄일 수 있다.

##### Constructing and maintaining SSTables

write가 언제 일어날지 모르는데 어떻게 key를 기준으로 sort할 수 있을까? 

tree구조를 사용해서 디스크에 sort해서 저장하는 것도 가능하고, memory에서 sort하는 것은 더 간단하다. write요청이 들어오면 메모리에 B-Tree에 데이터를 추가한다(memtable). memtable이 너무 커지면, 디스크의 SStable로 write한다. tree가 이미 key로 sort되어있기 때문에, write는 상당히 빠르다. 이 SSTable이 새로운 segment가 되고, memtable이 SSTable로 write되는 동안, 새로운 memtable에 계속 write하면 된다. 

read를 처리하기 위해서는, memtable에서 먼저 key를 찾고, 가장 최근에 disk에 기록된 segment에서 찾고, 점점 더 오래된 segment로 올라간다. 그리고 가끔씩 compaction을 통해서 merge해주면 된다. 

이것은 잘 작동하긴 하는데, 데이터베이스가 터지면 가장 최근의 write를 잃어버린다는 문제가 있다. memtable에 write된것을 아직 disk로 옮기지 못했기 때문이다. 이것을 대비해서 write가 일어날 때마다 disk에 따로 log를 기록하는 방법이 있다. 

이 log는 sort되지 않았지만, 

1. memtable에만 기록되고 데이터베이스가 터지는 경우를 방지하는 목적이고
2. memtable에서 disk로 기록될 때마다 날리기 때문에 

sort가 되지 않은 것은 문제되지 않는다. 

##### Making an LSM-tree out of SSTables

이전 섹션에서 언급된 방식은 LevelDB, RocksDB에서 사용되는데, Cassandra나 HBase에서 사용되는 방식이다. 이런식으로 merge되고 compact되는 스토리지 엔진을 LSM storage engine이라고 부른다. 

ElasticSearch와 Solr의 full-text search에 사용되는 Lucene이라는 인덱싱 엔진은, sorting을 위해 term dictionary라는 방식을 사용한다. full-text는 key-value 인덱스보다 더 복잡하지만, 같은 아이디어에서 출발한다. 쿼리를 위해 단어가 주어졌을 때, 해당 단어를 갖고있는 document를 찾는 것이다. 넘겨지는 단어가 key이고, 해당 key를 가진 document들을 list형식으로 value에 저장한다고 보면 된다. 

##### Performance optimizations

LSMTree는 데이터베이스에 존재하지 않는 key를 찾을 때 매우 느리다. 해당 key가 존재하지 않는 다는 것을 확신하기 위해 memtable을 훑고, segment를 다 찾아봐야하기 때문이다. 이를 위해 스토리지 엔진은 Bloom filter를 사용한다. Bloom filter는 set의 형식으로 데이터를 메모리에 저장한다. 메모리에서 key의 유무를 판단하기 때문에 불필요하게 disk를 접근할 필요가 없다. 

SSTable의 merge와 compact 효율을 계산하기 위해 size-tiered와 leveled compaction을 사용한다. LevelDB, RocksDB는 level compaction, HBase는 size-tiered, Cassandra는 둘 다 지원한다. 

size-tiered compaction에서는, 작고 새로운 SSTable 더 오래된 테이블로 merge된다. level compaction에서는 key의 range가 더 작은 SSTable로 쪼개지고, 오래된 데이터는 별도의 level로 옮겨서 compaction을 조금씩 진행하고, disk를 적게 사용하게 된다. 

이러한 LSMTree의 방식은 데이터셋이 메모리보다 훨씬 큰 경우에도 지속적으로 작동할 수 있다. 데이터가 sort되어서 저장되었기 때문에, range query도 효율적으로 처리할 수 있고, write도 상당히 높은 throughput으로 가능하다.

[Column-Oriented Storage에 관한글](https://jasonkang14.github.io/database/column-oriented-storage)로 이어집니다.