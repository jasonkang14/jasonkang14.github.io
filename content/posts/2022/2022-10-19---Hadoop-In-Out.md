---
title: Hadoop I/O
date: "2022-10-19T23:35:37.121Z"
template: "post"
draft: false
slug: "/hadoop/data-in-out"
category: "Hadoop"
tags:
  - "Hadoop"

description: "Hadoop이 데이터를 쓰고 읽는 법"
---

# Data Integrity

하둡처럼 방대한 양의 데이터를 처리할 경우에는, 데이터가 분실되거나 corrupt될 가능성이 높다. 데이터가 corrupt된 것을 탐지하기 위한 가장 일반적인 방법은 `checksum`을 계산하는 것인데, 데이터가 처음 하둡에 들어왔을 때 실시하고, 데이터가 이동하거나 corrupt될 가능성이 있는 경우에 다시 실시한다. 하지만 이 방법은 에러가 있는지 확인하는 절차에 불과하고, corrupt된 데이터를 회복시킬수는 없다. 데이터가 아니라 `checksum`이 corrupt되었을 수도 있지만 이 가능성은 매우 낮다.

`checksum`은 데이터의 authenticity를 판단하는 방법인데 아래와 같은 흐름으로 이루어진다.

![how-checksum-works](https://i.imgur.com/X4NoYCD.png)

일반적으로 32-bit 정수로 이루어진 `checksum`을 compute한다.

### Data Integrity in HDFS

HDFS는 데이터가 write될 때 checksum을 실시하고, 데이터를 read할 때 checksum을 verify한다. checksum의 default 크기는 512bytes이다. `datanode`가 클라이언트로부터 받은 데이터를 저장하거나, 어떤 데이터를 복제할 때마다 checksum을 실시한다. `datanode`가 checksum을 실시하다가 실패하면 클라이언트에게 `IOException`을 리턴하고, 클라이언트는 application상태에 따라 이 exception을 처리한다.

클라이언트가 데이터를 read할 때 checksum을 verify하기도 한다. 각각의 node들은 checksum의 기록을 남겨서 특정 block이 가장 최근에 verify된 시점을 확인한다. 또한 각 `datanode`는 백그라운드 thread의 `DataBlockScanner`를 통해서 주기적으로 block들을 verify한다.

HDFS가 데이터를 replication해서 저장 하기 때문에, 특정 block이 corrupt될 경우에 corrupt되지 않은 replica를 사용해서 해당 block을 복구할 수 있다. 클라이언트가 corrupt된 block을 발견하면, `ChecksumException`이 발생하기 전에 `namenode`로 부터 다른 block을 read할 수 있도록 요청한다. `namenode`는 다른 클라이언트들이 corrupt된 block에 접근할 수 없도록 처리하고, replication level을 유지하기 위해 다른 `datanode`에 block을 복제하고 corrupt된 block은 삭제한다. corrupt된 데이터를 확인하고 싶다면 checksum을 bypass하는 옵션을 추가할 숭 ㅣㅆ다.

### LocalFileSystem

하둡의 `LocalFileSystem`은 클라이언트에서 checksum을 실시한다.

클라이언트가 `filename`이라는 파일을 write할 때, `LocalFileSystem`은 해당 파일의 checksum과 같은 디렉토리에 `.filename.crc`라는 파일을 생성한다. checksum의 chunk size는 `.crc`파일의 metadata에 저장되어 chunk size가 변하더라도 read할 수 있도록 한다. checksum은 해당 파일을 read할 때 verify되고, 만약 문제가 있다면 `ChecksumException`을 return한다.

checksum을 실시하는 비용은 낮은 퍼센트의 오버헤드만 추가될 뿐 높지 않은 편이다. data integrity를 보장하기 위해서라면 이정도는 쓸만하다. 뒷단에 있는 filesystem이 checksum을 지원한다면 `LocalFileSystem`대신 `RawFileSystem`을 사용해서 checksum을 disable할 수도 있다.

### ChecksumFileSystem

`LocalFileSystem`은 `ChecksumFileSystem`을 사용해서 checksum을 실시한다. `ChecksumFileSystem` class를 사용하면 다른 filesystem들에 checksum을 쉽게 도입할 수 있다. 데이터가 corrupt된 경우 `ChecksumFileSystem`이 실패 flag를 만들면, `LocalFileSystem`이 문제가 있는 파일과 그 checksum들을 `bad_files` 라는 디렉토리로 옮긴다.

# Compression

파일 압축은 두가지 장점이 있는데

1. 파일을 저장하는 공간을 줄일 수 있고,
2. 디스크 간 네트워크를 사용해서 데이터 전송을 빠르게 할 수 있다.

압축에 대한 다양한 방법들은 아래 표에 상세히 나와있다
![how-hadoop-compresses-files](https://i.imgur.com/CY8ldEr.png)

위에서 언급한 두 장점은 서로 상쇄하는 문제(?)가 있는데, 압축과 푸는 속도가 빠르다면 공간을 많이 아낄 수 없다. 그리고 공간을 많이 아낀다면 압축하는데 시간이 오래 걸린다.

### Codecs

`codec`은 압축했다가 풀었다가 하는 알고리즘이다. 하둡에서는 `CompressionCodec` interface로 구현되어 있다. 책에는 예제 코드와 함께 여러가지를 설명한다.

### Compression and Input Splits

`MapReduce` job을 위해 데이터를 압축한다면, 압축 알고리즘이 각 block을 split할 수 있는지를 고려해야한다. 예를들면 1GB의 데이터가 있다면, HDFS가 128MB사이즈의 8개 block들로 나눠서 저장한다. `MapReduce`는 이 block들을 다시 8개의 input stream으로 나누어서 mapper에 넣으면 된다. 하지만 해당 데이터를 gzip으로 압축시키면 HDFS는 8개의 block으로 데이터를 저장하겠지만, gzip의 stream의 내의 임의의 포인트에서 데이터를 read할 수 없기 때문에 각각의 block을 split 할 수 없다.

gzip은 `DEFLATE`라는 방식으로 압축된 데이터를 저장하는데, `DEFALTE` 방식은 각 block의 시작점을 구분할 수 없다는 문제점이 있다. 그래서 gzip은 block split을 지원하지 않는다. 이런 경우에 `MapReduce`는 각 block을 split하지 않는다. 하지만 이 경우 locality를 위반한다. 왜냐면 하나의 mapper가 8개의 block을 처리해야 하기 때문인데, 이 block들이 mapper와 같은 위치에 존재할 가능성이 매우 떨어진다. 반면 `LZO`를 사용해서 압축하면 `MapReduce` job을 위해 데이터를 split할 수 있다.

# Serialization

`Serialization`은 object를 byte stream으로 변환하여 네트워크로 전송하거나 persistant storage에 저장할 수 있도록 변환하는 방법이다. `Deserialization`은 반대로 bytestream을 object로 변환한다.

데이터 분산과 처리에 `serialization`이 사용되는 경우는 process간의 소통과 persistant storage이다.

하둡에서는 process간의 소통은 Remote Procedure Calls(RPC)로 구현되었다. RPC 프로토콜은 메세지를 binary stream을 다른 node로 보내기 위해 `serialization`을 사용한다. binary stream을 받은 node는 `deserialization`을 통해 메세지를 확인한다. 따라서 serizliation 포맷은 간결하고, 빠르고, 확장성이 있으며, 상호 작동할 수 있어야한다.

하둡은 자체 `serialization` 포맷인 `Writables`를 사용한다. 매우 간결하고 빠르지만, Java이외의 언어로는 사용할 수 없다는 단점이 있다.

`Writable` interface에는 두가지 method들이 있다. 하나는 상태를 `DataOutput`의 binary stream에 write하는 것이고, 다른 하나는 `DataInput` binary stream으로 부터 상태를 read하는 것이다. `serialization`과 `deserialization`이 가능하다고 보면 되겠다. `Writable`도 다양한 종류의 class들이 있는데 원시 타입들마다 다른 class를 사용한다.

# File-Based Data Structures

가끔 데이터를 저장하기 위해 특별한 자료구조를 필요로 할 때가 있다. `MapReduce` job을 처리할 때, binary data의 blob별로 파일을 만들어서 저장하는 것은 scalability가 떨어지기 때문에 하둡은 다양한 higher-level container들을 갖고 있다.

### SequenceFile

한 줄짜리 텍스트로 이루어진 로그파일이 있다고 가정한다. binary type을 로깅한다면 plain text는 적합하지 않다. 따라서 하둡의 `SequenceFile` class는 binary key-value pair를 저장하기 위한 자료구조를 제공한다. `SequenceFile` class는 작은 파일들을 저장하는데도 유리하다. `HDFS`와 `MapReduce`는 큰 파일들을 처리하는 것에 최적화 되어 있기 때문에 큰 파일들을 `SequenceFile`로 packing하면 작은 파일들을 더 효율적으로 저장할 수 있다.

`SequenceFile.Writer` 인스턴스를 사용해서 `SequenceFile`을 write 할 수 있다. write를 실행할 stream과 configuration, key-value의 type과 같은 필수 항목들을 지정해주면 된다. 압축 타입과 codec, write중에 실행할 callback, metadata정보들도 optional하게 넣어줄 수 있다. write를 하려면 해당 인스턴스에서 계속 `.append()`해주면 된다.

유사하게 `SequenceFile.Reader`인스턴스를 사용해서 `SequenceFile`을 iteration을 통해 read할 수 있다.

### MapFile

`MapFile`은 key를 사용해서 lookup가능한 index로 정렬된 `SequenceFile`이다. index를 메모리에 저장해서 빠르게 검색한다는 장점이 있다. `set`, `array`, `bloommap`등의 자료구조를 포함한다. `set`과 `array`는 이미 익숙한 개념일거고, `bloommap`은 `get()` method를 조금 더 빠르게 처리해준다는 장점이 있는 자료구조이다. 자바스크립트에서 `Map`하고 비슷한 개념인 것 같다. 아??? 그래서 `MapFile`인가....
