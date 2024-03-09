---
title: Azure Kubernetes Deployment Center에서 환경변수 활용법
date: "2022-08-14T15:41:37.121Z"
template: "post"
draft: false
slug: "/azure/adding-kubernetes-secrets-to-deployment-center"
category: "Kubernetes"
tags:
  - "Kubernetes"
  - "Azure"

description: "Kubernetes Secret을 사용한 환경변수 적용"
---

Azure Kubernetes에서 Deployment Center라는 섹션을 통해 깃헙 레포와 쿠버네티스 클러스터를 연동하고 CI/CD를 구축하려고 한다. 단순한 연동을 통해서 서버 pod가 잘 올라가는 것은 확인했는데, 기존에 설정한 환경변수를 불러올 수 없는 문제가 있었다. 

기존에는 깃헙액션의 워크플로우를 사용할 때, `${{ secrets.ENVIRONMENT_VARIABLE}}` 과 같은 형태를 사용해서 도커 이미지를 빌드하고 컨테이너를 올렸기에, [이전 포스트](https://jasonkang14.github.io/azure/kubernetes-workflow-jobs)의 다양한 워크플로우에 같은 방식을 계속 시도했으나 실패했다. 

**Build and push image to ACR** 과정에서 github secrets에 있는 환경변수를 넣어주니, 이미지를 빌드할 때 빌드는 성공했지만 환경변수를 배포된 컨테이너에서 읽어올 수 없었다. 그래서 도커의 환경변수가 아니라 [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/)을 사용해보기로 했다. 

공식문서의 설명을 요약하면 아래와 같은 파일을 만들고
```yml
apiVersion: v1
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: { ... }
  creationTimestamp: 2020-01-22T18:41:56Z
  name: mysecret
  namespace: default
  resourceVersion: "164619"
  uid: cfee02d6-c137-11e5-8d73-42010af00002
type: Opaque
```

아래의 명령어를 적용하면 쿠버네티스 secret을 적용할 수 있다. 
```bash
kubectl apply -f mysecret.yaml
```

위의 `mysecret.yml`파일에 대해 설명하면, `data`가 내가 사용하고자 하는 secret, 즉 환경변수 들인데 base64로 인코딩 해서 넣어준 값이다. 
`kind`를 `Secret`으로 해주어야 Kubernetes Secret으로 인식하고 쿠버네티스 클러스터에서 사용할 수 있게 된다. 

base64로 인코딩 시 `-n` 옵션을 꼭 넣어줘야 하는데, 해당 옵션을 넣어주지 않으면 디코딩된 값에 `\n`이 들어가서 에러가 발생한다. 

```bash
echo -n 'KUBERNETES_SECRET' | base64
```

그리고 metadata 정보 중에서 `name`과 `namespace`가 중요하다. 별도의 설정을 하지 않는다면 같은 `namespace`내의 pod만 해당 secret에 접근이 가능하기 때문이다. `namespace`를 따로 지정해주지 않으면 `default`라는 `namespace`가 할당된다. Azure Kubernetes Deployment Center와 연동할 때 namespace를 입력하게 되어있는데, 여기에 `default`를 입력하지 않았다면, `namespace`를 지정해 줄 필요가 있다. 

기존 Azure 쿠버네티스 deployment center에서 생성해준 workflow 파일에서, `azure/k8s-deploy@v1.2`라는 workflow가 배포를 담당한다.

```yml
- uses: azure/k8s-deploy@v1.2
      with:
        namespace: CUSTOM_NAMESPACE
        manifests: |
          manifests/deployment.yml
          manifests/service.yml
        images: |
          DOCKER_IMAGE:${{ github.sha }}
        imagepullsecrets: |
          DOCKER_AUTH_SECRET
```

Azure 쿠버네티스의 deployment center와 깃헙 레포를 연동하면, `.github` 디렉토리 아래에 `workflow`디렉토리 말고, 프로젝트 root에 `manifests`라는 디렉토리를 생성해준다. 해당 디렉토리 아래에 `deployment.yml`과 `service.yml`이라는 파일이 있고, `azure/k8s-deploy` workflow에서 두 파일에 대해 `kubernetes apply -f FILENAME`을 실시한다.

따라서 `secret.yml`이라는 파일을 `manifests` 디렉토리 안에 추가하여 github secret으로 변수를 입력하려고 했으나, 환경변수는 자주 바뀔 일이 없으니 반복적으로 작업할 필요가 없다고 판단하여 일회성으로 업데이트했다. 아래와 같이 yml파일을 생성하고 `kubernetes apply` 명령어를 사용해서 secret을 적용한다. 

```yml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: CUSTOM_SECRET_NAME
  namespace: CUSTOM_NAMESPACE
data:
  CUSTOM_SECRET_KEY: YWRtaW4=
```

이제 pod에서 적용한 쿠버네티스 secret들을 사용할 수 있도록 `deployment.yml`파일을 수정해줘야한다. 

```yml
apiVersion : apps/v1
kind: Deployment
metadata:
  name: POD_NAME
spec:
    spec:
      containers:
        - name: POD_NAME
          image: IMAGE_NAME
          ports:
          - containerPort: 8000
          env:
            - name: ENVIRONMENT_VARIABLE
              valueFrom: 
                secretKeyRef:
                  name: CUSTOM_SECRET_NAME
                  key: CUSTOM_SECRET_KEY
                  optional: false
```

이제 깃헙 액션에서 워크플로우가 잘 돌아간 것을 확인하고, curl을 통해 간단하게 로그인만 테스트 해보기로 한다.
![login-success](https://i.imgur.com/zhQPQ8d.png)

잘 작동하는 것을 확인할 수 있다. 
