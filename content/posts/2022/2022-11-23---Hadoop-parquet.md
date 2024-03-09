---
title: 하둡 Parquet
date: "2022-11-23T19:35:37.121Z"
template: "post"
draft: false
slug: "/hadoop/what-is-parquet"
category: "Hadoop"
tags:
  - "Hadoop"

description: "Hadoop에서 parquet의 역할"
---

`parquet`은 nested data를 효율적으로 저장할 수 있게 하는 `columnar storage format`이다. column-oriented database의 일종인 것같다. 일반적으로 파일 크기와, 쿼리 속도면에서 훨씬 효율적이다. 파일 크기는 row-oriented storage보다 통상적으로 작지만, 작은 파일들이 가까이 저장되기 때문에 더 효율적인 인코딩이 가능하다. 쿼리 효율이 빨라지는 이유는 필요 없는 column은 바로 건너뛸 수 있기 때문이다.

`Parquet`의 강점은 deeply nested된 데이터를 column으로 저장하는 것에 있다. flat column을 사용해서 아주 작은 오버헤드를 가진채로 저장한다. flat column을 사용하면 Nested data도 다른 필드와 독립적으로 불러올 수 있기 때문에 쿼리 퍼포먼스가 훨씬 좋아진다--디펜던시가 줄어든다는 느낌인 것 같음.

`Parquet`의 또 다른 강점은 많은 툴들이 `parquet` format을 지원한다는 것이다. `parquet`은 기존에 있는 데이터에 쉽게 적용할 수 있도록 디자인 되었기 때문에, `avro`와 유사하게 언어와 무관한 포맷으로 관리되어 read/write 할 수 있다. 책에서 소개되는 `MapReduce`, `Pig`, `Hive`, `Spark`등등 모두 `Parquet` 포맷을 지원하고있다. `parquet`이 제공하는 자유도는, Java이외에 다른 포맷으로도 데이터를 read/write할 수 있게 해준다.

# Data Model

