---
title: "데이터 모델과 쿼리언어"
date: "2023-03-25T20:35:37.121Z"
template: "post"
draft: false
slug: "/database/data-models-and-query-languages"
category: "Database"
tags:
  - "Database"

description: "신선한 재료로 좋은 음식을 만드는 것처럼, 적합한 데이터 모델로 좋은 소프트웨어를 만든다"
---

데이터 모델링은 소프트웨어 개발에서 가장 중요한 개념일수도 있다. 모델링을 어떻게 하느냐에 따라, 소프트웨어를 어떻게 개발할지, 그리고 우리가 풀고자 하는 문제에 대해 어떻게 생각하는지를 결정하기 때문이다. 일반적으로 많은 어플리케이션들이 하나의 모델 위에 다른 모델을 쌓는 식으로 이루어진다. 그리고 그 각각의 layer를 어떻게 표현할지를 정해야한다. 

1. 우리가 살고있는 세계를 객체나, 자료구조로 나타내고, API를 사용해서 그 데이터를 변경한다. 
2. 그리고 그 자료를 저장할 때 json이나 xml과 같은 포맷을 사용하거나, 관계형 모델이나 그래프 모델의 테이블로 저장한다. 
3. 엔지니어들은 저장된 데이터를 메모리, 디스크, 네트워크에 바이트 단위로 저장한다. 어플리케이션은 데이터를 불러오거나, 검색, 변경등 다양한 방법으로 처리할 수 있다. 
4. 조금 더 low level로 가면, 하드웨어 엔지니어들은 이 바이트를 전기신호나 전자기장 등으로 나타내는 방법을 알아냈다. 

복잡한 소프트웨어일수록 구조는 더 복잡하겠지만 그 근간은 각 layer의 아래에 있는 층들의 복잡성을 숨기면서 깔끔한 데이터 모델을 제공한다는 것이다. 예를 들면 피자를 만들 때 피자를 만들 때 필요한 토마토, 바질, 올리브유 등등이 어떻게 오는지를 공개하지 않는 느낌이다. 결국 추상화를 잘 해야한다는 뜻인데, 추상화가 잘 된 경우 여러 엔지니어들이 효과적으로 협업할 수 있게 된다. 

토마토가 어디서 왔는지 알 필요는 없지만, 신선한 토마토를 사용했을 때 좋은 피자를 만들 수 있는 것처럼, 적합한 데이터 모델을 선택했을 때 좋은 소프트웨어를 만들 수 있다. 

# Relational Model vs Document Model 

SQL은 요즘 가장 유명한 데이터 모델이다. 데이터는 **table이라 불리는 관계**로 연결되어있고, 각각의 관계는 **row라 불리는 tuple**로 이루어져있다. 1980년대 중반부터 흔히들 일반적인 구조를 가진 데이터를 저장하고 불러오기 위해 RDBMS가 많이 사용되기 시작했다. 관계형 데이터베이스가 주목을 받았던 것은, `transaction`과 `batch` 처리가 가능했기 때문이다. 여기서도 핵심은 관계형 모델이 `transaction`과 `batch`를 어떤식으로 구현했는지는 그 하단의 layer에 숨겨진다는 것이다. 그 이후 관계형 모델에 대응하는 많은 경쟁 모델들이 나타났지만, 반짝 했을 뿐 오래가지 못했다. 

## The Birth of NoSQL

NoSQL은 별다른 뜻은 없고, 초반에 SQL과 다르다는 것을 나타내기 위한 트위터 해시태그로 시작했다. NoSQL의 등장배경은 아래와 같다.

1. 데이터넷이 크다거나, write throughput이 높아서 관계형 데이터베이스보다 확장성에 더 용이한 데이터 모델이 필요한 경우. 
2. 오픈소스로 이루어진 데이터베이스의 필요성
3. 관계형 모델이 잘 지원하지 않는 특별한 쿼리들
4. 관계형 스키마가 enforce하는 모델링의 구조
5. 조금 더 다이나믹하고 표현적인 데이터 모델링에 대한 필요성 

