---
title: 하둡 클러스터 설정하기
date: "2022-11-02T23:35:37.121Z"
template: "post"
draft: false
slug: "/hadoop/setting-up-hadoop-cluster"
category: "Hadoop"
tags:
  - "Hadoop"

description: "Hadoop Cluster 세팅하는 방법에 대해 작성해본다"
---

`HDFS`, `MapReduce`, `YARN`을 하나의 머신에서 돌려도 되지만, 제대로 하둡을 사용하려면 복수의 node를 사용해야 한다. 하둡 크러스터를 구동할 때는 일반적으로 아래 3개의 방법을 사용한다.

1. Apache tarballs

   - `tar`를 사용해서 설치
   - flexibility는 있지만 할게 많다

2. Packages

   - RPM, Debian 패키지 사용
   - `tar`에 비해 filesystem layout 제공, 하나로 묶여서 테스트가 가능하다는 등의 장점이 있다.

3. Hadoop cluster management tools
   - Claudera, Ambari 등의 툴을 사용하는 법(udemy 강의에서 한 것처럼)
   - Web UI도 있고, HA 설정 등 딸려오는 것들이 많아서 사용하기 편하다

# Cluster Specification

앞에서 언급한 것 처럼 하둡은 저렴한 툴에서 구동할 수 있게 설계되어 있다. 2014년도 기준으로 아래와 같은 사양이면 충분하다.

1. hex/octo-core 3 GHz CPU 2개
2. 64 - 512 GB 메모리
3. 1-4 TB사이즈의 디스크 12-24개
4. Gigabit ethernet

참고로 하둡은 노드간 데이터 복제(Replication)을 실시하기 때문에 굳이 RAID(Redundant Array of Independent Disks)는 필요 없다고 한다.

# Cluster Sizing

처음에는 작게 10개정도의 node에서 시작해서 필요에 따라 사이즈를 키우면 된다. 로깅 등으로 인해 얼마나 큰 storage가 얼마나 자주 추가되어야 할지는 계산이 필요하다

### master node scenarios

클러스터 사이즈에 따라 namenode, secondary namenode, resource manager, history, server 등과 같은 master daemon을 돌리는 방법은 다양하다. 작은 클러스터의 경우 namenode와 resource manager를 같은 마스터 머신에 돌리는 것이 일반적이다. 하지만 클러스터가 커진다면 나누는 것이 좋다.

namenode는 메모리가 많이 필요하다. file과 block의 metadata를 메모리에 저장하기 때문이다. secondary namenode도 checkpoint를 만드는 경우 primary만큼은 아니지만 상당한 크기의 메모리를 필요로 한다. 따라서 기계의 메모리가 작다면 primary와 secondary를 같은 머신에 돌리지 못 할 수도 있다.

메모리 크기를 떠나서 High Availability를 고려하면 노드들을 따로 두는 것이 좋다. 같은 머신에서 primary 와 secondary를 돌리다가 죽어버리면 살릴 수가 없다.

# Network Topology