![parquet-primitive-type](https://i.imgur.com/nQi0D3u.png)

`parquet`파일에 저장된 데이터는 schema를 가진다. 파일의 root에 field들에 대한 정보를 가지고 있고, 각각의 field는 `repitition`, `type`, `name`이라는 값을 가진다.

```javascript
message WeatherRecord {
  required int32 year;
  required int32 temperature;
  required binary stationId (UTF8);
}
```

원시타입에 문자열이 없는데, `parquet`는 문자열 대신에 원시 타입이 어떻게 번역되어야 하는지 정보를 가진 logical type을 정의한다. serialized representation이 원시타입이고, application specific한 정보가 logical type이라고 한다. 어차피 다른 언어에서 사용되면 다양한 포맷이 필요하니 굳이 `string`이라는 타입을 정의하지 않는다는 뜻인 것 같다. 위 예시에서 string은 binary, utf-8이라고 나타내고 있다.

![parquet-logical-type](https://i.imgur.com/60ZJU2I.png)

`complex type`은 `group type`이라는 것을 사용해서 만들어지는데, nested된 데이터에 Layer를 추가하게 된다. 별도의 annotation이 없으면 group들은 다 nested된 데이터라고 보면 된다.

`lists`, `maps`는 `two-level` group struture로 만들어져 있는데, 위에서 보다시피 `list`는 `(LIST)`가 nested되어있고, `map`은 `key-value` pair들이 nested되어있다.

### Nested Encoding

`column-oriented`로 저장될 때, column값들은 같이 저장된다. nested된 값이나 반복되는 값이 없는 flat table의 경우에는 특히 더 심플하다. 하지만 데이터는 일반적으로 nested되어있고, 이런 경우에는 nesting 자체도 인코딩 되어야 하기 때문에 조금 더 복잡하다. 상당수의 columnar format이 flattening을 통해 이 문제를 피해간다. flattening하면 최상단의 컬럼만 column-major하게 저장된다. 하지만 nested column들을 가진 map의 경우에는 `interleave` 되어있어서 key를 읽을 때 항상 value도 같이 읽어들여야 한다. key는 key끼리 value는 value끼리 저장할 수 없다는 뜻인 것 같다. 하지만 `parquet`는 flattening을 하지 않기 때문에 `key`별로 `value`별로 따로 읽어들일 수 있다는 장점이 있다.

`parquet`에서는 원시타입은 타입마다 각각 다른 컬럼에 저장된다. 그리고 write된 값들은 두 개의 정수를 인코딩한 값으로 저장되는데, 하나는 definition level이고 다른 하나는 repetition level이다. repition level은 복제를 몇 번 하느냐 같은데 definition level이 뭔지는 좀 더 찾아봐야한다.

# Parquet File Format

`parquet` header는 header-block-footer로 이루어져 있다.
`header`는 4byte `PAR1` 만 들어있는데, 이 파일이 `parquet` format이라는 것을 나타내고, 파일의 모든 메타데이터는 `footer`에 저장된다. 메타데이터는 포맷 버전, 스키마, extra key-value pair와 파일의 모든 block의 메타데이터 정보를 포함한다. footer의 마지막 두가지 필드는 footer 메타데이터의 길이를 인코딩한 4byte값과 `PAR1` 이다.

메타데이터를 `footer`에 저장하기 때문에, 파일을 read할 때 끝까지 seek행 한다는 단점이 있다. 하지만 이로 인해 `sync marker`가 필요없다는 장점이 있다. footer에 메타데이터가 write되는 시점이 모든 block이 write된 후이기 때문에, writer는 block의 boundary position 정보를 메모리에 저장하고 write가 끝나면 이 정보를 footer에 넘겨줄 수 있다. 따라서 `parquet` 파일들 또한 split이 가능하고, MapReduce job등에서 효율을 향상시킬 수 있다.

`parquet`에서 각각의 block들은 `row group`을 저장하는데, 이 `row group`들은 `column chunk로` 구성되어 있다. 각각의 `column chunk`는 `page`로 구성되어 있다.

하나의 column chunk에서 분산되는 각각의 page들은 압축을 쉽게한다. 값들이 어떻게 인코딩 되느냐에 따라 첫번째 압축 레벨이 정해진다. 가장 간단한 인코딩은 plain encoding인데, 이건 압축 안된다.

`parquet`는 compact encoding도 쓰는데, delta encoding, run-length encoding, dictionary encoding등의 방법들이 있다. 공간을 아끼기 위해 bit packing을 하기도 한다.

파일을 write할 때 `parquet`은 컬럼 타입에 따라 가장 적합한 인코딩 방식을 자동적으로 고른다. 에를 들면 boolean은 run-length encoding + bit packing으로 이루어진다. 대부분의 인코딩은 dictionary encoding을 사용하는데 dictionary encoding이 `dictionary page size`라는 값보다 커지면 plain encoding이 fallback으로 적용된다. 그리고 사용된 인코딩 값은 메타데이터에 저장된다.
snappy, gzip, LZO등을 사용한 추가 압축도 가능하다.

# Parquet Configuration

아래 표를 참고해본다
![parquet-configuration](https://i.imgur.com/ddLfGBD.png)

block size는 scanning 효율과 메모리 사용간 trade-off가 있다. 큰 block size는 scanning에는 좋지만 size가 크기 때문에 메모리를 많이 사용한다. `parquet` 파일의 block 크기는 `HDFS` block크기보다 작아야 한다. 그래야지 `HDFS` block안에 `parquet` 파일들을 저장할 수 있다.

`parquet`의 가장 작은 단위는 `page`인데, 따라서 하나의 row를 읽어들이는 경우 해당 row의 `page`들을 모두 압축 풀고, 디코딩 해줘야 한다. 따라서 row 하나만 read할 때는 page의 크기가 작은 편이 좋다. 그런데 page의 크기가 너무 작으면 나눠서 저장해야 하기 때문에 오버헤드가 발생한다.

Parquet 파일을 read/write하는 것은 책의 예제코드를 참고하도록 한다.