각각의 어플리케이션들은 모두 요구사항이 다르고, 따라서 그 요구사항에 맞는 기술또한 다 다르다. 따라서 관계형 모델과 비관계형 모델이 지속적으로 공존할 것으로 보인다. 

## The Object-Relational Mismatch 

요즘 프로그래밍의 패러다임은 대부분 객체지향 프로그래밍을 따른다. 따라서 관계형 모델링으로 데이터를 저장한다면, 데이터베이스에 저장된 table, row, column등을 객체로 전환하는 layer가 필요하다. 전환이 필요하기 때문에, table에 저장된 데이터가 프로그래밍 언어로 표현된 값과 일치하지 않을 수 있고, 이를 `impedance mismatch`라고 부른다. 해당 단어는 전기 회로에서 자주 사용되는 표현인데, 전류의 흐름을 방해하는 값이라고 보면된다. 데이터가 프로그래밍 언어로 완전히 represent될 수 없는 경우를 뜻하는 것 같다. 따라서 Object-relational mapping(ORM)이라는 기술을 사용한다. 데이터베이스에 저장된 데이터를 프로그래밍 언어가 사용할 수 있도록 변환하는 과정이라고 보면 된다. 

링크드인 프로필을 관계형 데이터베이스로 표현하는 방법은 다양하다. 

1. 전통적인 SQL 모델은, 경력, 학력, 연락처 등을 모두 다른 테이블에 저장하고, `users`라는 사용자 테이블과 `foreign key`를 사용해서 연결시키는 것이다. 
2. SQL 모델이 발전하면서, 여러 값을 하나의 column에 넣을 수 있다. 즉 경력, 학력, 연락처 등을 하나의 컬럼에 저장하는 것이다. 
3. 마지막으로 경력, 학력, 연락처 등의 정보를 json이나 xml로 변환해서 document형식으로 저장하는 방법이 있다. 

MongoDB같은 document-oriented 데이터베이스의 경우 json형식으로 데이터를 저장할 수 있다. 많은 개발자들이 json 형식으로 데이터를 저장하는 것이 위에서 언급한 `impednance mismatch`를 줄일 수 있는 방법이라고 말한다. 하지만 json format도 그 자체의 문제가 있긴 하지만, **locality**라는 측면에서는 확실히 우위를 갖는다. 이력서에 관한 정보를 관계형 데이터베이스에서 불러오려면 join을 한다던지 등으로 인해 쿼리 수가 많아질 수 있는데, json포맷은 그냥 한번만 불러오면 된다는 장점이 있다. 

## Many-to-One and Many-to-Many Relationships

이력서에서 경력을 입력할 때, 회사의 위치를 입력하는 것이 일반적이다. 하지만 서울시 강남구, 판교 등을 free text로 입력하는 것보다, 드랍다운에서 리스트를 제공하는 것이 아래와 같은 장점이 있다. 

1. 스타일이나 오타를 방지한다. 예를들면 사람들은 서울시 강남구대신 강남, 서울강남구, 서울시강남이라고 입력할 수 있는데, 이를 방지할 수 있다. 
2. 데이터의 정확성을 높힐 수 있다. 예를들면 중구는 어디에나 있다. 당신은 어디 중구에서 오셨나요?
3. 업데이트가 간편해진다. 
4. 위치나 언어에 따른 localization이 수월해진다. 
5. 검색에 용이하다. 

