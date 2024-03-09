---
title: Cassandra DB
date: "2022-07-10T18:21:37.121Z"
template: "post"
draft: false
slug: "/database/cassandra-db"
category: "Database"
tags:
  - "Database"

description: "NoSQL중 하나인 CassandraDB에 대해 알아본다"
---

# TL;DR

### CassandraDB는 consistency를 희생하고 availability와 partition-tolerance에 집중한 NoSQL 데이터베이스이다

[CassandraDB](https://cassandra.apache.org/_/index.html)는 페이스북에서 사용하는 NoSQL이다. 페이스북의 특징과, READ/WRITE가 빠른 NoSQL의 장점과, 분산저장과 availability에 최적화 된 CassandraDB의 특징이 잘 맞기 때문인 것으로 보인다. 

Database를 선정할 때 `consistency`, `availability`, `partition-tolerance`라는 3가지 항목 중 더 중요한 2가지를 고려해서 선정한다고 한다. CassandraDB는 `consistency` 가 부족한 대신, `availability`와 `partition-tolerance`에서 강점을 보인다. 

![CAP Theorem](https://i.imgur.com/AkOPQZi.png)

3가지 특성에 대해 먼저 살펴보자면, `consistency`는 데이터가 일관성이 있어야 한다는 뜻이다. 어떤 node에 접근하던지 상관없이 query를 시도하면 같은 데이터를 받아올 수 있어야 한다는 것을 뜻한다. 

CassandraDB는 `eventual consistency`를 따르는데, 간단하게 설명하자면 database에 write된 정보를 바로 read할 수 없다는 뜻이다. 이는 CassandraDB에 write를 시도하면, 데이터가 하나의 node에 저장되는 것이 아니라 여러 node에 분산저장되기 때문에, 분산저장이 마무리 되기 전까지는 Read할 수 없기 때문이다. 페이스북을 사용해보면, 내가 작성한 포스트가 바로 보이지 않고 새로고침을 하거나 조금 뒤에 보이는 것을 확인할 수 있다. post를 작성하면 어떤 node에는 내가 작성한 포스트 데이터가 있고, 어떤 node에는 없기 때문에 `consistency`가 성립하지 않는 것이다. 이런 부분이 CassandraDB의 특징을 반영한 것이다. 

`Availabililty`는 우리말 그대로 바꾸자면 이용이 가능한지를 뜻하는데, 데이터베이스가 갑자기 고장나지 않고 얼마나 잘 작동하는지를 말한다. 데이터베이스가 얼마나 reliable한지를 뜻하는데, 기술적으로 설명하자면 node하나가 죽더라도 클러스터에 접근할 수 있는지를 뜻한다. 

데이터베이스의 구조상 이런 부분이 어려운 경우가 있다. [Hbase](https://hbase.apache.org/)를 예로들면, Hbase는 HMaster라는 노드가 어떤 노드에 데이터가 있는지에 대한 정보를 가지고 있는데, 이 HMaster가 죽어버리면 HMaster가 다시 살아날 때 까지 클러스터에 접근할 수 없다. 

![Cassandra Architecture](https://i.imgur.com/qV14jtM.png)

이와 달리 CassandraDB는 별도의 master node가 존재하지 않고, 모든 node가 서로 소통하며 데이터를 저장 하기 때문에--여러 node들이 각각 소통한다고 해서 `gossip protocol`이라는 이름을 붙였다고 한다--하나의 node가 죽는다고 하더라도 데이터베이스 운영에 타격을 입지 않는다는 장점이 있다. 

마지막으로 `partition-tolerance`는 partition이 잘 작동하지 않더라도 데이터베이스 운용에는 문제가 없어야 한다는 것을 뜻한다. availability에 대해서 이야기 할 때, master node가 따로 없고 모든 node들이 서로 소통하며 데이터를 저장한다고 했는데, 네트워크 등의 문제로 이 node들 간의 소통이 원활하지 않을 때도 데이터베이스는 잘 작동해야 한다는 뜻이다. node들은 네트워크 상황이 좋아지면 서로와 다시 sync해서 통일성을 갖추는 방식이다. 

CassandraDB는 NoSQL이지만 [Cassandra Query Language(CQL)](https://cassandra.apache.org/doc/latest/cassandra/cql/)로 간단한 쿼리가 가능하다. de-normalized된 데이터--여러곳에 복제되어 데이터가 저장된 경우-에 대해 primary key를 쿼리할 수 있다. 

CassandraDB는 [DataStax](https://www.datastax.com/)와 같이 사용해서 [Spark](https://spark.apache.org/)와 연동할 수 있다.

![Cassandra with Spark](https://i.imgur.com/9uFBWJG.png)

Spark와 연동할 경우 CassandraDB의 table을 DataFrame처럼 사용할 수 있다. [movielens](https://grouplens.org/datasets/movielens/)데이터를 사용해서 Spark와 CassandraDB를 연동한 코드를 첨부한다. 

```python
from pyspark.sql import functions, Row, SparkSession

def parseInput(line):
	fields = line.split('|')
	return Row(user_id = int(fields[0]), age = int(fields[1]), gender = fields[2],\
		occupation = fields[3], zip = fields[4])

if __name__ == '__main__':
  # CassandraDB가 현재 localhost에서 돌고있음
	spark = SparkSession.builder.appName('CassandraIntegration')\
						.config('spark.cassandra.connection.host', '127.0.0.1')\ 
						.getOrCreate()
  
  # 데이터는 HDFS에서 가져온다
	lines = spark.sparkContext.textFile('hdfs://user/maria_dev/ml-100k/u.user')
	users = lines.map(parseInput)
	usersDataset = spark.createDataFrame(users)

	userDataset.write\
		.format('org.apache.spark.sql.cassandra')\
		.mode('append')\
		.options(table='users', keyspace='movielens')\
		.save()

	readUsers = spark.read\
		.format('org.apache.spark.sql.cassandra')\
		.options(table='users', keyspace='movielens')\
		.load()

	readUsers.createOrReplcaeTempView('users')

	sqlDF = spark.sql('SELECT * FROM users WHERE age < 20')
	sqlDF.show()

	spark.stop()
```


참고자료
[udemy 강의](https://www.udemy.com/course/best-hadoop/learn/lecture/28319536#content)
[You Can't Sacrifice Partition Tolerance](https://codahale.com/you-cant-sacrifice-partition-tolerance/)

