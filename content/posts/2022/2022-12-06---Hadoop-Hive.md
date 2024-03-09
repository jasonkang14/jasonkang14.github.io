---
title: 하둡 Hive
date: "2022-12-06T19:35:37.121Z"
template: "post"
draft: false
slug: "/hadoop/what-is-hive"
category: "Hadoop"
tags:
  - "Hadoop"

description: "Hadoop에서 Hive의 역할"
---

`Hive`는 Facebook(현 Meta)에서 SQL query에 능숙한 엔지니어들이 `HDFS`에 저장된 데이터를 사용하기 위해 만들어졌다.

# Installing Hive

`Hive`는 SQL query를 Hadoop 클러스터에서 구동되는 job들로 변경한다. 데이터를 테이블 형태로 저장하고, 테이블 스키마 등의 메타데이터는 `metastore`라는 곳에 저장한다. 일반적으로 metastore는 로컬에서 실행해서, 다른 사람들과 공유되지 않는다. `Hive` 설치는 hadoop 버전을 고려해서 맞는걸로 잘 설치하면 된다.

### The Hive Shell

`Hive`는 MySQL과 매우 유사한 `HiveQL`이라는 쿼리 언어를 사용하는데, `HiveQL`이 hive shell에서 돌아간다.

# An Example

1. `Hive`는 managed storage에서 데이터를 불러온다. 예제에서는 local file system을 storage로 사용한다.

2. 데이터를 table 단위로 분류하고, `CREATE TABLE` 명령어를 통해 테이블을 생성한다. 테이블을 생성할 때는 각 컬럼(`HiveQL`의 필드)의 타입을 지정해줘야한다

```sql
CREATE TABLE records (year STRING, temperature INT, quality INT)
ROW FORMAT DELIMITED
  FIELDS TERMINATED BY '\t';
```

일반적인 SQL과 다른점은 `ROW FORMAT`이 추가된다는 점이다. 위 예제에서는 각각의 row는 `tab`으로 분리된다는 뜻이다. 각 field는 tab으로 나뉘어져있고, 각 row는 new line으로 분리된다.

3. 로컬 파일 시스템에서 데이터를 Hive로 올린다

```sql
LOAD DATA LOCAL INPATH 'LOCAL_PATH'
OVERWRITE INTO TABLE records;
```

`LOAD DATA LOCAL`로 업로드 된 파일은 하이브 디렉토리에서도 찾을 수 있다. 예제에서는 파일 한 개만 사용하지만, 파일이 여러개인 경우 Hive가 쿼리할 때 모든 파일들을 다 read한다.

그리고 `OVERWRITE` 명령어를 실행할 때 테이블로 하이브 디렉토리에서 테이블로 데이터를 옮기고, 하이브 디렉토리에 있는 파일은 삭제한다.

4. `SELECT`등의 쿼리를 Hive가 Hadoop job으로 변경한다.

# Running Hive

### Configuring Hive

하둡과 유사하게 xml을 사용한다. 하이브는 `MapReduce`를 execution engine으로 사용하기 위해 만들어졌는데, `Tez`나 `Spark`를 사용할 수도 있다. udemy강의를 돌이켜보자면 Tez가 MapReduce보다 훨씬 빨랐던 것 같다.
하이브 사용 로그는 로컬 파일시스템에 저장된다.

### Hive Services

shell이외에 다양한 기능들이 있다.

1. cli: command line interface
2. hiveserver2: Thrift service를 expose하는 서버. 컴퓨터 언어에 무관하게 접근할 수 있다.
3. beeline: embedded mode에서 사용되는 cli
4. hwi: hive web interface
5. jar: java application을 구동할 수 있는 툴
6. metastore: 위에서 언급한 메타데이터를 저장하는 곳

### Hive clients

1. Thrift: hive server를 사용하면 Thrift를 사용해서 접근 가능
2. JDBC: Java로 접근
3. ODBC: Thrift를 사용해서 hive에 접근