id와 같은 고유값을 사용하는 것도 일종의 추상화이다. id 1번에 해당하는 인문학이라는 값은 어딘가에 저장되어있고, 사용자들은 id만을 사용해서 데이터에 접근하는 것이다. 또한 이 값은 사용자들에게 큰 의미가 있는 숫자는 아니기 때문에, 바뀔 필요가 없다는 장점이 있다. 사람들에 의미가 있는 인문학이라는 값은, 나중에 학계의 발전으로 세분화가 된다던지 등으로 인해 변경될 수 있지만, id는 변경될 필요가 없기 때문에 데이터베이스에 일관성을 보장할 수 있고, 중복으로 값이 입력되는 것도 방지할 수 있다. 이렇게 데이터 중복을 막는 것을 `정규화`라고 부른다. 

하지만 데이터의 정규화는 `many-to-one`이라는 관계를 필요로 한다. 서울시에는 다양한 사람들이 일을 하고 있고, 많은 사람들이 특정한 산업에 종사하는 것과 같은 관계를 나타낸다. 하지만 이것은 document model에 적합하지 않다. 관계형 모델에서는 join이 쉽기 때문에, 각 row를 id로 나타내서 사용하는것이 편리하지만, document model에서는 join 잘 이루어지지 않는 경우가 많기 때문에 적합하지 않다. 특히 join-free한 doucment model이 서비스 초반에는 적합할 수 있지만, 서비스가 확산되면서 데이터들이 관계를 갖는것이 더 적합해지는 순간이 올 수 있다. 

위에서 언급한 링크드인 이력서를 예로 들면, 학교나 경력에 대해 홈페이지를 추가한다거나, 주소, 팀 등을 추가한다면, free text로 입력되었던 값들이 변경되어야 한다. 그럴 경우 `free text`를 `entity`로 변경해야한다. 처음부터 모델링을 관계형으로 가져갔다면?? 서비스를 확산한다고 하더라도, 수정해야 할 사항들이 줄어든다는 장점이 있다. 그리고 이 관계가 점점 더 복잡해진다면 `many-to-one`으로 이루어진 관계들이 `many-to-many`로 확산되게 된다. 

## Are Document Databases Repeating History? 

`many-to-many`로 이루어진 관계형 데이터베이스는 join을 쉽게 할 수 있도록 지원한다. 하지만 document-based model은 `one-to-many`는 쉽게 표현할 수 있지만, `many-to-many`와 같은 관계를 데이터베이스에서 어떻게 표현할 수 있을 지 지속적으로 고민하고 있다. 과거에도 IBM에서 만든 IMS라는 데이터베이스에 이런 문제가 있었는데, 당시 개발자들은 `many-to-many`에 대해 join을 지원하지 않자, 데이터를 중복 저장하는 비정규화를 통해 이 문제를 해결했다. 

이에대한 해결책으로 나온 것이 관계형 모델과, 네트워크 모델이다. 

#### The network model

위에서 언급한 IMS는 `hierarchial`모델이었는데 tree구조로 이루어져있고, 하나의 record는 단 하나의 parent를 갖는다. network model은 이와 다르게, 하나의 record가 여러 parents를 가질 수 있다. 예를 들면 강남구라는 부모를 가진 record가 있다면, 강남구에서 일하는 직장인들이 해당 record 아래에 자식 records로 붙는 것이다. 두 record들은 관계형 모델처럼 foreign key를 사용하지 않고, 프로그래밍 언어의 pointer와 유사한 방식으로 연결된다. 하나의 record에 접근하려면, root부터 시작해서 이 pointer들을 계속 따라가야 하고, 이것을 `access path`라고 부른다. 

`access path`를 가장 간단한 방식으로 표현하자면, root부터 linked list로 연결할 수 있다. 하지만 `many-to-many`관계가 나타내는 데이터는, 특정 record에 접근하기 위해 댜앙한 access path들을 사용할 수 있을 것이다. 따라서 개발자들은 이 access path들을 관리하고 가장 적합한 access path를 제공해야 한다. 실제로 하나의 record가 여러 parent records를 가지고 있다면, 이것들을 모두 기억해야하는 문제가 있었고, manuaul하게 access path를 정할 수 있게 했지만, 해당 쿼리는 복잡하고 유연하지 못하다는 단점이 있었다. 

