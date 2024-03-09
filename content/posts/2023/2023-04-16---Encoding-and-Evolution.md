---
title: "Encoding and Evolution"
date: "2023-04-16T20:35:37.121Z"
template: "post"
draft: false
slug: "/database/encoding-and-evolution"
category: "Database"
tags:
  - "Database"

description: "과거 코드와 신규 코드가 정상적으로 조화(?)를 이룰 수 있도록 설계해야한다"
---

어플리케이션은 새로운 기능 추가나 변경에 대응할 수 있도록 설계되어야 한다. 기존의 테이블에 새로운 column을 추가하거나 수정할 수도 있고, 새로운 테이블이 추가되기도 한다. 관계형 데이터베이스의 경우 스키마가 고정되어있는데, 스키마가 변경될 경우 해당 데이터를 처리하는 코드 또한 같이 수정되어야 한다. 하지만 어플리케이션의 규모가 큰 경우, 즉각적인 코드 수정이 어려울 수도 있다. 

서버는 downtime을 최소화 하기위해 rolling update(continuous delivery)를 시도해야하고, 클라이언트의 경우에는 사용자가 업데이트를 하지 않는 경우 에러가 발생할 수 있다. 따라서 사용자들이 사용하는 어플리케이션은 신규코드와 과거코드가 조합된 어플리케이션이 상용화 되어있을수도 있다. 따라서, 신규코드와 과거코드는 서로 호환될 수 있도록 작성해야한다. 

새로운 코드가 과거 코드의 데이터를 읽어오는 것은 별로 어렵지 않다. 개발자가 본인이 작성한 코드를 기억할 수도 있고, 기억하지 못하면 다시 보면 되기 때문이다. 하지만 과거코드가 신규 코드로부터 데이터를 가져오는 것은 복잡하다. 따라서 이번장에는 다양한 데이터 인코딩 포맷들과 REST, RPC등의 네트워크 통신방식을 통해 스키마 변경에 어떻게 대처하는지 알아보도록 하겠다. 

# Formats for Encoding Data

데이터는 일반적으로 2가지로 나타낸다. 

1. 메모리에 저장되는 경우 object, struct, list, array, hashtable, tree등의 구조로 저장되어 CPU가 해당 데이터를 효율적으로 처리할 수 있게 하거나
2. 네트워크를 통해 데이터를 전송할 때는 JSON document와 같은 형식으로 데이터를 처리한다.

따라서 1 <-> 2 형식으로 데이터를 변환하는 절차가 필요하다. 1 -> 2로 데이터를 변경하는 것을 인코딩(또는 serializaion)이라 칭하고, 2 -> 1로 데이터를 변경하는 것을 디코딩(또는 parsing)이라 칭한다. 

### Language-Specific Formats

프로그래밍 언어들이 간단하게 in-memory 객체들을 byte sequence로 변경하기 위한 built-in 기능들을 제공하기도 한다. 하지만 몇몇 문제들도 있는데, 

1. 인코딩이 특정 언어와 묶여있기 때문에 다른 언어를 사용해서 해당 데이터를 읽어들이는 것이 어렵다. 해당 언어의 기능이나, 특정 라이브러리를 사용한다면, 그 언어를 지속적으로 사용해야 할수도 있다. 
2. 특정 언어를 사용해서 인코딩한 객체를 디코딩할 경우 추상적인 클래스의 인스턴스가 되는 경우가 있는데, 이는 보안에 매우 취약하다. 
3. 버전 관리가 안되기때문에 위에서 언급한 신규코드 <-> 과거코드간 데이터 교류가 불가능하다
4. 작성해야 하는 코드 수는 줄어들 수 있지만, CPU사용 측ㅁ면에서 효율이 떨어진다. 

따라서 왠만하면 프로그래밍 언어의 built-in기능을 사용해서 인코딩 하는 것은 피하는 것이 좋다. 

### JSON, XML, and Binary Variants

JSON과 XML이 가장 많이 사용되는 포맷이다. 이와 동시에 단점도 많은데, XML은 길이가 너무 길도, JSON은 web에서 지원해줘서 유명한 것 뿐이라는 반대의 견들이 있다. CSV도 많이 사용되긴 하는데, 활용할 수 있는 방법이 상대적으로 적다. 그래도 해당 포맷들을 사용하면 사람이 읽을 수 있는 데이터가 된다는 장점이 있는데(컴퓨터는 다 0, 1로만 저장을 하는 것과 비교하자면), 해당 포맷들이 가진 단점들도 상당하다. 