![hive-client](https://i.imgur.com/yYqgWNi.png)

### The Metastore

hive의 metadata를 저장하는 곳. service와 backing store로 나누어져 있다.

metastore의 종류에는 3가지가 있다.

1. embedded metastore

   - hive와 같은 JVM에서 돌아간다
   - 로컬 디스크에 embedded derby database를 가지고 있다.
   - 한번에 하나의 하이브 세션만 가동할 수 있다는 단점이 있다.

2. local metastore

   - standalone database를 사용해서 여러개의 하이브 세션을 동시에 돌릴 수 있다
   - metastore가 하이브와 같이 돌고 있기 때문에 local이라고 부른다
   - 데이터베이스만 외부에 있는 것
   - 데이터베이스는 MySQL을 많이 씀

3. remote metastore
   - 하나 또는 복수의 metastore 서버를 hive service와 다른 곳에서 돌린다
   - 관리에도 편하고, 보안에도 유리하다

![hive-metastore](https://i.imgur.com/Uit50uz.png)

# Comparision with Traditional Databases

SQL과 매우 비슷하지만 `HDFS`와 `MapReduce`를 고려해서 만들어졌기 때문에 아키텍쳐가 조금 다르다

### Schema on Read vs Schema on Write

기존 데이터베이스에서는 data load time에 schema가 필요하다. 불러온 데이터가 schema에 맞지 않는다면, 거절된다. 이걸 schema on write라고 부르는데, 이는 데이터가 write될 때 schema를 검증하기 때문이다.

하지만 하이브에서는 데이터가 load될 때 검증하지 않고, query를 날릴 때 schema를 확인한다. 이것을 schema on read라고 부른다.

schema on read는 initial load가 빠르다는 장점이 있다. 왜냐면 데이터의 내부 포맷에서 read, parse, serialize하는 과정이 빠지기 때문이다. load operation은 단순히 파일을 복사하거나 옮기는 과정이다. 따라서 조금 더 유연하다.

schema on write는 쿼리가 빠르다는 장점이 있다. 데이터베이스는 컬럼에 인덱싱을 적용할 수 있고, 데이터를 더 빠르게 비교할 수 있다. 하지만 schema on read와 다르게 로딩이 오래 걸린다.

### Updates, Transactions, and Indexes

하이브는 MapReduce를 사용해서 HDFS에 있는 데이터를 처리하기 위해 만들어졌기 때문이다. 따라서 테이블 업데이트는 MapReduce 후 새로운 테이블에 데이터를 생성하는 식으로 이루어진다.

하이브를 사용해서 많은 수의 row를 bulk로 테이블에 추가할 수 있고, UPDATE나 DELETE 또한 가능하다. HDFS in-place file update를 지원하지 않기 때문에 insert, update, delete가 발생하면 작은 delta files에 저장한다. 이 delta file들은 주기적으로 MapReduce job을 통해 테이블로 merge된다. 해당 MapReduce job들은 metastore에서 돌아간다.

하이브는 table, partition별 locking도 지원한다. lock을 사용해서 누군가가 read중인 테이블을 다른 누군가가 drop하는 등의 행동을 막는다. ZooKeeper를 사용해서 lock을 관리하기 때문에 사용자는 lock을 acquire하거나 release하지 않아도 된다.

하이브 인덱스를 사용해서 쿼리 효율을 향상시킬 수도 있다. 두가지 인덱스 타입이 있는데 `compact`와 `bitmap`이다. compact는 HDFS block을 인덱싱한다. 따라서 디스크에서 큰 공간을 차지하지 않으면서 매우 효율적이다. bitmap은 압축된 bitset을 사용하는데 일반적으로 cardinality가 낮은 컬럼에 적용된다.

### SQL-on-Hadoop Alternatives

Impala도 Hive처럼 쓸 수 있다. 하이브보다 빠르다. 임팔라는 클러스터 안에서 돌아가는 각 data node에서 돌아가는 대몬을 사용한다. 클라이언트가 쿼리를 날리면, 임팔라 대몬이 구동되는 노드와 소통하고, 이 노드가 쿼리를 전달한다. 임팔라는 해당 쿼리가 적용되어야 하는 다른 노드들에게 쿼리를 던지고, 마지막에 결과를 합쳐서 클라이언트에게 전달한다.

Presto, Apache Drill, Spark SQL, Apache Phoenix등도 하이브와 유사하다

# Tables

하이브에 테이블을 만들면, 하이브가 데이터를 `warehouse directory`로 옮긴다(`managed table`). 이게 싫다면 `external table`을 만들어서 하이브가 warehouse directory 밖의 데이터를 참조하도록 설정할 수 있다.

데이터를 managed table로 load하는 것은 빠르다. 그냥 옮기거나 파일시스템 내부에서 이름만 바꾸는 작업이다. 하지만 하이브는 그 테이블이 스키마에 맞는지 확인하지 않는다. 만약 문제가 있으면 쿼리할 때 발견된다. 테이블을 drop하면, 데이터와 메타데이터가 모두 삭제된다.

external table의 경우에는 데이터의 위치를 테이블을 생성할 때 지정해줘야한다. 테이블을 드랍하면 하이브는 데이터는 놔두고 메타데이터만 삭제한다.

### Partitions and Buckets

하이브는 테이블을 `partition`으로 관리한다. partition을 사용하면 쿼리를 더 빠르게 할 수 있다. 테이블과 파티션은 `bucket`으로 나눠질 수도 있는데 데이터를 구조화하고 쿼리를 더 빠르게 할 수 있다.

### Storage Formats

`row`와 `file` 포맷으로 나누어진다.

default storage format은 delimited text이다. 테이블을 row format으로 생성하는 경우에 해당한다. tab은 아니고 ctrl-A이다. 탭보다 텍스트에 들어갈 가능성이 적기 때문에 선택됐다고 한다. collection item delimiter는 ctrl-B이다. array, struct, map의 key-value pair를 구분할 때 사용된다. map key는 ctrl-C 로 구분되고, row들은 new line으로 구분된다.

binary storage format은 sequence files, avro, parquet, RCFiles, ORCFiles등이 사용된다.

### Importing Data

Insert

```sql
INSERT OVERWRITE TABLE target
SELECT col1, col2
 FROM source;
```

파티션을 지정해줄 수 있다.

```sql
INSERT OVERWRITE TABLE target
PARTITION (dt='2001-01-01')
SELECT col1, col2
 FROM source;
```

`overwrite`를 사용해서 기존 데이터를 날려버리고 새로운 것들로 추가할 수 있다.
dynamic partition insert라고 부른다.

```sql
INSERT OVERWRITE TABLE target
PARTITION (dt)
SELECT col1, col2, dt
 FROM source;
```

동시에 여러 테이블에도 insert할 수 있다.

```sql
FROM records2
INSERT OVERWRITE TABLE stations_by_year
 SELECT year, COUNT(DISTINCT station)
 GROUP BY year
INSERT OVERWRITE TABLE records_by_year
 SELECT year, COUNT(1)
 GROUP BY year
INSERT OVERWRITE TABLE good_records_by_year
 SELECT year, COUNT(1)
 WHERE temperature != 9999 AND quality IN (0, 1, 4, 5, 9)
 GROUP BY year;
```

### Altering Tables

table의 이름을 바꾸거나 컬럼을 추가할 수 있다.

```sql
ALTER TABLE source RENAME TO target;
ALTER TABLE target ADD COLUMNS (col3 STRING);
```

### Dropping Tables

```sql
TRUNCATE TABLE my_table;
```

external table에는 적용할 수 없다. 위에서 언급한 것 처럼 external table의 경우에는 metadata만 삭제되고, 외부 테이블의 실제 데이터는 삭제되지 않는다. reference를 제거한다고 보는게 맞는듯.

# Querying Data

`sort by`라는 구절을 사용해서 sorting이 가능하고. 또한 reducer를 지정하기 위해 `distribute by`를 사용할 수도 있다. 하이브가 HDFS + MapReduce를 위해 사용되기 때문에, MapReduce script를 사용할 수도 있다.

SQL과 유사하게 inner join, outer join이 가능하고, subquery를 활용한 semi join도 가능하다.

# User-Defined Functions

Hive가 Java로 되어있어서, Java로만 작성 가능하다. 다른 언어를 사욯안다면 `SELECT TRANSFORM` query를 사용해서, 데이터를 user-defined script를 통해 streaming할 수 있다. UDF에는 3가지 종류가 있다.

1. UDF - input과 output이 모두 row 1개
2. UDAF - 여러개의 row들을 input해서 output은 row 1개
3. UDTF - 하나의 row를 input해서 여러개의 row들이 output으로 나옴