#### The relational model

network model과 반대로 relational model은 데이터를 모두 tuple들을 모아둔 table로 관리한다. 따라서 nested된 데이터도 없었고, 복잡한 access path를 따라가지 않고도 원하는 record에 접근이 가능하다. 원하는 row에 쉽게 접근할 수 있고, column과 key를 추가해서 더 빠르게 조회도 가능하다. 그리고 foreign key관계에 대해 크게 고민하지 않고 table에 row를 추가할 수 있다는 장점이 있다. 

관계형 데이터베이스는 쿼리 옵티마이저가 쿼리를 어떻게 하는 것이 가장 효율적일지 결정한다. MySQL에서는 실행계획을 통해서 이것을 미리 확인할 수 있다. network model의 효율적인 access path는 개발자가 찾아서 정해줘야하지만, relational model의 최적화는 쿼리 옵티마이저가 담당한다. 또한 데이터 쿼리 효율을 향상 시키려면 서비스 상황에 맞게 index를 추가하면 된다. 쿼리 옵티마이저 자체는 상당히 복잡하지만, 여기서 중요한 점은, 쿼리 옵티마이저를 한 번만 빌드하면 모든 어플리케이션이 그것을 사용해서 쿼리를 최적화 할 수 있다는 것이다. 

#### Comparison to document databases

document 데이터베이스는 hierarchial 모델과 유사하게. 데이터를 nested record로 저장한다. 이력서를 예로들면, 경력, 학력, 연락처 등의 정보를 별도의 테이블에 저장하지 않고, 하나의 record에 저장한다. 

## Relational vs Document Databases Today

관계형 데이터베이스와 document 데이터베이스를 비교할 때 fault-tolerance, concurrency등 고려할 점이 많다. document model은 schema가 자유롭다는 장점이 있고, locality로 인해 성능이 더 좋다는 장점이 있다. 관계형 모델은 join을 지원하고, `many-to-one`, `many-to-many`와 같은 관계를 잘 나타낸다는 장점이 있다. 

#### Which data model leads to simpler application code? 

만약 데이터가 document스타일로 저장된다면, document 모델을 활용하는 것이 좋다. document와 같은 구조를 굳이 테이블로 나눈다면, 복잡한 스키마와 코드가 생긴다는 문제점이 있다. document 모델을 활용하면 nested된 record에 직접적으로 접근할 수 없다는 단점이 있지만, 그 nested가 너무 깊지 않다면 크게 문제되지 않는다. document model은 join이 어렵지만, 서비스 특성에 따라 이것도 크게 문제가 되지 않을수도 있다. 

하지만 서비스가 `many-to-many`관계를 많이 사용한다면, document model을 사용하는 것이 불리하다. 비정규화를 통해 join을 최소화 할 수 있겠지만, 비정규화를 할 경우 consistency를 유지하는 것이 어렵다. 데이터베이스에 요청을 여러번 보내서 join과 유사한 효과를 낼 수는 있지만, 어플리케이션이 복잡해지고, 데이터베이스에서 join을 하는 것보다 느린 경우가 더 많다. 특정 데이터 모델이 코드를 더 간단하게 작성할 수 있다고 말 할 수는 없지만, 본인의 서비스 특징에 적합한 모델을 선택하는 것이 좋다. 

#### Schema flexibility in the document model 

document 데이터베이스와 관계형 데이터베이스의 json은 schema를 강제하지 않는다. 관계형 데이터베이스의 xml은 가끔 schema validation을 필요로 한다. no schema는 해당 document에 어떤 값이든 추가될 수 있다는 것이고, read할 때 사용자가 꼭 특정 값을 불러온다는 보장이 없다. 

