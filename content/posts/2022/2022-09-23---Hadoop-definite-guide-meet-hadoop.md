---
title: Hadoop - the Definite Guide - Meet Hadoop
date: "2022-09-23T19:41:37.121Z"
template: "post"
draft: false
slug: "/hadoop/meet-hadoop"
category: "Hadoop"
tags:
  - "Hadoop"

description: "하둡의 등장 배경과 MapReduce"
---

[Hadoop - the Defnite Guide](https://www.amazon.com/Hadoop-Definitive-Guide-Tom-White/dp/1449311520)라는 책을 같이 스터디 하시는 분들과 읽기로 했다. 유데미에서 강좌를 다 듣고 책으로 넘어왔는데, 책 제목부터 `THE` definite guide이다. 마치 교본같은 느낌이다. 

[하둡](https://hadoop.apache.org/)은 데이터 분산시스템이다. 하둡이 등장한 이유 그리고 하둡을 사용하는 이유를 이해하려면 분산 시스템의 필요성을 깨달아야 한다. 저자가 말하는 하둡의 등장 배경은 데이터의 양이 많아졌기 때문이다. 짧은 인터벌로 데이터를 읽어들이는 IoT 센서부터 시작해서, 우리가 매일 스마트폰으로 찍는 사진들의 데이터도 엄청나다. 많아지면 잘 저장하고 잘 읽어들이면 되는데, 저자의 말에 따르면 하드 디스크 용량이 증가해서 `write capacity`는 급증했으나, 이를 `read speed`는 그 발전을 따라가지 못했다는 문제가 있다. 

1990년에 드라이브는 약 1370 MB의 데이터를 저장할 수 있었는데, 이것을 읽어들이는 속도는 4.4MB/s 수준이었다. 따라서 모든 데이터를 읽어들이는데 약 5분정도 소요가 됐다고 한다. 그런데 20년이 넘은 지금--책이 2015년도에 출판됨--하드 디스크는 약 1TB인데 읽어들이는 속도는 100MB/s에 불과해서 전체 데이터를 읽어들이는데 2.5시간이 걸린다. 드라이브 용량은 커졌지만, 전체를 읽어들이는데는 오히려 더 긴 시간이 걸리는 것이다. 그리고 위에서 언급한 `write capacity`는 최대 용량이다. 드라이드의 용량이 증가했을 뿐 `write speed`역시 현저히 떨어진다. 

이 문제를 해결할 수 있는 가장 간단한 방법은, 여러 디스크들로부터 동시에 데이터를 읽어들이는 것이다. 하나의 드라이버에 100개의 데이터를 저장하기 보다는, 100개의 드라이버에 한 개 씩 저장한다면, write나 read 모두 빨라질 것이다. 비용이 비싸다고 생각하는 사람들도 있을 수 있지만, 데이터를 처리/관리/유지/보수 등 하는 사람들은 시간이 더 중요한 사람들이기 때문에 크게 문제가 되지 않을 것이라고 말한다. 

그런데 이렇게 여러 디스크에 데이터를 쪼개서 저장할 경우 `hardware failure`가 발생할 수 있다. 디스크가 하나면 그거 하나만 잘 관리하면 되는데, 디스크가 여러개인 경우 모두 다 잘 관리해야한다. 어쩌면 사고 확률이 100배 증가한다고 볼 수 있다. 디스크가 고장났을 때 `data loss`를 방지하기 위해서 데이터 복제(`replication`)을 선택하는 경우가 많은데, 이것은 RAID(Redundant Array of Independent Disks)가 처리하는 방식이고, 하둡은 조금 다르다--나중에 설명한다.

데이터를 나누어 저장하는 것의 또 다른 문제는, 나눠서 저장한 데이터들을 다시 합쳐야 한다는 것이다. 여러 곳에 나누어 저장된 데이터를 하나로 합치는 것은 생각보다 어렵다. 하둡에서는 [MapReduce](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)가 이 역할을 담당한다. `MapReduce`는 데이터를 디스크로부터 읽어와서 `key : value` pair로 전환한다. 이름에서 알 수 있듯이 `MapReduce`는 `Map`과 `Reduce`로 이루어져 있다. `Map`은 데이터를 전처리 하는 과정이고, `Reduce`는 실제로 데이터를 조작하는 과정이라고 보면 된다. 

`MapReduce`는 이 `job`을 `batch`단위로 처리한다. 따라서 `ad hoc` query등에 매우 유리하다. 하지만 `batch` 단위로 데이터를 처리하기 때문에 시간이 오래 걸린다는 단점이 있다. 따라서 `MapReduce job`은 offline으로 처리하는 것이 일반적이다. 

하둡은 이를 극복하기 위해 다양한 툴들을 `hadoop ecosystem`에 추가했는데, [HBase](https://hbase.apache.org/)라는 key-value store를 추가했고, 리소스를 관리하는 [YARN](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YARN.html), 디스크에 저장된 데이터를 처리하는 `MapReduce`와 달리 메모리단에서 데이터를 관리하는 [Spark](https://spark.apache.org/), 데이터 스트리밍을 가능하게하는 [Storm](https://storm.apache.org/), 검색기능을 가진 [Solr](https://solr.apache.org/) 등이 추가됐다. 이것저것 추가됐지만 `MapReduce`는 여전히 batch process를 처리하는데 독보적이다.

`MapReduce`를 활용한 데이터 처리와 관계형 데이터베이스를 비교하자면, `MapReduce`가 훨씬 더 큰 용량의 데이터를 처리할 수 있다. 하지만 `MapReduce`는 주로 read를 담당하기 때문에, write와 read가 빈번하게 일어나고, transaction을 지원하는 관계형 데이터베이스와 달리, read는 여러번 할 수 있으나 write는 한 번만 가능하고, transaction은 지원하지 않는다. 하지만 요즘 관계형 데이터 베이스가 점점 발전하면서 하둡과의 경계가 모호해진다고 한다. 

그래도 하둡이 관계형 데이터베이스와 비교했을 때 우위에 있는 점은, 정형화 되지 않은 데이터를 처리할 수 있다는 것이다. 웹서버 로그처럼 다양한 구조의 데이터를 유연하게 처리할 수 있고, 분산 저장하기 때문에 scalable하며, 더 빠르게 데이터를 처리할 수 있다. 하둡은 `data locality`를 중요하게 생각하는데, 다양한 노드에서 `MapReduce` job들이 돌아간다 하더라고, 가능한 그 데이터를 가진 노드에서 `MapReduce` job을 돌리려고 노력한다. 또한 특정 job의 실패 여부를 파악해서 reduce job을 다시 돌리는 등의 Relability를 확보한다. 

책에서는 `MapReduce`를 활용해서 기온 데이터를 처리하는 것을 보여준다. Java, Ruby, Python등의 언어로 `MapReduce` job을 작성할 수 있다. 다양한 job들을 병렬적으로 돌릴 수 있기 때문에 상대적으로 빠르고--근데 유데미 강의 보면 다른 빠른것들 많음--reliable하다는 특징이 있다. 또한 `Map`에서 데이터를 순서대로 정리하기 때문에, 더 효율적으로 `Reduce`할 수 있고, `Combiner`라는 기능을 통해 효율을 높일 수 있는다. 그리고 `MapReduce`는 batch processing전문이긴 하지만 `.jar` 파일을 사용해서 streaming도 할 수 있다고 하는데, streaming을 위해 만들어진 툴들이 있는데 굳이 `MapReduce`를 사용헤서 스트리밍을 할 필요가 있는지는 의문이다. `MapReduce`에 대한 자세한 내용은 추후에 별도의 섹션에서 다룰 예정이다. 