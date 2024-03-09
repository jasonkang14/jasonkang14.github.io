---
title: Oozie로 hadoop task 관리하기
date: "2022-09-18T19:41:37.121Z"
template: "post"
draft: false
slug: "/hadoop/managing-hadoop-tasks-with-oozie"
category: "Hadoop"
tags:
  - "Hadoop"

description: "Oozie로 hadoop task를 실행해본다"
---

[Oozie](https://oozie.apache.org/)는 hadoop workflow를 관리할 수 있는 스케쥴러이다. 따라서 hadoop task를 돌리고, 스케쥴링 할 수 있으며, periodic job들을 처리할 수 있다. 

xml로 작성된 workflow를 통해 다양한 action들을 엮을 수 있는데, MapReduce, Hive, Pig,  Sqoop, 등등을 순차적으로, 그리고 병렬적으로 처리할 수 있다. 일반적인 oozie의 workflow는 아래와 같다.

![oozie-workflow](https://i.imgur.com/xYsP57c.png)

각각의 영역(?)을 node라고 부른다. `start` node와 `end` node는 말 그대로 시작과 끝을 나타낸다. `fork` node는 다음 action들--여기서는 pig와 sqoop job--을 시작하라는 명령을 하고, `join`은 dependency들--여기서는 pig와 sqoop job--이 완료되면 다음 action을 시작하라는 명령을 한다. 

위 workflow를 xml로 보면 아래와 같다. 

![oozie-workflow-in-xml](https://i.imgur.com/4xERcTR.png)

각 xml tag를 보면 `name`과 `to`가  있는데, `name`은 해당 node에 부여된 고유값이고, `to`는 지금 `node`가 끝나면 어떤 `node`를 실행할 것인지를 나타낸다. 예를들어

```xml
<join name="joining" to="hive-node" />
```

의 경우에는, 해당 join node의 고유값은 `joining` 이고, 해당 node가 잘 끝나면 `hive-node`로 이동하라는 뜻이다. 위의 workflow의 diagram과 일치한다. 

예제에 나온 node들 이외에 switch-case를 사용해서 선택을 하도록 할 수도 있다.
```xml
<workflow-app name="foo-wf" xmlns="uri:oozie:workflow:0.1">
    ...
    <decision name="mydecision">
        <switch>
            <case to="reconsolidatejob">
              ${fs:fileSize(secondjobOutputDir) gt 10 * GB}
            </case>
            <case to="rexpandjob">
              ${fs:filSize(secondjobOutputDir) lt 100 * MB}
            </case>
            <case to="recomputejob">
              ${ hadoop:counters('secondjob')[RECORDS][REDUCE_OUT] lt 1000000 }
            </case>
            <default to="end"/>
        </switch>
    </decision>
    ...
</workflow-app>
```

Oozie workflow를 설정하고 구동하기 위해서는 HDFS에 디렉토리를 만들고, 해당 디렉토리에 `workflow.xml`을 작성하면 된다. `job.properties`에 `workflow.xml`에 필요한 변수들을 저장하고, 다양한 workflow들에서 사용하게 할 수도 있다.

Oozie에는 `coordinator`라는게 존재하는데, 말 그대로 oozie workflow를 관리하는 것이다. 하나의 coordinator에 cronjob이나 interval로 oozie workflow들을 돌릴 수 있다. 

```xml
<coordinator-app name="hello-coord" frequency="${coord:days(1)}"
                  start="2009-01-02T08:00Z" end="2009-01-04T08:00Z" timezone="America/Los_Angeles"
                 xmlns="uri:oozie:coordinator:0.1">
  <controls>
    <timeout>10</timeout>
    <concurrency>${concurrency_level}</concurrency>
    <execution>${execution_order}</execution>
    <throttle>${materialization_throttle}</throttle>
  </controls>      
  <datasets>
    <dataset name="din" frequency="${coord:endOfDays(1)}"
            initial-instance="2009-01-02T08:00Z" timezone="America/Los_Angeles">
      <uri-template>${baseFsURI}/${YEAR}/${MONTH}/${DAY}/${HOUR}/${MINUTE}</uri-template>
    </dataset>
    <dataset name="dout" frequency="${coord:minutes(30)}"
            initial-instance="2009-01-02T08:00Z" timezone="UTC">
      <uri-template>${baseFsURI}/${YEAR}/${MONTH}/${DAY}/${HOUR}/${MINUTE}</uri-template>
    </dataset>
  ...
```

최상단의 부모 tag가 `workflow`에서 `coordinator`로 다른 것을 확인할 수 있다.
그리고 이 coordinator를 모아둔 `bundle`도 있다. 

```xml
<bundle-app name=[NAME]  xmlns='uri:oozie:bundle:0.1'> 
  <controls>
    <kick-off-time>[DATETIME]</kick-off-time>
  </controls>
  <coordinator name=[NAME] >
    <app-path>[COORD-APPLICATION-PATH]</app-path>
    <configuration>
      <property>
        <name>[PROPERTY-NAME]</name>
        <value>[PROPERTY-VALUE]</value>
      </property>
      ...
    </configuration>
  </coordinator>
	
</bundle-app>
```

마찬가지로 최상단의 부모 tag가 `bundle`이고, 여기에는 여러 `coordinator`들이 들어갈 수 있다.

다른 Workflow와 비교하자면, oozie는 사실상 거의 사용되지 않는 것 같다.
[Github Star로 보는 workflow들의 인기](https://star-history.com/#argoproj/argo-workflows&apache/airflow&mlflow/mlflow&spotify/luigi&apache/oozie&kubeflow/kubeflow&Date)

oozie는 사실상 인기를 얻었던 적도 없다. 
그리고 구글은 [oozie에서 airflow로 migration하는 중](https://conferences.oreilly.com/strata/strata-eu-2019/cdn.oreillystatic.com/en/assets/1/event/292/Migrating%20Apache%20Oozie%20workflows%20to%20Apache%20Airflow%20Presentation.pdf)이라고 한다.

