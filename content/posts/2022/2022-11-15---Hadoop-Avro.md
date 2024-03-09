---
title: 하둡 Avro
date: "2022-11-15T23:35:37.121Z"
template: "post"
draft: false
slug: "/hadoop/what-is-avro"
category: "Hadoop"
tags:
  - "Hadoop"

description: "Hadoop에서 Avro가 필요한 이유"
---

`Avro`는 언어와 무관하게 사용할 수 있는 data serialization 시스템이다. 하둡은 다양한 언어를 지원하지 않는다는 단점이 있는데, Avro는 이 단점을 보완하기 위해 만들어졌다. 따라서 Avro는 언어와 독립적인 schema를 사용하고, 코드를 작성하지 않고서도 데이터를 read하고 write할 수 있다. 코드를 작성하지 않기 때문에 Avro는 read하거나 write할 때 모두 schema를 필요로 한다.

Avro schema는 주로 JSON으로 작성된다. 데이터 인코딩에는 다양한 방식이 있지만 일반적으로 binary format으로 인코딩 된다. C와 같은 언어로 schema를 작성할 수 있는 Avro IDL이라는 상위 언어가 있고, 프로토타입과 디버깅에 편리한 JSON-based data encoder도 있다.

`Avro specification`은 `avro` 사용 설명서라고 이해하면 된다. 가장 큰 장점은 `schema resolution`인데, read에 필요한 schema와 write에 필요한 schema가 일치하지 않아도 된다. 예전 schema로 작성된 데이터가 있고, 거기에 새로운 field가 추가됐다고 할 때, 같은 데이터에 old schema를 사용해서 read하고, new schema를 사용해서 write를 할 수 있는 장점이 있다. 만약 old schema를 가진 클라이언트가 해당 데이터를 read한다면, new schema에 해당하는 값들은 읽어올 수 없지만 에러가 발생하지는 않는다. 아래에 예제로 첨무되어 있다.

# Avro Data Types and Schemas

Avro에는 primitive type과

