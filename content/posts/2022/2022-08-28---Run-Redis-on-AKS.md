---
title: Azure Kubernetes(AKS)에서 Redis 구동
date: "2022-08-28T19:41:37.121Z"
template: "post"
draft: false
slug: "/azure/redis-on-aks"
category: "Kubernetes"
tags:
  - "Kubernetes"
  - "Azure"

description: "Redis를 올려본다"
---

Azure Kubernetes(AKS)에서 redis pod를 쿠버네티스에 올리는 방법은 두가지이다. 
1. container registry에 redis image를 push하고 구동한다
2. redis관련 yaml 파일을 작성한다.

1번으로 하면 더 빠를 것 같다는 생각이 들었다. container registry에 push하면 해당 이미지를 바로 가져와서 redis pod를 올릴 수 있기 때문이다. 하지만 대부분의 쿠버네티스 설정들이 yaml 파일로 이루어져있기 때문에, yaml 작성에 익숙해질 필요가 있는 것 같아서 2번으로 진행하기로 결정했다. 데드라인만 맞추면 되는 거 아니겠나?!? 라는 생각으로 

쿠버네티스 홈페이지에 아주 친절하게 레디스를 위한 yaml 작성법이 나와있다. 그래서 yaml 작성법은 크게 공부가 되지 않았다. 시키는대로 따라서 작성해보도록 한다. 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
  namespace: CUSTOM_NAMESPACE
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    command:
      - redis-server
      - "/redis-master/redis.conf"
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: redis-config
        items:
        - key: redis-config
          path: redis.conf
```

`namespace` 설정을 빠트리면 default로 배정되기 때문에 꼭 작성해줘야한다. 

redis-config도 공식문서에서 제공해준다.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
  namespace: CUSTOM_NAMESPACE
data:
  redis-config: |
    maxmemory 1mb
    maxmemory-policy allkeys-lru    
```
maxmemory 설정이 필요한 이유는, explicit하게 지정해주지 않으면 redis가 메모리 사용량을 제한하지 않기 때문이다. 공식문서에는 2mb로 되어있는데 1mb도 충분할 것 같아서 1mb로 설정했다. maxmemory-policy설정이 필요한 이유는, default 값으로 두면 redis가 데이터를 스스로 지우지 않고 계속 가지고 있기 때문이다. `allkeys-lru`는 오래된 데이터부터 날리라는 뜻이다. FIFO!

이렇게 해서 pod를 올리면 구동은 되는데 다른 pod로부터 redis pod에 요청을 보낼 수 없다. 찾아보니 이는 `service`가 필요하기 때문이다. pod만 돌아가게 되면, pod가 죽고 다시 생성될 때 새로운 IP가 부여된다. 그렇다면 다른 pod들 또한 설정을 바꿔서 다시 구동해야 한다. service를 통해서 이 문제를 해결할 수 있는데, 각각의 service에 고정된 Cluster IP를 부여해서, pod가 죽었다가 살아나더라도 고정된 IP를 사용할 수 있도록 한다. 사실 이건 내가 겪는 문제와 크게 상관은 없는데, 무튼 service가 있어야지 pod로 연결을 할 수 있는 구조이다. 

그래서 pod를 설정한 yml 파일의 하단에 service 설정도 추가한다. 

```yml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: voice-safety-dev
  labels:
    app: redis
spec:
  selector:
    app: redis
  ports:
  - name: redis
    protocol: TCP
    port: 6379
    targetPort: 6379
```

service는 deployment center와 연동할 때 생성된 다른 `service.yml` 파일을 보고 작성했다. 여기서 주의할 점은 label과 selector가 기존에 작성한 redis설정과 동일해야 한다는 점이다. 이제 환경변수를 보면 `REDIS_HOST`라는 값을 사용해서 redis pod에 연결할 수 있다. 