하둡 클러스터는 일반적으로 2단계의 network topology를 갖는다.
![hadoop-network-topology](https://i.imgur.com/8JuUWW8.png)

rack마다 30-40개의 서버가 있고, rack에는 10GB switch가 있다.

### Rack Awareness

하둡이 최대 퍼포먼스를 내려면, 네트워크 topology를 이해하는 것이 중요하다. single rack이면 딱히 할만한게 없는데, rack이 여러개라면, node를 rack과 mapping하는 과정이 필요하다. 하둡은 rack 내부의 node에서 `MapReduce` task를 옮기는 것을 선호하기 때문이다. `HDFS` 또한 replica를 만들 때 rack내부를 우선시한다.

# Cluster Setup and Installation

설정을 위한 각종 명령어들이 나열되어있다.

# Hadoop Configuration

하둡은 전체를 관리하는 configuration파일은 없고, 각각의 node별로 configuration을 관리한다. `dsh`, `pdsh`와 같은 툴을 사용해서 configuration들을 일치 시킬 수는 있다. 설정이 어렵거나 귀찮다면 위에서 언급한 cloudera나 ambari를 사용하는 것을 추천한다.

하둡 설정에 머신을 추가하는 경우, 머신의 사이즈가 다르다면 새로운 configuration이 필요하다. 이런 경우에는 각각의 머신을 `class`로 취급하고, 각 클래스마다 configuration을 다르게 관리하는 편이 좋다. `Chef`, `Puppet`, `CFEngine`, `Bcfg2` 와 같은 툴들을 사용하면 관리하기 편해서 사용을 추천한다.

# Environment Settings

하둡이 Java로 작성되어있기 때문에 Java설정을 해줘야하고, 1GB가 디폴트로 설정되어있는 memory heap size도 조절할 수 있다. 메모리를 많이 사용하는 `namenode`의 경우에도 일반적으로 1GB정도면 충분하다. 그밖에 log file의 위치화 ssh 설정 등도 할 수 있다.

### HDFS

우선 하나의 머신을 `namenode`로 지정해야 한다. `fs.defaultFS`에서 IP 주소를 담고, 별도로 설정하지 않는다면 8020 포트를 사용한다. 추가로 namenode가 persistent filesystem metadata를 어디에 저장할지, 저장하는 디스크의 갯수는 몇개인지, datanode들의 위치는 어디인지, secondary namenode는 어디에 있는지 등의 정보 또한 설정해야 한다.

### YARN

우선 하나의 머신을 `resource manager`로 지정해야 한다.

`MapReduce` job을 시작하면, 중간에 생성되는 데이터는 임시 로컬 파일에 write된다. 데이터 양이 생각보다 많을 수 있기 때문에 `yarn.nodemanager.local-dirs`를 설정 해줘야 한다. 그래야지 충분한 용량의 디스크 파티션을 사용할 수 있다. 이름이 `dirs`인 만큼 디렉토리를 여러개 넣을 수도 있다.

`YARN`은 application이 동작할 때 마다 각 태스크 별로 메모리를 요청할 수 있게 한다. node mangaer는 pool에서 메모리를 할당하기 때문에, 한 번에 돌아갈 수 있는 태스크의 수가 메모리 할당량에서 결정된다. 그리고 각각의 job에 메모리를 어떻게 할당할지도 결정해야 한다. `YARN`이 할당하는 container 사이즈 기반으로 가던지, Java 프로세스의 heap size 기준으로 갈 수도 있다.

virtual memory 또한 설정해줘야한다. virtual memory가 physical memory를 초과하는 경우 node mangaer는 process를 멈춰버린다. 각 job이 얼마나 많은 memory를 사용하는지를 확인하고, 관리하는 것이 좋다.

`YARN`에서 메모리를 관리하는 방법과 유사하게 각 application은 CPU도 요청할 수 있다. `MapReduce`는 각 mapper와 reducer에 몇 개의 core가 필요한지를 `mapreduce.map.cpu.vcores`, `mapreduce.reduce.cpu.vcores` 를 통해 설정할 수 있다. scheduling중에 core수는 tracking되지만, node manager는 구동중인 컨테이너의 cpu 갯수를 제한 하지는 않는다. 따라서 container는 이미 할당 받은 것 보다 더 많은 cpuf를 할당받을 수 있다.

# Security

하둡 초기는 다 같은 머신에서 돌아갔는데, 이제 네트워킹이 포함되기 때문에 보안에도 신경 써야 한다. 하둡은 기본적으로 authrization을 제공한다. 따라서 일단 모두 하둡 자체에는 접근이 가능하고, 어떤 데이터에 접근할 수 있는지 권한만 분리가 가능하다. authentication, 하둡 자체에 접근권한 제어,는 안되기 때문에 IP나 사용자 이름 등으로 제한할 수 있다.

이를 편하게 하기위해 `Kerberos`라는 것이 도입됐다. authentication을 하고, authorization을 받은 후에 하둡 서버에 접근하는 방식이다

![kerberos](https://i.imgur.com/gMVlMGL.png)

이외에 토큰을 상요할 수도 있고, cache등을 사용해서 보안을 강화할 수도 있다.