![avro-primitive-type](https://i.imgur.com/4XdyBY3.png)

complex type이 있다.

![avro-complex-type](https://i.imgur.com/9K0Inqv.png)

스크린샷 이외에 Union도 있는데, 다양한 schema들이 섞인거라고 보면 된다. JSON array이고, array의 각 element들은 독립적인 schema를 가진다

complex type에서는 `record`가 가장 흔한 것으로 보이는데,

1. 필수항목

   - name: `record`의 이름을 나타냄
   - fields: schema값들을 나타내는 JSON array. 여기서도 name은 필수인데 나버지는 옵셔널

2. 선택항목
   - namespace:
   - doc: schema 사용자의 정보를 나타내는 JSON string
   - aliases: `record`의 별칭

```json
{
  "type": "record",
  "name": "LongList",
  "aliases": ["LinkedLongs"],
  "fields": [
    { "name": "value", "type": "long" },
    { "name": "next", "type": ["null", "LongList"] }
  ]
}
```

각각의 `Avro Language API`는 각 언어에 맞는 Avro type을 갖는다. `Avro double`은, C, C++, Java에서는 `double`이지만 python에서는 `float`이다.

만약 런타임에 schema를 파악할 수 없다면 dynamic mapping을 사용한다. Java에서는 `Generic` mapping이라고 부른다.

추가로 Java나 C++로 구현할 경우, Avro schema에 맞게 작성할 수 있다. 코드를 generate하는 것을 Java에서는 `Specific` mapping이라고 부른다. 이것은 `Generic` mapping과는 다르게 schema를 갖고 있는 경우에 사용된다. schema를 가지고 있기 때문에 `Generic`보다 더 domain-oriented된 API를 제공한다.

Java는 `Reflect` mapping이라는 것도 있는데, `Avro type`을 reflection을 사용해서 `Java` type들과 mapping하는 것이다. `Generic`이나 `Specific`보다는 느리지만, `Avro`가 schema를 미리 파악할 수 있기 때문에 타입 선언할 때 편리하다.

![avro-java-mapping](https://i.imgur.com/pehj43e.png)
![avro-java-mapping](https://i.imgur.com/aFB9m8x.png)

`avro string`을 예로 들면 java에서 `string`일 수도 있고 `utf8`일수도 있다. `utf8`을 사용하게 된다면 `utf8`이 같은 instance를 read하거나 write할 때 재사용할 수 있고, mutable하기 때문이다. 그리고 java string은 utf-8을 Object construction time에 decode하는 반면 avro utf-8은 lazy하게 decode되는데, 어떤 경우에는 퍼포먼스에 훨씬 유리하다.

# In-Memory Serialization and Deserialization

Avro는 serialization과 deserialization을 위한 api를 제공한다. `Java` 예제와 함께 살펴본다

```json
{
  "type": "record",
  "name": "StringPair",
  "doc": "A pair of strings.",
  "fields": [
    { "name": "left", "type": "string" },
    { "name": "right", "type": "string" }
  ]
}
```

위 schema가 `StringPair.avsc`라는 파일에 저장된다면, 아래와 같은 코드로 불러올 수 있다.

```java
Schema.Praser parser = new Schema.Parser();
Schema schema = parser.parse(
  getClass().getResourceAsAstream("StringPair.avsc")
);
```

`Generic API`를 사용하면 avro instance를 생성할 수 있다.

```java
GenericRecord datum = new GenericData.Record(schema);
datum.put("left", "L");
datum.put("right", "R");
```

이제 output stream으로 serialize한다

```java
ByteAraryOutputStream out = new ByteArrayOutputStream();
DatumWriter<GenericRecord> writer = new GenericDatumWriter<GenericRecord>(schema);

Encoder encoder = EncoderFactory.get().binaryEncoder(out, null);
writer.write(datum, encoder);
encoder.flush();
out.close();
```

여기서 중요한 것은 `DatumWriter`와 `Encoder`이다. `DatumWriter`는 data objects들을 `Encoder`가 이해할 수 있는 type으로 변경하고, `Encoder`는 이해한 data를 output stream에 write한다. `null`을 encoder factory로 보내는 이유는, 예전에 선언된 encoder를 재사용하지 않기 때문이다.

예제에서는 `.write()`에 하나만 들어가지만, `.close()`를 호출하기 전에 여러 object들을 넣을 수도 있다. `GenericDatumWriter`는 전달 받은 schema를 사용해서 어떤 데이터를 write할지 결정한다. `.write()`를 호출하고 `.flush()`한 후에 `.close()`하는 과정을 거친다. 위 순서를 역행해서 byte buffer에서 object를 다시 읽을 수도 있다.

```java
DatumReader<GenericRecord> reader = new DatumReader<GenericRecord>(schema);
Decoder decoder = DecoderFactory.get().binaryDecoder(out.toByteArray(), null);
Generic Record result = reader.read(null, decoder);
assertThat(result.get("left").toString(), is("L"));
assertThat(result.get("right").toString(), is("R"));
```

`null`은 object를 재사용하지 않기 때문에 사용되고, `results.get()`이 return하는 것은 `utf8`이다. 따라서 `.toString()`을 호출해서 `Java String`으로 변환한다.

### The Specific API

위에서 진행한 것을 specific API로 확인해보자 `StringPair` class를 schema 파일로부터 만들고, avro의 `maven` 플러그인을 사용해서 schema를 컴파일 할 수 있다.

```xml
<project>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.avro</groupId>
        <artifactId>avro-maven-plugin</artifactId>
        <version>${avro.version}</version>
        <executions>
          <execution>
            <id>schemas</id>
            <phase>generate-sources</phase>
            <goals>
              <goal>schema</goal>
            </goals>
            <configuration>
              <includes>
                <include>StringPair.avsc</include>
              </includes>
              <stringType>String</stringType>
              <sourceDirectory>src/main/resources</sourceDirectory>
              <outputDirectory>${project.build.directory}/generated-sources/java
              </outputDirectory>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
 </build>
 ...
</project>
```

`maven` 대신 `Avor's Ant Task`, `org.apache.avro.specific.SchemaTask`, 또는 `Avro command-line tools`를 사용할 수도 있다.

serialization과 deserialization에서는 위에서 언급한 `GenericRecord` 대신 `StringPair` instance를 생성하고, 이 인스턴스를 `SpecificDatumWriter`를 사용해서 stream에 write한다.

```java
StringPair datum = new StringPair();
datum.setLeft("L");
datum.setRight("R");

ByteArrayOutputStream out = new ByteArrayOutputStream();
DatumWriter<StringPair> writer = new SpecificDatumWriter<StringPair>(StringPair.class);
Encoder encoder = EncoderFactory.get().binaryEncoder(out, null);
writer.write(datum, encoder);
encoder.flush();
out.close();

DatumReader<StringPair> reader = new SpecificDatumReader<StringPair>(StringPair.class);
Decoder decoder = DecoderFactory.get().binaryDecoder(out.toByteArray(),
null);
StringPair result = reader.read(null, decoder);
assertThat(result.getLeft(), is("L"));
assertThat(result.getRight(), is("R"));
```

### Avro Datafiles

Avro의 object container file format은 avro object들의 sequence를 저장하기 위해서 사용된다. Hadoop의 sequence file format과 매우 유사한데 차이점이 있다면 avro는 다양한 언어를 지원한다는 것이다. 예를 들면 python으로 Write하고 C로 read 할 수 있다.

`datafile`는 header와 file data block으로 구성된다.

header는

1. 4bytes, ASCII 'O','b', 'j', 1 이들어있고
2. schema 정보를 포함한 metadata
3. 16byte random-generated `sync marker` 로 이루어져있다.

살짝 이런느낌

```json
{ "type": "map", "values": "bytes" }
```

data block은

1. block안에 object를 count 하는 `long`
2. `codec` 이 적용된 후에 현재 block에 serialized된 object들의 크기를 나타내는 `long`
3. serialized objects. codec이 적용되어있으면 codec으로 압축됨
4. 파일의 16-byte sync marker 로 구성되어 있다.

codec은 required codec과 optional codec으로 나누어지는데

1. Required Codecs

- null : 데이터를 압축하지 않음
- deflate RFC 1951에 나오는 defalte 알고리즘을 사용해서 data block을 write함. 일반적으로 zlib library사용

2. Optional Codecs

- bzip2
- snappy: 구글의 Snappy compression library 사용. 각각의 압축된 block들은 4-byte big-endian CRC32 checksum으로 구분됨
- xz
- zstandard: 페이스북(메타)의 Zstandard compression library 사용

object를 `datafile`에 write하는 것은 stream에 write하는 것과 유사하다. `DatumWriter`를 사용하는 것 까지는 같은데, `Encoder`대신 `DataFileWriter`를 사용한다. 새로운 `datafile`을 생성하고, object를 append하는 방식으로 write한다. object는 파일의 schema를 따르지 않으면 exception이 발생한다.

```java
File file = new File("data.avro");
DatumWriter<GenericRecord> writer = new GenericDatumWriter<GenericRecord>(schema);
DataFileWriter<GenericRecord> dataFileWriter = new DataFileWriter<GenericRecord>(writer);
dataFileWriter.create(schema, file);
dataFileWriter.append(datum);
dataFileWriter.close();
```

local file말고 `OutputStream`에도 write할 수 있다.

```java
public FSDataOutputStream create(Path f) throws IOException
```

이렇게하면 HDFS에도 write가 가능하다.

역순으로 하면 stream에서 한 것 처럼 read도 된다. 하지만 이번에는 파일의 metadata에서 읽기 때문에 stream에서 read할 때와 다르게 schema를 선언해주지 않아도 된다.

```java
DatumReader<GenericRecord> reader = new GenericDatumReader<GenericRecord>();
DataFileReader<GenericRecord> dataFileReader = new DataFileReader<GenericRecord>(file, reader);
assertThat("Schema is the same", schema, is(dataFileReader.getSchema()));
```

`DataFileReader`는 interator이기 때문에, `hasNext()`, `next()`등의 method를 호출해서 iteration할 수도 있다.

```java
assertThat(dataFileReader.hasNext(), is(true));
GenericRecord result = dataFileReader.next();
assertThat(result.get("left").toString(), is("L"));
assertThat(result.get("right").toString(), is("R"));
assertThat(dataFileReader.hasNext(), is(false));
```

`next()`를 호출하는 것보다 return될 object의 인스턴스를 사용하는 `overloaded form`을 사용하는 것이 더 좋다. 이렇게 할 경우 object를 재사용 할 수 있고, allocation과 garbage collection 비용을 줄일 수 있다.

```java
GenericRecord record = null;
while (dataFileReader.hasNext()) {
  record = dataFileReader.next(record);
}

for (GenericRecord record : dataFileReader) {
  // object 재사용이 중요하지 않은 경우
}
```

hadoop에서 파일을 read하는 경우에는 `FsInput`을 사용해서 Hadoop의 `Path` object를 작성해주는 것이 좋다. `DataFileReader`는 avro datafiles에 random access를 제공하긴 하지만 `DataFileStream`을 사용한 sequencial streaming이면 충분하다. `DataFileStream`은 java의 `InputStream`으로 부터 read할 수 있다.

# Interoperability

python으로 write하고 Java로 read 해본다.

### Python API

csv에서 데이터를 불러와서 `StringPair` schema로 write해서 avro datafile에 기록한다.
Java에서 했던 것과 유사하게 `DatumWriter`와 `DataFileWriter` object를 사용한다. python은 avro record를 dictionary형태로 나타내기 때문에, 읽어온 값을 `dict` object로 변환한 다음에 `DataFileWriter`에 append한다.

```python
import os
import string
import sys

from avro import schema
from avro import io
from avro import datafile

if __name__ == '__main__':
  if len(sys.argv) != 2:
  sys.exit('Usage: %s <data_file>' % sys.argv[0])
  avro_file = sys.argv[1]
  writer = open(avro_file, 'wb')
  datum_writer = io.DatumWriter()
  schema_object = schema.parse("\
    {
      "type": "record",
      "name": "StringPair",
      "doc": "A pair of strings.",
      "fields": [
        {"name": "left", "type": "string"},
        {"name": "right", "type": "string"}
      ]
    }
  ")
  dfw = datafile.DataFileWriter(writer, datum_writer, schema_object)
  for line in sys.stdin.readlines():
    (left, right) = string.split(line.strip(), ',')
    dfw.append({'left':left, 'right':right});
    dfw.close()
```

### Avro tools

jar파일 가져와서 읽어들일 수 있다.

```bash
% java -jar $AVRO_HOME/avro-tools-*.jar tojson pairs.avro
{"left":"a","right":"1"}
{"left":"c","right":"2"}
{"left":"b","right":"3"}
{"left":"b","right":"2"}
```

# Schema Resolution

write할 때 사용한 schema와 다른 schema를 사용해서 read할 수도 있다. 아래와 같은 새로운 schema가 있다고 하자

```json
{
  "type": "record",
  "name": "StringPair",
  "doc": "A pair of strings with an added field.",
  "fields": [
    { "name": "left", "type": "string" },
    { "name": "right", "type": "string" },
    { "name": "description", "type": "string", "default": "" } // 추가됨
  ]
}
```

새로운 schema를 사용해서 기존의 schema로 write된 데이터를 읽어올 수 있다. 왜냐면 저기 `default`가 있기 때문이다. avro는 해당 schema에 값이 없다면 default를 사용해서 read한다.
아래처럼 하면 default를 empty string대신 null로 할 수도 있다.

```json
{ "name": "description", "type": ["null", "string"], "default": null }
```

타입이 여러개도 된다는 것 같다.

schema가 여러개인 경우에는 `GenericDatumReader`에 두가지 schema를 모두 넘겨준다

```java
DatumReader<GenericRecord> reader = new GenericDatumReader<GenericRecord>(schema, newSchema);
Decoder decoder = DecoderFactory.get().binaryDecoder(out.toByteArray(),
null);
GenericRecord result = reader.read(null, decoder);
assertThat(result.get("left").toString(), is("L"));
assertThat(result.get("right").toString(), is("R"));
assertThat(result.get("description").toString(), is(""));
```

위에서는 datafile에서 read할 때는 metadata에 schema가 이미 있기 때문에 별도로 제공하지 않아도 된다고 했지만, read할 때 사용하는 schema가 write할 때 사용된 schema와 다른 경우에는 새로운 schema는 제공해야한다.

```java
DatumReader<GenericRecord> reader = new GenericDatumReader<GenericRecord>(null, newSchema);
```

read할 때 `projection`을 통해 schema의 field를 drop하기도 한다. field가 많은 데이터가 있을 때 특정 데이터만 불러오는 경우에 유용하게 사용된다.

```json
{
  "type": "record",
  "name": "StringPair",
  "doc": "The right field of a pair of strings.",
  "fields": [{ "name": "right", "type": "string" }] // left가 빠짐
}
```

![avro-schema-resolution](https://i.imgur.com/nQi0D3u.png)

`alias`를 사용해서 schema가 다른 경우를 보완할 수도 있다. 각 schema의 field에 `alias`를 넣어주는 방식인데, write할 때 사용된 field가 read할 때 사용된 field와 다른 경우에 사용된다.

```json
{
  "type": "record",
  "name": "StringPair",
  "doc": "A pair of strings with aliased field names.",
  "fields": [
    { "name": "first", "type": "string", "aliases": ["left"] }, // left로 쓰고 first라고 읽어옴
    { "name": "second", "type": "string", "aliases": ["right"] } // right로 쓰고 second라고 읽어옴
  ]
}
```

`alias`는 이미 translated 되었기 때문에 사용자에게 공개되지 않는다.

# Sort Order

avro는 객체의 sort order를 정의한다. 대부분의 avro type들은 숫자 오름차순, enum은 정의된 순서 등으로 sort된다. avro type들은 sort order가 `specification`의 내용대로 미리 정해져있다. 하지만 `record`의 경우에는 `order` attribute를 변경해서 sort order를 조정할 수 있다. `ascending`, `descending`, `ignore`의 옵션을 사용할 수 있다.

```json
{
  "type": "record",
  "name": "StringPair",
  "doc": "A pair of strings, sorted by right field descending.",
  "fields": [
    { "name": "left", "type": "string", "order": "ignore" },
    { "name": "right", "type": "string", "order": "descending" }
  ]
}
```

위와 같은 방식을 사용하면 `right`의 내림차순으로 정열 된다. `left`는 sort 방식에서는 사용되지 않지만, 값은 남아있다.

`record`의 필드는 reader의 schema에 따라 pair로 구분된다. 따라서 schema를 어떻게 선언하느냐에 따라서 간접적으로 record의 sort order를 정할 수 있다. 아래의 경우에는 right로 먼저 정렬하고 그 다음에 left를 처리한다.

```json
{
  "type": "record",
  "name": "StringPair",
  "doc": "A pair of strings, sorted by right field descending.",
  "fields": [
    { "name": "right", "type": "string" },
    { "name": "left", "type": "string" }
  ]
}
```

avro는 효율적인 binary comparison을 하기 때문에, binary data를 deserialize해서 순서를 비교하지 않아도 된다. 제일 처음 언급한 `StringPair`를 예로 들면, `left`는 `utf-8` enconding된 string이고, 따라서 bytes를 알파벳 순으로 비교한다. 만약 다르다면 순서는 바로 결정되고, 비교를 멈춘다. 만약 두개의 byte sequence가 일치한다면, `right`로 이동해서 비교한다.

# Avro MapReduce

avro는 `MapReduce` job을 쉽게 처리하기 위한 class들을 제공한다. `org.apache.avro.mapreduce` package를 사용해서 어떻게 처리되는지 보여주도록 하겠다.

사용할 schema는 아래와 같다.

```json
{
  "type": "record",
  "name": "WeatherRecord",
  "doc": "A weather reading.",
  "fields": [
    { "name": "year", "type": "int" },
    { "name": "temperature", "type": "int" },
    { "name": "stationId", "type": "string" }
  ]
}
```

첫번째 예제는 `Generic` mapping이다. type 에러가 발생할 수 있지만 그래도 `record`를 처리하기위해 코드를 작성하는 수고를 덜 수 있다. `schema`는 가독성을 위해 변수를 사용하지 않고 inline으로 작성했다. 실제로는 schema를 파일에 따로 작성하고 불러오는 편이 더 합리적일 것이다.

일반적인 Hadoop MapReduce API와의 차이점이 몇가지 있다.

하나는 `Avro Java type`에 wrapper를 사용하는 것이다. 예제에서 key는 연도인데(숫자), value는 avro의 `GenericRecord`이다. output에서 key의 타입은 `AvroKey<Integer>` 로 변환되고, value의 타입은 `AvroValue<GenericRecord>`로 변환된다. 이 타입은 map output/reduce input의 타입과 일치한다. `MaxTemperatureReducer`는 record에 대해 key를 사용해서 iterate하고, 가장 높은 기온을 찾는다. 지금까지 찾은 가장 높은 기온을 기록해뒀다가, 이 값을 재사용한다

두번째는 `AvroJob`을 사용한다는 것이다. `AvroJob`은 input, map input, output의 avro schema를 선언할 수 있는 class이다. text file에서 읽어들이기 때문에 예제에 input schema는 없다. map output key의 schema는 `Avro int`이고, value는 `weather record`이고, output은 `AvroKeyOutputFormat`이다. 이 output은 avro datafile로 write된다.

아래 명령어로 실시할 수 있고

```bash
% export HADOOP_CLASSPATH=avro-examples.jar
% export HADOOP_USER_CLASSPATH_FIRST=true # override version of Avro in Hadoop
% hadoop jar avro-examples.jar AvroGenericMaxTemperature \
 input/ncdc/sample.txt output
```

아래 명령어로 output을 확인할 수 있다.

```bash
% java -jar $AVRO_HOME/avro-tools-*.jar tojson output/part-r-00000.avro
{"year":1949,"temperature":111,"stationId":"012650-99999"}
{"year":1950,"temperature":22,"stationId":"011990-99999"}
```

# Sorting Using Avro MapReduce

avro의 정열 기능을 MapReduce와 연결해서 avro datefile을 sort해본다.

예제는 다시 한 번 `Generic` mapping을 사용한다. avro 의 모든 type을 sort할 수 있는 코드이다. Java Generic에서는 `<K>`로 표현된다. key와 같은 값을 사용해서 값들이 key를 사용해서 구분될 수 있게 했고, 만약 중복되는 key가 있다면 제거해서 효율을 높일 수 있다. mapper는 key-value pair를 `AvroKey - AvroValue` pair로 변경한다. 그리고 reducer는 이 값들을 output key로 전달하고, 이 값들이 avro datafile에 기록된다.

sorting은 MapReduce shuffle에서 일어나고, sort 함수는 avro schema에 의해 결정된다.

# Avro in Other Languages

Java이외에 달느 언어들도 쓸 수 있는데, 언어를 언급하지는 않는다. `AvroAsTextInputFormat`은 Hadoop streaming을 통해 avro datafile을 read할 수 있게한다. 파일의 각 datum은 JSON을 나타내는 string으로 변환되거나, 만약 `Avro bytes`라면 bytes로 변환된다.

`AvroTextOutputFormat`을 streaming job의 output으로 선언하면, streaming의 결과를 avro datafile에 bytes schema로 저장할 수 있다. 각 datum은 tab-delmitaed key-value pair로 저장된다.

`Pig`, `Hive`, `Crunch`, `Spark`를 사용해서 avro processing을 할 수도 있다. 저것들 다 avro datafile을 read/write가 가능하다.