따라서 document 데이터베이스들은 **schemaless**라고 불린다. 하지만 이는 정확하지 않다. schema를 강제하지 않을 뿐 schema가 아예 없는 것은 아니기 때문이다. 더 정확한 용어는 **schema-on-read**이다. 데이터를 읽어올 때 schema가 정해진다는 것이다. 이와 유사하게 관계형 데이터베이스는 **schema-on-write**이라고 불린다. 

schema-on-read는 프로그래밍 언어에서 dynamic type checking과 유사하고, schema-on-write은 static type checking과 유사하다. type checking의 장단점과 유사하게 schema방식의 차이에도 장단점이 있다. 

schema를 변경하는 것은 시간이 오래 걸리고 많은 downtime을 소요한다고 알려져있다. 하지만 꼭 그렇지는 않은데, 테이블을 전부 복사하는 MySQL을 제외하면, 일반적으로 `ALTER TABLE` command는 10ms내외로 처리되기 때문이다. 또한 `UPDATE` 명령어는 모든 row들을 찾아서 처리해야하기 때문에 어떤 데이터베이스든 오래걸린다. 만약 무슨 이유에서든 각각의 document가 다양한 schema를 갖는다면 schema-on-read를 사용하는 것이 더 좋다. 하지만 모든 document가 정해진 schema를 따라야 한다면 관계형 데이터베이스를 사용하는것이 좋다. 

#### Data locality for queries

document는 일반적으로 하나의 json이나 xml로 인코딩되어 하나의 긴 string으로 저장된다. 만약 웹사이트처럼 document전체를 접근해야한다면, 이런식으로 데이터를 저장하는 것이 좋다. 만약 데이터가 여러 테이블에 흩어져있다면 많은 index lookup을 한 후에 join을 해서 처리해야 하는데, disk에 접근을 더 많이해야해서 시간이 오래 걸린다. 

하지만 이 locality가 주는 장점은 하나의 document내에서 꽤 많은 데이터에 접근하는 경우이다. 왜냐하면 document의 작은 부분만 읽어온다고 할지라도, document전체를 불러온 다음에 parsing해야하기 때문이다. 업데이트 할 때도 마찬가지이다, document 전체를 다시 쓰는 일이 발생한다. 따라서 document는 가능한 작게 유지하는 것이 좋고, document 자체의 크기를 늘리는 write는 될수록 자제해야 한다. 

서로 연관된 데이터를 locality의 목적으로 묶는 것은 document 모델만 지원하는 것이 아니다. 예를들면 구글의 Spanner 데이터베이스는 테이블의 row들을 부모테이블에 nested시켜서 관계형 데이터 모델과 유사한 locality 를 제공한다. 오라클은 multi-table index cluster table을 통해 이를 지원한다. 

#### Convergence of document and relational models 

MySQL을 제외한 관계형 데이터베이스들은 2000년대 중반부터 xml을 지원한다. 이를 통해 xml document를 수정할 수 있고, xml document를 쿼리할 때 index를 사용할 수 있다. 또한 json에 대해 유사한 기능을 지원하기 시작했다. 관계형 데이터베이스가 점점 document 모델의 기능을 지원하는 것이다. 이와 유사하게 document 데이터베이스도 관계형 데이터 마냥 join을 지원하고, MongoDB는 object마다 자동으로 고유값을 부여한다. 데이터 모델들이 다른 데이터 모델의 장점을 포용하기 시작했다고 불 수 있다. 

# Query Languages for Data

처음 관계형 모델이 나왔을 때, 선언적 쿼리 언어인 SQL이 같이 등장했다. 일반적인 쿼리 언어들은 명령적이었던 것과 비교했을 때 상당히 혁신적이었다. 여기서 다시 추상화의 개념이 등장한다. 예를들면 아래 쿼리를 실행했을 때 

```sql
SELECT * FROM animals WHERE family = 'Sharks';
```