1. 숫자를 인코딩할 때 불확실성이 너무 많다. XML과 CSV의 경우에는 숫자와 문자열을 구분할 수 없다는 단점이 있다. JSON은 구분은 가능하지만, int와 float을 구분하지 못하고, precision을 제공하지 않는다. 큰 숫자를 처리할 때 특히 문제가 되는데, 2^53 보다 큰 숫자의 경우 float을 사용하는 언어로 디코딩됐을 때 정확도가 떨어진다. 
2. JSON과 XML은 Unicode character string을 지원하지만, binary string은 지원하지 않는다. 따라서 binary string을 사용하려고 하는 개발자들은 base64로 인코딩해서 사용한다. 작동은 하지만 1/3 정도의 데이터 손실이 발생한다. 
3. JSON은 스키마를 지원하지 않는 경우가 많다. RDB를 사용해서 데이터를 저장했다고 할 때, 검증이 부족한 것이다. 
4. CSV는 스키마가 아예 없기 때문에 어플리케이션단에서 각각의 row와 column을 처리해야한다. 

이러한 단점들이 있긴 하지만, 아마 왠만하면 이것들을 사용해서 데이터를 처리할 수 있을 것이다. 

##### Binary encoding

데이터 사이즈가 커져서 TB단위에 들어가면 인코딩 방식이 중요해진다. JSON은 XML보다 짧지만, 둘 다 binary format과 비교하면 용량이 너무 크다. JSON에 binary encoding을 지원하기 위해 BSON, BJSON 등등이 만들어졌지만, JSON을 활용한 텍스트 인코딩에 비하면 사용율이 훨씬 떨어진다. `MessagePack`이라는 JSON binary encoding방식을 사용하게 되면, 아래 데이터를 인코딩할 때 15 byte를 아낄 수 있다. 

```json
{
  "userName": "Martin",
  "favoriteNumber": 1337,
  "interests": ["daydreaming", "hacking"]
}
```

앞으로 다양한 인코딩 기술을 소개하면서 어떻게 용량을 더 줄일 수 있을지 논할예정이다. 

### Thrift and Protocol Buffers

