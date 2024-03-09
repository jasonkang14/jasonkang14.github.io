---
title: HDFS가 데이터를 저장하는 방식
date: "2022-10-03T14:35:37.121Z"
template: "post"
draft: false
slug: "/hadoop/how-hdfs-handles-data"
category: "Hadoop"
tags:
  - "Hadoop"

description: "HDFS가 node failure와 data loss에 대비하는 방법"
---

udemy강의를 들으면서 [HDFS에 대해 한 번 정리한 적](https://jasonkang14.github.io/hadoop/hdfs)이 있다. 
이번에는 [Hadoop - the Defnite Guide](https://www.amazon.com/Hadoop-Definitive-Guide-Tom-White/dp/1449311520)를 읽으면서 조금 더 구체적으로 HDFS에 대해 알아보도록 한다. 

# HDFS의 정의
HDFS는 `저렴한 하드웨어`를 사용해서 `큰 파일`을 `streaming을 통한 접근`이 가능하도록 저장한다. 

하나씩 살펴보면. 
1. 저렴한 하드웨어 
    - 하둡은 고성능의 하드웨어를 필요로 하지 않는다. 
    - 저렴한 하드웨어들은 클러스터 안의 node 에서 불량이 발생할 가능성이 높다.
    - 하둡은 이러한 불량이 발생했을 때 사용자들이 불편함을 느끼지 못하도록 설계되어 있다. 

저렴한 하드웨어가 정확히 뭔지 찾아보았다 

![hardware-spec](https://i.imgur.com/jIHi2bN.png) 

출처: [Hadoop Cluster Hardware Recommendation](https://docs.informatica.com/data-engineering/data-engineering-integration/h2l/1415-tuning-and-sizing-guidelines-for-data-engineering-integrati/tuning-and-sizing-guidelines-for-data-engineering-integration--1/sizing-recommendations/hadoop-cluster-hardware-recommendations.html)       

공식문서에 따르면 [RAM 4GB도 충분하도록 설계되었다](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html#The+Persistence+of+File+System+Metadata)고 한다     

2. 큰 파일
    - 테라바이트, 심지어 페타바이트 크기의 파일들도 저장함

3. streaming을 통한 접근
    - HDFS는 한번 write하고 여러번 read하는 패턴을 따른다. 
    - 일반적으로 데이터는 한 번 write하면 다양한 사람들이 read해서 분석하기 때문이다. 
    - 데이터 분석은 모든 node를 필요로 하기 때문에, 첫번째 node를 얼마나 빨리 읽어내느냐보다 모든 데이터들을 얼마나 빨리 읽어내느냐가 중요하다. 

### <ins>따라서 아래와 같은 상황에서는 HDFS가 적합하지 않을 수 있다.</ins>
1. 데이터를 읽어들이는데 10ms정도 소요되는 경우 (매우 짧다는 뜻)
2. 작은 크기의 파일 여러개를 읽어들이는 경우
    - HDFS의 namenode가 데이터가 datanode들에 어떻게 저장되어 있는지에 대한 metadata를 메모리에 저장해서 가지고 있는데, 파일이 여러개면 메모리가 엄청 커야하는 문제가 있음
3. write가 빈번한 경우

# HDFS Concepts

### Blocks
HDFS는 데이터를 `128MB 사이즈의 block` 단위로 저장한다. 저장되는 파일의 크기가 128MB보다 작은 경우에는 디스크에서 128MB를 모두 차지하지 않는다.

일반적으로 disk 에서 block의 크기는 512bytes 수준이기 때문에 128MB가 너무 크다고 생각할 수도 있다. HDFS는 데이터를 찾는 시간을 줄이기 위해 큰 block 사이즈를 선택했다. 

책에 나와있지는 않지만 하둡의 등장 배경이 read가 오래 걸리기 때문에 분산저장을 한다고 했는데, 어차피 read하는 속도는 100MB/s 수준이니까, 분산저장되는 node의 숫자를 줄여서 빠르게 찾아내서 전송하는 것을 택한 것 같다. 
128MB는 default이고, 더 크게 설정하는 경우도 많다고 한다.

Block Abstraction이 가져오는 장점은 여러가지가 있는데
1. 파일이 디스크보다 커도 된다는 것
    - 하나의 파일의 block들이 같은 클러스터에 저장될 필요가 없기 때문에 어떠한 용량의 파일도 저장이 가능함
2. Abstraction을 파일 단위가 아니라 block단위로 하면서 시스템이 더 간단해 진다는 것
    - block은 용량이 고정되어 있기 때문에 디스크에 몇 개의 block이 저장될 수 있는지 계산하기 쉽고
    - permission과 같은 파일의 기타 metadata는 block과 같이 저장될 필요가 없기 때문
    - 다른 metadata는 다른데서 관리함 -> 파일 저장이 쉬워진다는 뜻인 것 같음
3. replication에 유리하다는 것 
    - block이 corrupt되는 것과 각종 에러에 대비하기 위해 block은 일반적으로 3개씩 replicate되어 저장됨
    - 특정 block에 접근할 수 없게 되면, replicate된 다른 block에 접근할 수 있음 
    - **사용자는 이 사실을 모름** <<<<< 이 말을 정말 자주함

### Namenodes and Datanodes
HDFS는 master 역할을 하는 `namenode`와 worker 역할을 하는 `datanode`로 이루어져 있음

`namenode`는 `namespace image`와 `edit log`의 형태로 filesystem tree와 각 파일들의 metadata를 저장한다. 그리고 특정 파일의 block들이 어떤 `datanode`에 저장되어 있는지에 대한 정보도 가지고 있다. 하지만 `datanode`의 정보는 memory에서 관리한다. HDFS가 재가동되면, `namenode`는 `edit log`를 기반으로 만들어진 `namespace image`에서 block들의 정보를 가져오고, 빠지는 정보들만 `datanode`와 소통해서 다시 채운다. `edit log`는 `namespace image`가 업데이트 될 때마다 버려진다 

`datanode`는 block들을 저장하고, 요청이 들어오면 그 block들을 client에 전달한다. 그리고 주기적으로 `namenode`에 자신들이 저장하고 있는 block 정보들을 전달한다.

`namenode`가 HDFS내에 파일이 어떻게 저장되어있는지에 대한 정보를 가지고 있기 때문에, `namenode`가 날아가면 시스템을 다시 구축할 수 없다. 그래서 하둡은 `namenode`가 내려가는 것에 대비해서 두가지 방식을 구현한다.(뒤에 나오는 HA까지 하면 사실 3가지)

1. secondary namenode
    - 주기적으로 `edit log`가 너무 커지기 전에 `namespace image`를 merge함
        - 도커에서 이미지 커밋하는거랑 비슷한 것 같음
    - `namespace image`를 merge하는데 CPU를 많이 사용하기 때문에 일반적으로 primary namenode와 다른 기기에서 구동됨
    - 주기적으로 merge가 이루어지기 때문에 primary node와 완전히 똑같지는 않음 
    - 따라서 primary node가 완전히 죽으면 data loss는 불가피함 
    - primary node가 죽으면 NFS에 백업된 metadata를 사용해서 secondary node를 primary로 전환해서 구동하는 것이 일반적임

2. filesystem metadata를 백업하는 것 
    - `namenode`는 persistent state(`edit log`)를 local disk나 remote NFS에 저장할 수 있다. 
    - secondary namenode와 다르게, 죽으면 백업된 metadata를 사용해서 기존의 primary를 다시 올린다는 것 같다

### Block Caching
일반적으로 `datanode`는 disk에서 block을 읽어오는데, 자주 접근하는 block들은 `off-heap block cache`에 저장할 수 있다. default는 하나의 `block`을 하나의 `datanode`의 메모리에 cache하는 것이다(파일마다 설정을 다르게 할 수는 있음). `MapReduce`나 `Spark` JobScheduler들은 cache를 활용해서 빠른 read를 통해 효율을 높일 수 있다. cache를 원하는 클라이언트는 `cache directive`를 `cache pool`에 추가해서 어떤 파일을 얼마나 오랫동안 `cache`할지 지정할 수 있다. 

![how-hdfs-cache-works](https://i.imgur.com/DgUmbuy.png)

`cache directive`는 cache에 저장하고자 하는 경로이다. cache는 재귀형식이 아니라서 해당 경로에 있는 파일들만 cache된다. 하위 디렉토리의 파일들은 cache에 저장되지 않는다 `cache pool`은 `cache directive`와 permission을 모아둔 그룹정도로 이해하면 된다. `cache pool`은 resource 관리도 가능한데, pool에 모인 directive들의 최대 용량을 제한할 수 있다. HDFS 내에 따로 cache를 위해 따로 배정된 메모리가 있는데, 디폴트는 최대로 다 쓰는건데, 설정해서 줄일 수 있다.

### HDFS Federation
`namenode`가 파일시스템에 대한 metadata를 메모리에 저장하기 때문에, 엄청 많은 파일들을 저장하는 클러스터의 경우에는 memory 크기 때문에 scalability가 떨어질 수 있다. 그래서 하둡 2.X대 버전부터 HDFS Federation이라는 컨셉을 도입했는데, 클러스터가 namenode를 추가해서 scaling할 수 있도록 하는 방식이다. 예를들면 namenode 1은 /user에 저장된 파일들의 metadata를 관리하고 namenode 2는 /share에 저장된 파일들의 metadata를 관리하는 식.

그렇다면 사용자가 HDFS에 접근할 때 어떤 namenode를 통해 데이터를 불러와야 하는지에 대한 정보를 어딘가 또 저장해야하는데, 이거는 client에서 file-path와 `namenode`를 mapping한 테이블을 저장해서 가지고있다.

### HDFS High Availability
`namenode` 하나를 active standby로 두고, primary로 구동되는 것이 죽으면 바로 교체하는 방식. secondary namenode를 사용할 경우 다시 올라갈 때까지 데이터 write가 불가하다던지 하는 문제가 있는데 이를 해결한다. active standby는 primary와 storage와 `edit log`를 같이 사용한다. 따라서 primary가 죽어서 active standby가 구동될 때, 이 `edit log`를 활용해서 기존에 돌고있던 primary와 같은 configuration을 가지고 돌아가게 된다. 

위에서 언급한 `storage`는 두가지 옵션이 있다. 
1. NFS filer
2. Quorum Journal Manager(QJM)
    - QJM은 HDFS 구현을 위해 존재한다. 
    - 목적 자체가 highly available edit log를 active standby에 제공하기 위한 것이다. 
    - 그래서 이걸 추천한다
    - QJM은 `journal node`들로 이루어져 있는데, `ediit log`를 이 `journal node`들에 저장한다. 
    - ZooKeeper와 유사하게 3개의 `journal node`들이 있다.

`namenode`가 metadata를 disk가 아니라 memory에 저장하기 때문에, datanode는 primary namenode와 active standby 모두에게 block의 위치를 전달해야 한다. active standby도 metadata를 메모리에 저장하고 있기 때문에, 구동하는데 걸리는 시간이 매우 짧다는 이점이 있다. 

하지만 실제로는 조금 오래 걸리는 것처럼 느껴질 수 있는데, 이는 시스템이 `정말로 primary namenode가 내려갔는지`를 판단하는데 오래 걸리기 때문이다. 예를 들면 네트워크 문제로 파일을 못 불러오는 것인데, primary namenode가 내려갔다고 판단해서 active standby를 구동하는 것은 낭비가 될 수 있다. 

primary가 내려갔는지 판단해서 secondary를 구동시키는 시스템을 `failover system`이라고 한다. 그리고 `ZooKeeper`같은 것을 사용해서 하나의 namenode만 올라간게 맞는지 확인하고, secondary나 standby가 올라가는 준비중일때, 기존에 구동되고있던 primary namenode가 시스템을 corrupt하는 것을 방지하는 것을 `fencing`이라고 한다. QJM은 한번에 하나의 `namenode`만 `edit log`를 작성할 수 있도록 하는데, 기존 active namenode가 클라이언트의 read request를 수용할 수도 있다. 따라서 SSH fencing등의 코드를 작성해서 방지하는 것이 중요하다. 

NFS filer는 QJM과 달리 한 번에 하나의 `namenode`만 `edit log`를 업데이트 할 수 있는 설정이 없기 때문에 fencing은 더 빡세게 작성해야 한다. `fencing`에는 다양한 방법들이 있는데, `namenode`가 NFS에 접근할 수 있는 권한을 제거하거나, 네트워크 설정을 통해 해당 port로의 접근을 제한하는 것들을 포함한다. 최악의 경우에는 기존 namenode의 전원을 꺼버릴 수도 있다. 

client-side도 에러가 발생할 수 있는데, 이건 library를 사용해서 처리한다.

# The Command-Line Interface
하둡은 개발자들이 좋아하는 CLI를 제공한다. 아래 예제는 HDFS를 하나의 machine에서 구동한다는 전제를 가지고 한다. 
```bash
# configuration

fs.defaultFS = hdfs://loaclhost/
dfs.replication = 1 # default는 3인데 예제가 기계 하나에서 돌기때문에 replication을 방지함
```

로컬에서 하둡으로 파일 옮기기 `hdfs://localhost`는 생략해도 된다
```bash
hadoop fs -copyFromLocal input/docs/quangle.txt hdfs://localhost/user/tom/quangle.txt
hadoop fs -copyFromLocal input/docs/quangle.txt /user/tom/quangle.txt
hadoop fs -copyFromLocal input/docs/quangle.txt quangle.txt
```

하둡에서 로컬로 옮기기
```bash
hadoop fs -copyToLocal quangle.txt quangle.copy.txt
```

하둡에 디렉토리 생성
```bash
hadoop fs -mkdir books
hadoop fs -ls .
```
![hadoop-fs-ls](https://i.imgur.com/2I1mdWw.png)
<br />

파일 정보를 리눅스랑 유사하게 읽어올 수 있고, 사용자와 권한 사이에 숫자는 replication 숫자를 나타낸다. 
하둡에서 파일 권한은 리눅스랑 유사하게 read(r), write(wr) execute(x)로 구성된다. read, write는 말 그대로이고, execute는 무시되는데, HDFS 안에서 파일을 실행하는 것이 불가능하기 때문이다. 
각각의 파일과 디렉토리는 `owner`, `group`, `mode`가 있다. `mode`는 `owner`의 권한이고, `owner`는 `group`에 속해있다. 

# Hadoop Filesystems
파일시스템 종류는 다양하다. HDFS를 꼭 사용할 필요는 없다. 
![hadoop-filesystem-table](https://i.imgur.com/2I1mdWw.png)
이거 이외에 Azure와 [OpenStack](https://www.openstack.org/) Swift도 존재한다 - Swift == Openstack equivalent of AWS S3 

### Interfaces
하둡은 Java로 구현됐기 때문에, 상당수의 interaction이 Java API를 통해 이루어진다. Java API이외에 하둡과 소통할 수 있는 다른 방법들을 소개한다. 

#### HTTP
`WebHDFS` protocol이 제공하는 HTTP REST API를 사용해서 하둡에 접근할 수 있다. HTTP Client는 native Java Client보다 느리기 때문에 큰 용량의 데이터를 transfer할 때는 가급적 사용하지 않는 편이 좋다. HTTP를 사용해서 hadoop에 접근할 수 있는 방법은 두가지가 있는데, 
1. HDFS daemon이 HTTP request를 직접 처리하는 것과
2. `DistributedFileSystem API`를 사용해서 proxy를 통해 접근하는 방법이 있다. 

![how-to-access-haddop-via-http](https://i.imgur.com/0LZICwc.png)

#### C
`libhdfs`라는 C library를 사용해서 하둡에 접근할 수 있다. `libhdfs` 라이브러리는 Java의 `FileSystem` interface와 매우 유사하다.

#### NFS(Network FileSystem)
HDFS를 로컬 클라이언트의 filesystem에 NFSv3 gateway를 사용해서 mount시킬 수 있다. read와 write가 가능하다. append를 통한 파일 수정은 가능한데, HDFS가 파일의 끝에만 write만 가능하도록 디자인 되어있기 때문에 랜덤으로 수정하는 것은 불가능하다. 

#### FUSE
Filesystem in Userspace의 약자. Unix filesystem처럼 user space에 구현된 filesystem이다. (kernerl에 파일을 write할 수 있는 기능??) HDFS를 local filesystem처럼 mount시킬 수 있는 방법이다. NFS가 FUSE보다 안정적이기 때문에 가능하면 안쓰는걸 추천한다.
# Java 예제
DataFlow에 예제 코드로 추가했다

# Data Flow
### File Read
![how-to-read-from-hdfs](https://i.imgur.com/Ii0s5lg.png)

1. `FileSytem` object에서 `.open()`을 호출한다

```java
public class FileSystemCat {
    public static void main(String[] args) throws Exception {
        String uri = args[0];
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(uri), conf); // 이 파일시스템을 사용해서
        InputStream in = null;
        try {
            in = fs.open(new Path(uri));  // 여기 호출! FSDataInputStream을 return함
            IOUtils.copyBytes(in, System.out, 4096, false);
            // IOUtils는 하둡에서 제공
            // 4096은 buffer size이고 
            // true/false는 데이터 read가 끝나면 연결을 끊을지/말지 결정하는 것
        } finally {
            IOUtils.closeStream(in);
        }
    }
}
```

2. `DistributedFileSystem`이 `namenode`에 접근해서 요청하는 파일의 block 위치를 읽어온다
    - `namenode`는 처음 block들이 어떤 `datanode`에 저장되어 있는지 `client`와 가까운 순서대로 sorting해서 정보를 제공한다.
    - `MapReduce`등의 케이스로 client가 datanode라면 client는 local datanode에서 데이터를 read한다. 
    - `DistributedFileSystem`은 `FSDataInputStream`을 return한다

3. `client`가 `FSDataInputStream`에 `.read()` 호출

```Java
public class FSDataInputStream extends DataInputStream
    implements Seekable, PositionedReadable {
}

public interface Seekable {
    void seek(long pos) throws IOException;
    long getPos() throws IOException;
}

public interface PositionedReadable {
    public int read(long position, byte[] buffer, int offset, int length) throws IOException;

    public void readFully(long position, byte[] buffer, int offset, int length) throws IOException;

    public void readFully(long position, byte[] buffer) throws IOException;
}
```

`read()`를 호출하면, position에서부터 데이터를 read하기 시작함. 읽어드린 bytes를 `int`로 리턴한다. `readFully()`호출하면 다 read함
`DFSInputStream`이 가지고 있는 client로부터 가장 가까운 datanode의 주소를 사용해서 `client`는 `datanode`와 연결한다.

4. `DFSInputStream`이 `datanode`에 `.read()`호출해서 데이터를 read함. 
5. `block`의 끝에 도달하면, `DFSInputStream`은 connection을 닫고, 다음 `block`을 읽어들일 `datanode`를 찾는다. 
    - 이 과정은 client에게 안내되지 않는다(예를들면 3번 데이터노드에서 데이터를 읽기 시작합니다 등의 메세지 없음)
    - `block`들은 순서대로 읽어진다
    - `DFSInputStream`이 `datanode`로 연결하고, 이후에 `block`을 더 read해야하면 `namenode`와 소통해서 다음 `datanode`의 위치를 받아온다. 
    - `datanode`로부터 데이터를 읽어올 때 에러가 발생하면 가장 가까운 `datanode`로부터 데이터에서 데이터를 읽으려고 시도한다.
    - 또한 실패한 `datanode`를 기억해서, 해당 `datanode`로는 다시 read request를 보내지 않는다. 
    - 문제가 있는 `block`은 `namenode`에게 알려준다

![distance-between-datanodes](https://i.imgur.com/xcIUF0l.png)

6. read가 끝나면 `FSDataInputStream`에 `.close()`호출
    - `IOUtils`는 하둡에서 제공
    - [API 문서](https://hadoop.apache.org/docs/r2.6.3/api/org/apache/hadoop/io/IOUtils.html)가 나름 친절한편

HDFS에서는 클라이언트가 `namenode`가 제공하는 정보를 바탕으로 `datanode`에 직접적으로 연결한다. 따라서 traffic이 각각의 datanode로 분산되기 때문에 여러 사용자들이 동시에 접근해도 문제없이 요청을 처리할 수 있다는 장점이 있다. 

### File Write
![hdfs-write](https://i.imgur.com/02uyjyh.png)

1. `DistributedFileSystem`에 `.create()`호출
```Java
public class FileCopyWithProgress {
    public static void main(String[] args) throws Exception {
    String localSrc = args[0];
    String dst = args[1];

    InputStream in = new BufferedInputStream(new FileInputStream(localSrc));

    Configuration conf = new Configuration();
    FileSystem fs = FileSystem.get(URI.create(dst), conf);
    OutputStream out = fs.create(new Path(dst), new Progressable() { // 여기 호출
        public void progress() {
            System.out.print(".");
        }
    });

    IOUtils.copyBytes(in, out, 4096, true);
}
```

2. `DistributedFileSystem`이 `namenode`와 소통해서 새로운 파일을 생성하려고 함. 
    - `namenode`는 client가 생성하고자 하는 파일이 존재 하지 않는 것을 확인하고,
    - client가 write permission이 있는지 확인하고, 
    - 새로운 파일이 생성된다는 것을 기록한다. 
    - 어떤 `datanode`에 write를 시작할지도 정보를 전달함

3. client가 write를 시작한다
    - `DFSOutputStream`이 파일을 packet으로 나눔
    - internal queue인 `data queue`와 `ack queue`에 저장한다. 

4. `DataStreamer`는 `namenode`로 부터 데이터를 저장할 `datanode`를 받아와서 `data queue`에서 packet을 꺼내서 `datanode`에 전달한다
    - replication level에 따라 `datanode`는 packet을 write한 후에 relaction할 다음 `datanode`들로 packet을 전달한다.

5. `DFSOutputStream`은 `ack queue`를 활용해서 `data queue`에서 나간 packet들이 `datanode`에 제대로 write됐는지 관리한다
    - 모든 `datanode`들에 packet이 write된 것을 확인하면 `ack queue`에서 해당 packet을 제거한다.(replication 확인)
    - `datanode`가 패킷을 제대로 write하지 못한 경우 아래와 같이 해결한다
        - write pipeline이 닫힌다
        - `datanode`들이 남김없이 packet을 write하기 위해 `ack queue`에 남아있는 패킷들은 `data queue`의 앞으로 옮겨진다.
        - write에 실패한 `datanode`의 `block`들은 제거된다. 
        - 정상적으로 작동하는 `datanode`는 `block`의 상태를 `namenode`에 전달하고, 이 `datanode`를 기반으로 새로운 pipeline이 작동하기 시작한다. 
        - replication이 replication level만큼 이루어지지 않은 경우 추가 replication을 진행한다. 

6. write가 끝나면 client는 `.close()`를 호출한다.
    - datanode pipeline에 packet이 남아있다면 flush하고
    - `DFSOutputStream`으로부터 `ack` signal을 기다린다.

7. `namenode`에 write가 끝났다고 전달한다.
        
### Replication
`namenode`는 replication을 저장할 `datanode`를 선택할 때 write bandwidth와 read bandwidth를 고려한다. 모든 replica들을 하나의 `datanode`에 저장하면 write bandwidth에 유리하지만, 데이터가 정상적으로 relicate되었다고 볼 수 없다. 그렇다고 replica를 아예 다른 데이터 센터에 저장하면 relication은 완전하겠지만 write bandwidt측면에서 불리하다. 

하둡은 일반적으로 첫번째 relica를 client와 같은 node에 저장한다. client가 cluster밖에서 돌고있으면 랜덤으로 node를 선택한다. 두번째 replica는 다른 rack에 저장되고, 세번째 replica는 두번째 replica와 같은 rack의 다른 node에 저장된다. 


### Coherency Model
coherency는 read와 write를 쉽게하기 위해 고안되었다. 
file을 write했다면 파일을 읽어드릴 수 있어야한다.
```Java
Path p = new Path("p");
fs.create(p);
assertThat(fs.exists(p), is(true));
```

그런데 write된 파일을 읽어드릴 수 없는 경우에는 length가 0로 나타난다
```Java
Path p = new Path("p");
OutputStream out = fs.create(p);
out.write("content".getBytes("UTF-8"));
out.flush();
assertThat(fs.getFileStatus(p).getLen(), is(0L));
```

적어도 하나의 block이 write되어야 첫번째 block이 read가능하다. 다양한 사용자가 동시에 접근한다고 할 때 같은 값을 read하는 것을 보장하기 위한 것으로 보인다. 
HDFS는 모든 buffer들을 flush하기 위해 `hflush()` method를 제공한다. `hflush()`가 성공했다고 return하면, 모든 `datanode`들에 파일이 정상적으로 write됐다는 것을 의미한다. 

```Java
Path p = new Path("p");
FSDataOutputStream out = fs.create(p);
out.write("content".getBytes("UTF-8"));
out.hflush();
assertThat(fs.getFileStatus(p).getLen(), is(((long) "content".length())));
```

`hflush()`가 return하는 것은 디스크에 성공적으로 block들을 write한 경우가 아니라 `datanode`의 메모리에 정상적으로 write된 것을 나타낸다. 따라서 disk에 저장하기 전에 전원이 꺼진다던지 등의 문제가 생기면 dataloss가 발생한다. 조금 더 확실하게 write를 보장하기 위해 `hsync()`라는 method를 사용할 수 있다. disk와 sync하는 것을 뜻한다. 

`hflush()`나 `hsync()`를 통해 write를 guarantee하지 않으면, data loss가 발생할 수 있다. 따라서 적당한 시점에(?) 호출해줘야한다

### Parallel Copying with distcp
지금까지 소개한 HDFS 패턴은 모두 single-thread이다. `distcp`를 사용하면 파일을 병렬적으로 처리할 수 있다. 
```bash
hadoop distcp file1 file2
hadoop distcp dir1 dir2
```

`distcp`의 장점은, dir2가 존재하지 않는다면, dir2를 새로 생성하고 그 내부에 dir1의 내용을 복사한다는 것이다. dir2가 이미 있다면, 그 안에 dir1의 내용을 append할 수도 있고(dir2/dir1), `-overwrite` option을 사용하면 dir1의 내용을 사용해서 dir2를 덮어쓸 수도 있다.

`distcp`는 `MapReduce`로 구현되었다. 따라서 복사는 MapReduce의 map job으로 이루어져서 클러스터 내에서 병렬로 구동된다. reduce는 일어나지 않고(데이터 변형이 필요 없으니 그런 것 같음) `distcp`는 각각의 `mapper`에 동일한 양의 데이터를 주려고 한다. 최대 20개의 mapper가 사용되는데 설정하면 더 사용할 수도 있다. 

일반적으로 hdfs 클러스터 간 데이터를 옮기기 위해 주로 사용한다. 
```bash
hadoop distcp -update -delete -p hdfs://namenode1/foo hdfs://namenode2/foo
```

`-delete` 옵션을 사용하면 destination에서 해당 위치에 이미 존재하는 것들을 모두 삭제하고 덮어쓰는 것을 뜻한다. 
`-p` 옵션은 permissoin, block size, replication등의 파일 정보를 변경하지 않고 그대로 전달하는 것을 뜻한다.
만약 data를 전송하려는 두 hdfs 클러스터가 다른 버전의 hdfs를 사용한다면 `webhdfs`를 사용하면 된다. 

```bash
hadoop distcp -update -delete -p webhdfs://namenode1/foo webhdfs://namenode2/foo
```

### Balancing HDFS Cluster
복사를 맘대로 하면 cluster내에 하나의 rack에 데이터가 엄청 모인다거나 하는 불상사가 발생할 수 있다. HDFS는 데이터가 균일하게 분산되어야 잘 작동하기 때문에, 이걸 주의해야한다. 