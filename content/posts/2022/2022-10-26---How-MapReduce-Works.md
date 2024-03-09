---
title: MapReduce 작동방법
date: "2022-10-26T23:35:37.121Z"
template: "post"
draft: false
slug: "/hadoop/how-map-reduce-works"
category: "Hadoop"
tags:
  - "Hadoop"

description: "MapReduce는 어떻게 돌아가는가"
---

# Anatomy of MapReduce Job Run

MapReduce job은 단순히 `Job` 객체에서 `submit()`을 호출하면 된다. `waitForCompletion()`이라는 method도 있는데, 이 method는 중복으로 `submit()`되는 것을 방지하고, 이미 `submit()`됐다면 해당 `job`이 끝나기를 기다린다.

MapReduce job은 다섯종류의 참여자(?) 가 있다.

1. `client` - `MapReduce` job을 `submit()` 함
2. `YARN` resource manager - 클러스터 내에서 컴퓨터 리소스를 관리함
3. `YARN` node manager - 클러스터 내에서 컨테이너를 구동하고 모니터함
4. `MapReduce` application manager - `MapReduce` job을 돌리는 task들을 관리함
5. `HDFS` - `MapReduce`에서 사용되는 파일들을 저장/전달/공유함

![how-map-reduce-works](https://i.imgur.com/IINEWDw.png)

### Job Submission

`submit()` method는 내부의 `JobSubmitter`라는 인스턴스에 `Job` 을 생성한다. 그리고 생성된 `Job`에서 `submitJobInternal()`을 호출한다. 왜 `submitInternalJob()`이 아닌지 이해가 되지 않는다. job을 submit하고 나면, `waitForCompletion()`이 job의 상태를 주기적으로 확인한다. job이 성공적으로 마무리되면 job counter가 증가하고, error는 콘솔에 기록된다.

`JobSubmitter`는 아래와 같은 과정을 통해 `job`을 처리한다.

1. `YARN` resource manager에게 `MapReduce` job id로 사용할 새로운 application id를 요청하고,
2. job의 specification을 확인한다. output destination등의 조건을 충족하지 못하면 error로 기록된다.
3. job을 split한다(I/O section에서 설명된 것처럼). split에 실패할 경우 error로 기록된다.
4. job을 실행하기 위한 리소스를 job id의 이름을 가진 디렉토리로 복사한다. job도 replicate돼서 node manager가 job을 처리할 때 확인할 수 있는 파일의 수가 충분하도록 한다
5. `submitApplication()`을 호출해서 job을 resource manager에게 전달한다.

### Job Initialization

resource manager가 job을 받으면, resource manager는 해당 job을 `YARN`으로 전달한다. `YARN`은 컨테이너를 시작하고, resource manager는 해당 컨테이너에서 master process를 시작한다.

`MapReduce` job의 application master는 `MRAppMaster`라는 클래스를 가진 Java application이다. master는 job의 경과를 트래킹 하기 위한 객체들을 만들면서 job을 시작한다. 이 객체들을 통해서 job이 진행되면 경과를 보고받고, 완료되면 완료를 보고 받는다. master는 HDFS에서 split된 파일들을 가져와서, 각각의 split에 map task와 reduce task를 부여한다.

application master는 job을 어떻게 구동할지를 결정해야 한다. job이 작으면 그냥 같은 JVM에서 구동해도 된다. 그리고 병렬로 처리하면서 데이터를 주고 받는 것에 대한 오버헤드가 sequence로 처리하는 것보다 크다고 생각하는 경우에는 job이 크다고 해도 그냥 같은 JVM에서 실행하기도 한다. 일반적으로 mapper가 10개 미만이거나, reducer가 하나거나, 인풋 사이즈가 HDFS block보다 작으면 해당 job은 작다고 여긴다. 이런 경우를 `uber task` 라고 부른다. 해당 조건에 부합한다고 무조건 같은 JVM에서 실행하는 것은 아니고 `mapreduce.job.ubertask.enable == true`가 설정되어야 한다.

그리고 job을 본격적으로(?) 실행하기 전 application master는 `setupJob()`을 호출해서 해당 job의 최종 directory와 임시 파일들이 저장될 위치를 생성한다.

### Task Assignment

만약 job이 uber task로 구동될 수 없다면, application master는 job을 돌릴 컨테이너가 필요하다. 이 컨테이너들은 resource manager로부터 배정 받는다. mapper가 reducer보다 우선순위를 갖고 먼저 배정된다(순서대로 해야하니까 당연한 이야기).

reducer는 클러스터 안에 어디서 돌아가던 상관이 없다. 그런데 mapper는 가능한 data locality를 보장해야 한다. 최상의 시나리오는 split된 인풋이 있는 node에서 mapper job이 동작하는 것이다. 같은 node에서 작업이 어렵다면 적어도 같은 rack에서 이루어지는 편이 좋다.

application master는 각 task에 필요한 CPU와 메모리도 요청한다. default는 1개의 CPU core와 1024MB의 메모리이다.

### Task Execution

task를 실행할 컨테이너가 배정되면, application master는 node manager와 소통해서 컨테이너에서 task를 시작한다. task는 `YarnChild`라는 클래스를 가진 Java application이 구동한다. task를 시작하기 전에 task에 필요한 리소스들을 localize한다.

##### Streaming

![map-reduce-job-streaming](https://i.imgur.com/ava7Ra6.png)

streaming task는 input/output stream을 사용해서 Process와 소통한다. task가 작동하는 동안, Java process는 input key-value 쌍을 외부 process로 전달한다. 이 process는 사용자가 지정한 mapper나 reducer 함수에서 돌아간다. 그리고 돌아간 key-value pair는 다시 input으로 들어가게 된다.

### Progress and Status Updates

`MapReduce`는 오래 걸리는 `batch` job들이다. 오래 걸리기 때문에 사용자에게 진척도를 알려주는 것이 중요하다. mapper의 경우 input이 어느정도 처리되었는지를 나타내고, reducer도 조금 복잡하긴 하지만 얼마나 처리됐는지 나타낼 수 있다. 또한 task중에 mapper의 output이 write된 횟수 등의 다양한 이벤트 발생 빈도를 체크한다.

`MapReduce` task가 돌아가는 동안, 자식 process는 부모 application master와 `umbilical interface`를 통해 소통한다. task는 진행상태와 status를 application master에게 전달하고, application master는 3초마다 전체 진행상황을 확인한다.

`MapReduce` job이 진행되는 동안 클라이언트는 전체 진행 상태를 확인한다. 필요 시 `Job` instance의 `getStatus()` method를 호출해서 전체 상태를 확인할 수 있다.

### Job Completion

application master가 `MapReduce` job의 마지막 task가 끝났다는 알림을 받으면, application master는 job의 상태를 `successful`로 변경한다. 처음에 언급한 `waitForCompletion()` method가 기다리는 상태가 바로 `successful`이다. job이 마무리되면 application master와 task container들은 정리되고, 중간 결과물들은 삭제된다.

# Failures

job이 늘 성공하는 것은 아니다. bug도 있고, process가 꺼질 수도 있고, 기계적 결함이 있을 수도 있다. 하둡을 사용하는 이유 중 하나는, 하둡이 그런 실패들을 잘 처리해서 job이 잘 마무리 될 수 있게 해준다는 것이다.

### Task Failure

가장 빈번한 오류는 사용자가 mapper나 reducer에 작성한 코드가 `Runtime Exception`을 초래하는 것이다. JVM은 해당 exception을 report하고, master가 job을 끝내기 전에 부모 application에게 전달한다. error는 기록되고, applicatoin master는 task를 `failed`로 변경한다. 해당 컨테이너는 다른 task가 사용할 수 있게 된다. Streaming task의 경우, streaming task가 nonzero exit code로 마무리되면, `failed`로 기록된다.

갑자기 JVM task가 끝나버리는 경우도 있다. node manager는 process가 끝났다는 것을 확인하고, application master에게 알려서 상태가 `failed`로 변경되도록 한다.

진척이 없는 task들은 조금 다르게 관리된다. application master가 해당 task로 부터 진척도 업데이트를 받지 못한 것을 확인하면 해당 task를 `failed`로 처리한다. timeout은 일반적으로 10분인데 설정할 수 있다.

application master가 task가 `failed`됐다고 확인하면, task를 다시 reschedule한다. reschedule할 때 방금 fail 한 node manager는 기피한다. task를 4번 실패하면 재시도 하지 않고, 특정 task가 4번 실패하면 job전체가 Fail된다. task가 kill될 수도 있는데, 이는 `failed`와는 조금 다르다. killed되면 4번 실패한다는 count에 추가되지 않는다.

### Application Matser Failure

application master의 fail은 최대 2번까지 허용된다. 2번 실패하면 재시작 하지 않고 job은 fail된다. `YARN`은 `YARN` application master에 fail count를 설정하고, 각 application master들도 이 Fail count를 초과할 수 없다.

application master는 resource manager에게 주기적으로 heartbeat를 보낸다. application master가 fail하면 resource manager가 fail을 확인하고, application master의 새로운 instance를 새로운 컨테이너에서 시작한다. `MapReduce` application master의 경우 job의 진척상태를 확인하고, 거기부터 pickup해서 다시 시작할 수 있도록 한다.

클라이언트는 처음 요청할 때 Resource manager로부터 application master의 위치를 받아두고, 이 정보를 cache한다. 따라서 application master가 죽으면 job의 상태를 확인할 때 resource manager로부터 새로운 application master의 위치를 받는다. 늘 그렇듯 사용자는 이를 알 수 없다.

### Node Manager Failure

node manager가 fail하는 경우는 crash되거나, 매우 느리게 돌아가는 경우이다. node manager는 resource manager로 보내던 heartbeat를 멈춘다. resource manager는 fail을 확인하고 해당 node를 scheduling pool에서 제거한다.

application master는 특정 node manager가 fail 하기 전 해당 node에서 `successful`로 기록된 map task가 있다면, 그 job도 다른 node에서 다시 돌아갈 수 있도록 처리한다. mapper의 결과물이 그 node에 남아있다면 reducer가 접근할 수 없기 때문이다.

### Resource Manager Failure

resource manager가 fail하면 새로운 job이나 task가 새로 돌아갈 수 없다. resource manager는 single point of failure인데, 하드웨어가 고장날 가능성은 매우 낮고, 그럴 경우에는 모든 Job들이 fail되고 회복할 수 없다는 문제강 ㅣㅆ다.

High Availability를 구현하려면 resource manager를 병렬로 운용해야한다. 돌아가고있는 모든 application의 정보는 `ZooKeeper`나 `HDFS`에 저장된다. standby는 중요 정보들을 최대한 빨리 회복해야한다. node의 정보들은 heartbeat를 통해 다시 retrieve할 수 있기 때문에 따로 저장되지 않는다.

새로운 resource manager가 시작하면, application의 정보를 state store에서 불러오고, application master들을 컨테이너에서 시작한다. 이 전환은 failover controller에서 관리한다. default는 `ZooKeeper`이다. HDFS HA와 다르게 standalone으로 구현할 필요는 없다. manual하게 failover를 처리할 수도 있지만 권장하지 않는다.

# Shuffle and Sort

`MapReduce`는 reducer로 들어가는 모든 input들이 key순으로 정렬된다.

### The Map Side

mapper가 시작해서 output을 만들기 시작하면, 이 output은 단순히 disk에 기록되는 것이 아니다. buffering을 사용해서 메모리에 기록하고, 효율을 위해 pre-sorting을 실시한다.

![shuffle-and-sort-in-map-reduce](https://i.imgur.com/xlo9UjG.png)

각 mapper는 output을 write할 100MB사이즈의 circular 메모리 buffer를 갖는다. buffer가 80%차면 buffer에 기록된 내용들을 disk로 옮긴다. disk로 옮겨지는 동안 buffer에 다시 기록하고, disk로 옮겨지기 전에 buffer가 다 차버리면, mapper는 task를 멈추고 buffer가 비워질 때까지 기다린다.

disk에 write하기 전에 mapper들은 reducer로 보낼 것을 고려해서 데이터를 partition한다. 각 partition은 background thread에서 in-memory sorting을 실시하고, combine이 필요하다면 sort의 output에서 이루어진다. combine은 데이터를 더 compact하게 만들기 때문에 disk에 기록할 양이 줄어들게 되는 이점이 있다.

mapper의 output을 압축하는 것을 권장하는데, 더 빨리 write할 수 있고, disk 용량을 아낄 수 있고, reducer로 보내는 데이터의 크기가 줄어들기 때문이다.

### The Reduce Side

reducer는 mapper의 output중에 본인이 담당하는 partition을 찾아야 한다. 일반적으로 다양한 mapper의 output들을 사용해야 하기 때문에, mapper들이 output을 기록하면 reducer들은 복제하기 시작한다. 복제된 output들은 크기가 작다면 JVM memory에 저장되고, 너무 크면 disk에 저장된다. 복제되면 background thread에서 복제된 데이터들을 더 큰 규모의 파일들로 합친다. 이를 `copy phase`라고 부른다.

mapper의 output이 모두 복제되면, reduce task는 `sort phase`로 넘어간다. output들을 merge한 후에 sort의 순서를 유지한다. 이건 merge될 때 여러번 반복하는 것으로 보인다. 최종적으로 disk에 쓰고나서 reducer로 보내지 않고, merge된 파일들은 reducer로 바로 들어간다. reducer는 output을 바로 file system에 기록한다.

# Task Execution

사용자들은 configuration 설정을 통해 생각보다 많은 것들을 할 수 있다. streaming을 어떻게 처리할지, output을 어디에 저장할지 , task를 언제 구동할지, fail count는 몇으로 할지 등등을 따로 설정할 수 있다.
