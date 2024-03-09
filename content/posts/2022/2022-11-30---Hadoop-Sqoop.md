---
title: 하둡 Sqoop
date: "2022-11-30T19:35:37.121Z"
template: "post"
draft: false
slug: "/hadoop/what-is-sqoop"
category: "Hadoop"
tags:
  - "Hadoop"

description: "Hadoop에서 sqoop 역할"
---

`Sqoop`은 사용자들이 다른 데이터베이스에서 하둡으로 데이터를 가져올 때 사용하는 툴이다. 또한 하둡에서 데이터 처리가 끝나면 데이터를 불러왔던 데이터베이스로 전처리/후처리 된 데이터를 다시 보낼 수도 있다.

# Getting Sqoop

`Sqoop`을 구동하는 방법은 여러가지인데, 근본은 Apache이다. `Sqoop`의 source code와 문서를 모두 가지고 있다.

Apache에서 `Sqoop`을 설치하면 `/home/name/sqoop-x.y.z` 디렉토리에 설치되고, 이 위치를 `$SQOOP_HOME`이라고 부른다. `Sqoop`은 `$SQOOP_HOME/bin/sqoop` 명령어로 실행된다 Sqoop1의 단점을 보완하기 위해 Sqoop2도 출시됐는데, Sqoop2는 Java API, CLI, web UI, REST API를 제공한다.

# Sqoop Connectors

Sqoop은 데이터를 import/export할 수 있는 extension이 있다. bulk data transfer가 가능한 storage system이라면 모두 Sqoop과 연동할 수 있다. `connector`은 다른 storage system과 연결을 가능하게하는 모듈이다. MySQL, PostgreSQL, Oracle, SQL Server, DB2, Netezza와 연결이 가능하고, JDBC protocol또한 지원한다.

built-in connector이외에 다양한 third-party 툴도 존재한다. 따라서 Sqoop은 Teradata, NoSQL등과도 연결할 수 있다.

# A Sample Import

MySQL에 데이터베이스와 테이블을 생성하고, 그 안에 row들을 추가한다. 그리고 MySQL Connector와 같이 사용될 JDBC driver를 설치하고 sqoop의 classpath에 추가한다.

```bash
sqoop import --connect jdbc:mysql://MYSQL_HOST/DATABASE_NAME --table TABLE_NAME -m 1
```

위 명령어를 통해 `Sqoop`의 import tool이 `MapReduce` job을 실행해서 MySQL 데이터베이스와 테이블 연결한다. default는 4개의 map task를 병렬로 돌려서 효율일 높힌다. 각각의 task는 import된 결과를 같은 디렉토리 내의 다른 파일들로 write한다. -m 뒤에 숫자는 파일의 갯수를 나타내는데, 예제에서 row가 3개밖에 없기 때문에 굳이 여러개의 파일로 나누지 않고 하나의 파일에 write하도록 설정한듯 하다.

아래 명령어를 통해 파일을 확인할 수 있다.

```bash
hadoop fs -cat widgets/part-m-00000
```

Sqoop은 CSV와 유사한 형태의 textfile을 생성한다. 값을 구분하는 Delimiter도 설정을 통해 지정할 수 있다.

### Text and Binary File Formats

`Sqoop`은 import한 데이터를 다양한 포맷으로 저장할 수 있다. textfile이 기본값이지만 textfile은 binary field를 갖지 못하고, `null`과 `"null"`을 구분하지 못하는 단점이 있다. 이것을 해결하기 위해 `SequenceFiles`, `Avro`, `Parquet` 파일들을 지원한다. 이 3가지 포맷은 import된 데이터를 가장 잘 나타내고, 압축도 가능하게 하기 때문에 MapReduce job을 병렬 처리하는데도 효율적이다. 하지만 지금 버전에서는 `Avro`나 `SequenceFiles`들을 Hive로 바로 올릴 수 없다는 제한사항이 있다. 그리고 `SequenceFiles`는 java에서만 쓸 수 있다는 단점이 있다.

# Generated Code

`Sqoop`은 데이터를 import할 때 java로 작성된 source file(`TABLE_NAME.java`)도 제공한다. 이 파일에 있는 코드를 사용해서 테이블에서 가져온 데이터를 deserialize 할 수 있다. 또한 `MapReduce` job을 실행할 수도 있고, `SequenceFiles` 형태로 변환해서 `HDFS`에 저장할수도 있다. 값을 불러올 때마다 같은 파일명의 java 파일이 생성되기 때문에, 이를 대비해서 `codegen`명령어를 사용해서 생성되는 파일의 class 이름을 변경할 수도 있다. 최신 버전들은 `Avro` 기반의 serialization과 schema generation을 지원한다. 이런 경우에는 별도의 코드를 사용하지 않고도 sqoop을 사용할 수 있다.

# Imports: A Deeper Look

`Sqoop`이 `MapReduce`를 사용해서 테이블의 row를 read하는 과정에 대해 조금 더 상세히 이야기 해본다.

`Sqoop`이 java로 작성되었기 때문에, JDBC를 사용해서 RDBMS에 접근한다. 대부분의 데이터베이스는 JDBC가 접근할 수 있는 driver를 제공한다. import가 시작되기 전에, `Sqoop`은 JDBC를 사용해서 테이블을 확인한다. SQL data type의 컬럼 정보들을 가져오고, SQL column의 data type을 java의 type으로 매핑한다. 매핑된 값들은 `MapReduce`에서 field value로 사용된다. `Sqoop`의 code generator는 이 정보를 사용해서 table-specific한 class를 만들어서, 테이블에서 가져온 데이터를 기록한다.