[Apache Thrift](https://thrift.apache.org/)와 [Protocol Buffers(protobuf)](https://protobuf.dev/)는 인코딩 되는 데이터의 schema를 강제하는 binary encoding library이다. 위에서 작성한 JSON데이터를 Thrift를 사용해서 인코딩한다면, 아래와 같은 스키마가 필요하다 

```c
struct Person {
  1: required string userName,
  2: optional i64 favoriteNumber, 
  3: optional list<string> interests
}
```

Protobuf도 아래와 같은 스키마를 필요로한다 

```
message Person {
  required string user_name       = 1;
  optional int64  favorite_number = 2;
  repeated string interests       = 3;
}
```

Thrift와 Protobuf는 위에서 언급한 스키마를 활용해서 데이터를 개발자가 사용하는 프로그래밍 언어의 클래스로 인코딩한다. `MessagePack`의 인코딩 방식과 가장 큰 차이는, key를 string으로 가지고 있지 않고, 스키마에 선언한 숫자(필드태그)로 관리한다는 점이다. 이에 따라 용량이 감소하고, 필드 타입과, 필드 태그를 하나의 byte에 패킹하면서 용량을 아낀다. Protobuf 또한 이와 유사한 방식으로 binary encoding을 실시하고, 처음에 언급한 데이터를 33byte로 인코딩하면서, MessagePack의 50%의 공간만 사용하게 된다. 

##### Field tags and schema evolution

스키마는 바뀔수밖에 없다는 것을 이 장의 맨 처음에 언급했고, 이를 schema evolution이라 칭한다. Thrift와 Protobuf는 모두 field tag, 위에 작성된 1, 2, 3과 같은 숫자들을 사용해서 신규코드와 과거코드가 호환될 수 있도록 한다. 신규 클라이언트 코드가 과거 서버 코드에 데이터를 요청할 때, 과거코드에 작성되지 않은 field tag를 요청한다면, 해당 값을 무시해버린다. 이와 유사하게 과거 클라이언트 코드가 신규 서버 코드에게 신규코드에 작성된 field tag를 요청하지 않는다면, 신규 코드에는 해당 값이 작성되어있다고 하더라도, 이를 보내지 않는 식으로 에러를 방지한다. 

##### Datatypes and schema evolution

요청하는 데이터의 타입이 변경된 경우는 어떻게 처리할까? `Protobuf`는 list 타입을 지원하지 않고 `repeated`라는 타입을 사용한다. 말 그대로 어떤 값이 반복되어 나타난다는 것이다. 따라서 신규코드나 과거코드는 해당 값이 반복되는 정도에 차이가 있을 뿐, 에러는 발생하지 않는다. Thrift는 list 타입을 지원한다. Protobuf 만큼의 유연성은 없지만 nested list를 사용해서 데이터타입 변경에 대응할 수 있다. 

### Avro

[Apache Avro](https://avro.apache.org/)는 Hadoop에서 사용된다. Thrift, Protobuf와 유사하게 스키마를 사용한다. 

```avro
record Person {
  string                userName;
  union { null, long }  favoriteNumber = null; 
  array<string> interests;
}
```

위 포맷은 Avro IDL인데, JSON형식으로 나타내자면 아래 느낌이다. 

```json
{
  "type": "record",
  "name": "Person",
  "fields": [
    {"name": "userName", "type": "string"},
    {"name": "favoriteNumber", "type": ["null", "long"], "default": null}, {"name": "interests", "type": {"type": "array", "items": "string"}}
  ] 
}
```

Thrift, Protobuf와 비교하자면, field tag라고 칭했던 숫자를 사용하지 않는다. 인코딩할 값이 더 작아지면서 Avro를 사용하면 32byte의 공간만 차지한다. 데이터 타입이나 key를 정의하지 않고, 하나로 concat된 형태이기 때문이다. 따라서 인코딩 된 데이터를 parsing할 때는, 디코딩하면서 선언된 스키마를 보고 데이터타입을 파악하는 과정이 필요하다. 

Avro에 대해서는 예전에 Hadoop을 공부하면서 [자세하게 작성한 글](https://jasonkang14.github.io/hadoop/what-is-avro)이 있다. 

### The Merits of Schemas

XML, JSON, CSV와 같은 텍스트 인코딩 방식과 비교했을 때, Thrift, Protobuf, Avro와 같은 바이너리 인코딩은 아래와 같은 장점이 있다. 

1. binary JSON보다 훨씬 작은 용량으로 인코딩할 수 있다. 
2. 디코딩 시 스키마를 필요로 하기 때문에 지속적으로 관리해서 에러 가능성을 낮출 수 있다. 
3. 신규코드와 과거코드의 호환성을 스키마를 활용하여 검증할 수 있다. 
4. static type 프로그램이 언어를 사용하는 경우, 스키마를 사용해서 코드 생산성을 향상시킬 수 있다. 

# Modes of Dataflow

위에서 언급한것과 같이 메모리를 공유하지 않는 다른 프로세스와 데이터를 주고 받으려면 byte sequence로 인코딩 해야한다. 그리고 신규코드와 과거코드가 서로 데이터를 주고 받을 때 에러가 발생하지 않도록 서비스를 설계해야한다(Compatibility). 데이터를 전달하는 측은 인코딩을 해야하고, 받는 측은 디코딩을 해야한다. 그렇다면 인코딩/디코딩의 주체는 어디인가?

### Dataflow through Databases

데이터베이스에서 read/write를 하는 경우, read를 하는 프로세스는 디코딩하고, write하는 프로세스는 인코딩한다. 데이터베이스와 연결된 프로세스가 하나이고, 해당 프로세스와 read/write를 모두 실시할수도 있다. 그렇다면 미래의 나에게 전송할 데이터를 저장하는 기능을 구현할수도 있다(cache)

이 경우 신규코드와 과거코드의 호환이 더욱 중요해진다. 해당 프로세스의 과거코드가 cache한 값을 해당 프로세스의 신규코드가 처리할 수 없다면 에러가 발생한다. 또 주목할 점은 신규코드가 write한 값을 과거코드가 read할 수도 있다. 서비스 기획 변경이나 추가로 데이터베이스 스키마가 수정되면, 이런일은 빈번하게 발생할 수 있다. 

특히 데이터베이스에서 불러온 값을 프로그래밍 언어의 객체로 디코딩 한 후에, 해당 값을 활용해서 데이터베이스에 write하기 위해 encoding을 실시한다면, 과거코드는 새롭게 추가된 값들을 잃게될 것이다. 

##### Different values written at different times 

백엔드 서버를 신규코드로 배포하면, 짧은 시간안에 과거 코드를 모두 제거하고 새로운 코드로 업데이트 할 수 있다. 하지만 인코딩되어 데이터베이스에 저장된 값들은 서버 업데이트와 관계없이 그대로 유지된다. 

서버를 업데이트 할 때마다 데이터베이스도 업데이트 할 수도 있겠지만, 비용이 너무 비싸다. 특히 데이터셋이 크다면 시간이 엄청 오래걸릴 것이다. 따라서 데이터베이스는 null을 할당한다거나 하는 방식으로 스키마 변경시 비어있는 값들을 처리한다. 

##### Archival storage

백업을 위해 데이터베이스의 스냅샷을 떠놓는다고 가정해보자--클라우드들이 이런걸 잘 지원해준다. 그렇다면 데이터베이서 dump는 특정 데이터의 인코딩 방식을 고려하지 않고 가장 최근의 스키마를 사용해서 값들을 인코딩할 것이다. 이럴 경우 Avro를 활용해서 인코딩 변경에 대응하는 것이 좋다. 

### Dataflow through Services: REST and RPC

client와 server의 경계는 명확하지 않다. 마이크로 서비스로 구현되어있다면, 일반적으로 서버 백엔드라고 불리는 서버가 다른 서버의 클라이언트 역할을 할 수도 있다. 마이크로서비스의 목적은 서비스를 분산해서 관리 및 유지보수를 편하게 하는 것에 있다. 

##### Web services

HTTP를 사용해서 통신하는 경우 web service라고 부른다. 웹서비스에서는 REST와 SOAP를 주로 사용해서 데이터를 주고받는다. 

REST는 프로토콜은 아니고, HTTP의 디자인 원칙에 가깝다. 

1. 간단한 데이터 포맷을 사용하고,
2. url을 통해서 해당 엔드포인트의 역할을 파악할 수 있고 
3. 인증/인가/데이터 타입들을 정의한다. 

HTTP는 주로 JSON을 사용해서 데이터를 주고받고, SOAP는 XML 포맷을 사용한다. 

##### The problems with remote procedure calls (RPCs)

RPC는 1970년대에 처음 나왔는데, 서버에 있는 프로세르를 클라이언트가 실행시켜서 데이터를 가져오는 느낌으로 이해하면 된다. HTTP request를 사용해서 데이터를 받아오는 것이 아니라, 실제로 해당 프로세스의 함수를 호출하는 것이다. 

아래와 같은 특징이 있다.

1. HTTP call은 예측이 어려운 반면, local function call은 클라이언트가 전달하는 파라미터에 따라 성공/실패가 결정될 수 있기 때문에 예측하기 쉽다. HTTP call은 네트워크 상태에 따라 request나 response를 분실할 수도 있고, remote machine이 느리거나 고장난 경우에도 생각했던 결과를 받지 못할 수도 있다.  
2. local function call은 결과를 리턴하거나, exception을 throw하거나, 아무것도 리턴하지 않는다. 하지만 HTTP call은 timeout을 걸어두면 에러의 원인을 정확히 할 수 없는 상태가 된다. 요청을 제대로 보낸건지, 아니면 요청은 보냈는데 반응이 안좋은건지 등등을 파악할 수 없다. 
3. 실패한 HTTP call을 다시한다고 가정할 때, request는 잘 보냈는데 response만 분실한 것일수도 있다. 그렇다면 deduplication(idempotence)와 같은 기능을 구현하지 않았다면, 서버는 같은 요청을 반복해서 처리하게 된다. local function call은 이와같은 문제점이 없다. 
4. local function은 호출할 때마다 비슷한 시간이 소요된다. HTTP call은 함수 호출보다 시간이 오래걸리고 latency를 예측할 수 없다. 
5. local function call은 parameter를 로컬 메모리에 저장할 수 있다는 장점이 있다. 
6. 클라이언트와 서버가 다른 프로그래밍 언어로 구현되어 있다면, RPC framework가 프로그래밍 언어를 번역하는 절차가 필요하다. 

##### Current directions for RPC

RPC가 문제가 많다고 하는데, 위 섹션에서 언급한 것들은 RPC의 장점이 아닌가? 좀 더 찾아보고 자료를 업데이트 해야할 것 같다. 

##### Data encoding and evolution for RPC

유지보수를 위해서 RPC 클라이언트와 서버는 따로 관리되어야 한다. 데이터가 데이터베이스로부터 오는게 아니라 프로세스에서 오기 때문에, 일반적으로 서버를 먼저 업데이트하고 클라이언트를 업데이트한다. 이에따라 신규코드와 과거코드 호환도 고려해야한다. 

1. Thrift, gRPC, Avro RPC당을 사용할 수 있다. 
2. SOAP은 XML을 사용하기 때문에 스키마를 수정해야 한다. 
3. RESTfulAPI는 JSON 스키마를 수정해야한다. 

### Message-Passing Dataflow

비동기 메시징도 데이터를 주고 받는데 많이 사용된다. 특히 마이크로서비스의 경우 많이 사용되는데, 데이터를 어디다가 던져두고, 해당 데이터를 처리해야하는 쪽에서 새로운 데이터가 들어왔는지 안들어왔는지 보고있다가, 새로운 데이터가 들어오면 그 데이터를 처리하는 식이다. 