개발자는 어떤 데이터를 불러올지를 SQL을 사용해서 전달할 뿐, 그 데이터를 어떻게 불러올지는 감춰져있다. 선언적 쿼리 언어는 훨씬 간단하고, 사용하기 쉽기 때문에 더 매력적인 것도 있지만. 위에서 언급한 추상화로 인해, 개발자가 사용하는 쿼리 양식을 변경하지 않고, 데이터베이스의 효율이나 성능을 향상시킬 수 있다. 또한 선언식은 순서를 중요하게 생각하지 않기 때문에, 데이터를 불러오는 순서에 있어 자유롭고, 병렬 처리가 가능하기 때문에 더 빠르게 read가 가능하다는 장점이 있다. 

## Declarative Queries on the Web

선언적 쿼리 언어의 장점은 데이터베이스에 국한되지 않는다. query를 날리기 쉬운 방식으로 클라이언트 사이드를 표현할 수 있다면, 웹 프로그래밍의 구조에서도 장점을 가질 수 있다. 

## MapReduce Querying 

`MapReduce`는 완전 선언형도 아니고, 완전 명령적인것도 아닌 그 중간 어딘가에 있다. 쿼리의 로직은 간단한 코드로 표현할 수 있는데, 이 코드를 프레임워크를 실행하면서 반복적으로 호출한다. [MapReduce](https://jasonkang14.github.io/hadoop/how-map-reduce-works)는 예전에 하둡을 공부하면서 접한적이 있다. 

`MapReduce`는 map과 reduce로 이루어져있는데, map은 데이터를 전처리하는 과정이라고 보면되고, reduce는 전처리된 데이터로 작업하는 과정이라고 보면된다. 각각의 과정은 **pure function**으로 이루어져야 한다. 즉 각 함수는 새로운 Query를 날리지 않고, input으로 들어온 데이터만 활용할 수 있으며, side effect를 가지면 안된다. 따라서 이 함수는 데이터베이스에 접근권한만 있다면 순서에 상관없이 어디서든 실행될 수 있다는 장점이 있다. 

하지만 MapReduce도 코드를 최대의 효율을 낼 수 있도록 작성해야 하는데, 이것은 일반적이 쿼리문을 하나 작성하는 것과 비교했을 때 훨씬 더 어렵다. 또한 선언적 쿼리 언어가 쿼리 옵티마이저를 활용하는데 더 유리하기 때문에, MapReduce 코드를 사용하는 것이 효율 측면에서 떨어진다. 이로 인해 MongoDB는 aggregation pipeline이라는 선언적 쿼리 언어를 지원하기 시작했다. SQL과 비슷하지만 document based의 특징에 맞게 json 포맷의 쿼리언어이다. NoSQL이 점점 SQL과 유사해진다는 것을 볼 수 있다. 

# Graph-Like Data Models

`many-to-many`관계는 다양한 데이터모델의 특징을 구분하는 중요한 요소이다. 만약 어플리케이션이 대부분 `one-to-meny` relationship을 갖거나, 서로 연관성이 전혀 없다면 document model이 제일 적합하다. 

하지만 `many-to-many`가 데이터에 많이 보인다면, 간단한 관계정도는 관계형 모델을 통해서 처리할 수 있다. 하지만 만약 `many-to-many`가 점점 더 복잡해진다면, graph를 활용해서 데이터모델링을 하는 것이 더 좋다. 

graph는 `verticies`와 `edges`로 구현된다. verticies는 nodes나 entities라고도 불리고, edges는 relationships나 arcs로 불린다. verticies는 각각의 record를 나타내고, edges가 그 record들이 어떻게 연결되는지를 나타내는 것 같다. graph로 모델링되는 데이터는 일반적으로 아래와 같다. 

1. social graph: 링크드인 처럼 1촌, 2촌, 3촌 등의 관계를 나타내는 경우
2. web graph: 웹페이지가 서로 어떻게 연결되는지
3. road or rail networks: 길이 서로 어떻게 연결되는지.

여기서 중요한 단어는 **연결**이다. 차량 네비게이션은 길이 graph의 형태로 저장되기 때문에, 현재 node로부터 목적지 node까지 가장 짧은 경로를 찾는다. 또한 verticies는 하나의 타입에 국한되지 않는다. 페이스북을 예로 들면 사람, 학교, 위치, 이벤트 등등의 정보를 모두 vertex로 관리해서 하나의 큰 social graph로 저장한다. 이에 따라 edge는 사람들간의 관계가 될 수도 있고, 어떤 행사에 누가 참여했는지도 될 수 있고, 다양한 연결성들을 나타낼 수 있다. 

graph에는 다양한 종류가 있지만 이 책에서는 `property graph`와 `triple-store`에 대해 논한다. 

## Property Graphs

property graph모델에서 각 vertex는 아래의 항목들로 이루어져 있다. 

1. 고유값
2. 밖으로 나가는 edge들 (outgoing edge)
3. 안으로 들어오는 edge들 (incoming edge)
4. key-value pair로 이루어진 값들

그리고 각각의 edge는 아래 항목들로 이루어져 있다. 

1. 고유값
2. 해당 edge가 시작하는 vertex (tail vertex라고도 부른다)
3. 해당 edge가 끝나는 vertex (head vertex)
4. 두 vertex의 관계를 설명하는 값
5. key-value pair로 이루어진 값들

graph는 vertex로 이루어진 테이블과 edge로 이루어진 테이블을 가지고 있다고 보면 된다. head와 tail vertex들은 각 edge에 저장되고, 하나의 vertex의 outoging edge와 incoming edge를 알고 싶다면, vertex 테이블에 쿼리를 날리면 된다. 

```sql
CREATE TABLE vertices (
  vertex_id integerPRIMARYKEY, 
  properties json
);

CREATE TABLE edges (
  edge_id integer PRIMARY KEY,
  tail_vertex integer REFERENCES vertices (vertex_id), 
  head_vertex integer REFERENCES vertices (vertex_id), 
  label text,
  properties json
);

CREATE INDEX edges_tails ON edges (tail_vertex);
CREATE INDEX edges_heads ON edges (head_vertex);
```

graph 모델에서 중요한 점들은 아래와 같다. 

1. 모든 vertex는 다른 vertex와 edge를 통해 연결될 수 있다. 
2. 하나의 vertex에서 outgoing edge와 incoming edge를 사용해서 graph를 탐색할 수 있다. 
3. 다양한 관계들을 설명하는 **label**을 활용해서 하나의 그래프에 다양한 정보들을 저장할 수 있다. 

이로 인해 graph modeling은 유연하게 데이터를 저장할 수 있다. 

## The Cyper Query Language

`cypher`는 [Neo4j](https://neo4j.com/)를 위해 만들어진 `property graph`의 선언적 쿼리 언어이다. 아래 예제에서는 각 vertex는 이름을 갖고, arrow notation을 사용해서 edge를 표현한다. 

```sql
CREATE
  (Asia:Location  {name:'Asia',        type:'continent'}),
  (ROK:Location   {name:'South Korea', type:'country'  }),
  (Seoul:Location {name:'Seoul',       type:'city'    }),
  (Kang:Person    {name:'Jason Kang' }),
  (Seoul) -[:WITHIN]->  (ROK)  -[:WITHIN]-> (Asia),
  (Kang)  -[:BORN_IN]-> (Seoul)
```

이제 이 graph에 저장된 데이터를 활용해서 다양한 값들을 불러올 수 있다. 한국에서 미국으로 건너간 이민자들을 찾아온다거나, `BORN_IN`이라는 edge를 사용해서 `KOR`에 거주하는 사람들의 정보를 가져온다던지 등이다. 

```sql
MATCH
(person) -[:BORN_IN]->  () -[:WITHIN*0..]-> (kr:Location {name:'South Korea'}),
(person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'}) RETURN person.name
```

위 쿼리는 아래와 같은 방식으로 읽어낼 수 있다. 

1. `person`은 다른 vertex와 연결된 `BORN_IN`이라는 outgoing edge를 가지고있다. 해당 vertex에서 `WITHIN`이라는 edge를 찾아서 원하는 vertex를 찾는 것이다. 
2. 그리고 같은 `person` vertex는 `LIVES_IN`이라는 outgoing edge도 가지고 있는데, 그 edge를 따라가서 원하는 vertex를 찾은 후 필요한 값을 리턴하는 방식이다. 

지금은 사람에서 시작했지만, location vertex에서 시작해서 incoming edge를 따라갈 수도 있다. 

## Graph Queried in SQL

graph를 관계형 구조에 저장한다면, 조금 어렵긴하지만 SQL을 사용해서 쿼리할 수 있다. 관계형 데이터베이스는 각 테이블이 foreign key로 연결되어있기 때문에 쿼리에서 어떻게 값을 join할지 미리 알 수 있다. 하지만 graph의 경우 원하는 vertex를 찾기 위해 많은 edge들을 훑어내야한다. 따라서 join을 어떻게 할지 미리 알 수 없다. 

위의 예제에서 `LIVES_IN`이라는 edge를 예로 들면, `LIVES_IN`이 동과 연결되어있는지, 구와 연결되어있는지, 도시인지 나라인지 알 수 없다. 만약 사용자가 원하는 vertex와 연결된 edge가 없다면 다시 찾아야한다는 문제점이 있다. 위에서 작성한 `:WITHIN*0`와 같은 notation이 이 문제를 해결한다. 사용자가 원하는 vertex를 찾을 때까지 여러번 훑으라는 뜻이다. 재귀와 비슷한 원리라고 보면 된다. 

## Triple-Stores and SPARQL

triple-store model은 property graph model의 vertex와 edge를 다를 용어로 나타낼 뿐 매우 비슷하다. triple-store model은 데이터를 `(subject, predicate, object)`와 같은 3가지 묶음으로 저장한다. 예를 들면 `(jason, likes, python)`과 같은 느낌이다. `subject`는 `vertex`와 유사한 개념이고, `object`는 아래 두가지 중 하나이다. 

1. string이나 number와 같이 원시 데이터 타입으로 이루어진 값: `predicate`와 `object`는 key-value pair로도 설명할 수 있다. `(jason, age, 20)`이라는 값이 있다면 `{"age" : 20}`과 같다. 
2. 또다른 vertex. `predicate`는 `edge`와 같은 역할을 하고, `subject`는 tail vertex가 된다. `(jason, marriedTo, sarah)`와 같은 값이 있다면, jason과 sarah는 vertex이고 marriedTo가 edge가 된다. 


semantic web이 triple-store와 연결되어 설명되는 경우가 많다. semantic web은 웹 프로그래밍에서 웹사이트 구조에 따라 적합한 태그를 선택해야하고 등등에 관한 개념인데, [W3School의 HTML Semantic Elements](https://www.w3schools.com/html/html5_semantic_elements.asp)에서 매우 자세히 설명한다. 2000년대 초반에 유행하던 개념인데, 웹사이트의 구조를 통일하면서 여러 웹사이트에 있는 데이터들이 만약 같은 html tag안에 있다면, graph마냥 하나로 엮으려는 시도였던 것 같다. 

Resource Description Framework의 약자이다. 다양한 웹사이트에서 일정한 형식으로 데이터를 publish하기 위해 처음 고안되었다고 한다. xml형식으로 작성될 수도 있다고 한다. 웹에서 데이터를 주고 받기 위해서 만들어졌기 때문에, triple store에서 언급한 subject, predicate, object를 url을 통해서 나타낸다. 

또한 triple store를 위해서 SPARQL이나 Datalog와 같은 쿼리언어를 사용해서 쿼리가 가능하다.