![sqoop-import](https://i.imgur.com/F0tZa0g.png)

JDBC의 `ResultSet` interface의 `readFileds()` method가 쿼리 기록을 읽어온다(import). 또한 `write()` method를 사용해서 `Sqoop`이 MySQL 테이블로 row를 추가할 수도 있다(export).

import 중 `MapReduce` job은 JDBC를 통해 읽어온 테이블의 섹션들을 `InputFormat`을 사용해서 구동된다. 효율을 높이기 위해서 각 쿼리들을 primary key를 기준으로 여러 node들로 분산한다.

모든 컬럼을 read할 필요가 없는 경우에는 원하는 column만 read할 수도 있다. 또한 query의 WHERE 절도 지정해서 원하는 데이터만 가져올 수도 있다.

`HDFS`로 데이터를 가져올 때, source data의 consistency를 유지하는 것이 중요하다. 따라서 import중에는 해당 table이 update되지 않도록 막는 것이 중요하다.

데이터를 주기적으로 가져오는 경우도 있을텐데, 이런 경우에는 새로 추가된 데이터만 가져올 수 있도록 확인하는 절차가 필요하다. 추가로 가져온 데이터는 `append`하는 형식으로 `HDFS`에 추가된다.

`Sqoop`은 데이터를 가져올 때 다양한 전략을 사용할 수 있지만, 그중에 특정 소스에 대해 특화된 방법들이 있다. 예를들면 MySQL에서는 `mysqldump`가 JDBC를 사용하는 것보다 더 빠른데, 이와 비슷한 경우라고 볼 수 있다.

이것을 `Direct-Mode Import`라고 부른다. `--direct` argument를 추가해서 구동되어야 한다. 하지만 단점도 있는데, `MySQL`을 예로 들면, `CLOB`, `BLOB`같은 사이즈가 큰 데이터를 처리할 수 없다.

# Working with Imported Data

데이터를 import한 후에는 delimiterer를 파악하고, 데이터를 처리해야 한다. 또한 text file에 있는 데이터들의 type을 매핑해서 변환시켜줘야한다. 예를들면 `"1"`은 string이 아니라 int로 처리될 수 있도록. 다행히 `Sqoop`을 통해 가져온 데이터는 이런 작업들을 자동으로 처리해준다.

`Avro`를 사용해서 가져온 파일들은 `Generic Avro Mapping`이 이루어지기 때문에 `MapReduce`가 schema-specific한 code를 사용하지 않고도 작업을 처리할 수 있다.

### Imported Data and Hive

`Hive`를 쓰면 효율이 개선된다. 관계형 자료형에서 가져온 데이터의 경우 특히 그렇다. `Hive`를 사용하면 다른 테이블에서 불러온 데이터들을 혼합해서 분석할 수도 있고, 더 효율적으로 `MapReduce` job들을 처리할수도 있다. 일반적으로 아래와 같은 순서로 진행된다

`HDFS`로 data import -> `Hive` table 생성 -> `HDFS` 데이터를 `Hive`로 load

### Importing Large Objects

large object를 불러오는 것은 시간이 오래 걸리기 때문에, 일반적으로 각각의 row의 외부에 저장되어 있다. 이러한 large object들에 접근하기 위해서는 해당 object를 reference하는 row에서 `opening`을 열어줘야 한다.

![accessing-large-objects](https://i.imgur.com/3yXDRmU.png)

일반 데이터베이스에서 large object들을 저장하는 과정이 복잡하기 때문에, 더더욱 hadoop을 사용하는 것이 좋다. `Sqoop`은 저런 large object들을 불러와서 hadoop에 저장하고, 추가 처리가 가능하도록 한다.

`MapReduce`는 이러한 large object들을 처리하기 위해 `materialize`라는 과정을 거쳐야 한다. 이게 오래 걸리는 것 같다. 그리고 파일이 너무 크면 메모리에서 `materialize`하는 것이 불가능하다. 따라서 `Sqoop`은 이를 해결하기 위해서 large object들은 `LobFile`이라는 별도의 파일에 따로 저장한다. threshold는 16MB이다.

# Performing an Export

import해서 가져온 데이터 처리가 끝나면 다시 해당 source로 export가 가능하다. delimiterer를 따로 지정해주지 않으면 `unicode 0x0001`이 기본으로 사용된다.

export는 import와 비슷하다. database connect string에 따라 적합한 strategy를 선택한다(거의 JDBC). JDBC를 사용하는 target table에 따라 적절한 Java Class를 생성한다. 생성된 class는 textfile로부터 record를 parsing해서, 테이블에 적합한 타입으로 변경한다. 그러면 `MapReduce` job이 source를 불러오고, java class를 사용해서 변환한 다음에, export하는 방식으로 진행된다.

![sqoop-data-export](https://i.imgur.com/ngHykhI.png)

JDBC방식은 INSERT statement를 batch job으로 처리한다. 엄청 많은 single row insert보다는 적당한 수의 multile rows insert가 효율적이다. 사용자는 병렬 처리할 정도를 argument로 처리해서 export의 효율을 향상시킬 수 있다.

### Exports and Transactionality

병렬처리 하기 때문에 export는 atomic하지 않은 경우가 많다. 따라서 insert가 됐는데 관련된 정보가 비어있다거나 하는 경우가 있을 수 있다. `Sqoop`은 메모리 관리를 위해 한번에 수천개의 row까지만 insert한다. 이것을 해결하기 위해서 temporary staging table을 export할 수도 있다. transaction이 보장된 staging table에 export가 끝나면, 이 테이블을 실제 테이블로 write한다.
