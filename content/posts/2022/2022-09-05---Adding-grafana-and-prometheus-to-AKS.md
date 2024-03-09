---
title: Azure Kubernetes(AKS)를 Grafana로 모니터링
date: "2022-09-05T19:41:37.121Z"
template: "post"
draft: false
slug: "/azure/grafana-on-aks"
category: "Kubernetes"
tags:
  - "Kubernetes"
  - "Azure"

description: "Grafana로 AKS 모니터링"
---

글또콘에 다녀왔는데 같은 테이블에 앉은 분들이 [Grafana](https://grafana.com/)로 쿠버네티스 모니터링을 한다고 해서 연동해보았다. 서버 모니터링이 중요해서 툴들을 찾아야겠다 라고 고민하고 있었는데, 많이들 쓴다니까 이걸로 가보기로 했다. 그 팀들에서 얼마나 많은 기술 검토를 통해 이 툴로 정했을까를 생각하면 많은 시간을 아낀 것 같다. 

Azure는 문서가 잘 안되어있는데 검색을 잘 한건지 [Using Azure Kubernetes Service with Grafana and Prometheus](https://techcommunity.microsoft.com/t5/apps-on-azure-blog/using-azure-kubernetes-service-with-grafana-and-prometheus/ba-p/3020459)라는 문서를 찾았다. 디테일한 명령어에는 오류가 있지만 그래도 꽤나 정확하다. 

Grafana는 시각화를 통한 모니터링이 가능하고 [Prometheus](https://prometheus.io/docs/introduction/overview/)도 모니터링 툴인데 그라파나랑 같이 도는 것 같다. 연동을 해보니 helm으로 Prometheus를 설치하면 Grafana도 같이 돌아간다. 

Prometheus repository를 helm repository에 추가하고 update를 진행한다
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update
```

그리고 쿠버네티스에 올린다. 문서에는 `--create-namespace`라는 명령어가 있지만, 나는 이미 있는 namespace에 추가하는 것이기 때문에 그건 생략한다. 
```bash
helm install prometheus \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring
```
![prometheus-online](https://i.imgur.com/butuIrj.png)

아주 잘 올라간 것을 볼 수 있다. 

이제 port fowarding을 통해 로컬에서 확인해본다. 

```bash
kubectl port-forward -n CUSTOM_NAMESPACE prometheus-promethus-kube-prometheus-prometheus-0 9090
```
포트가 9090번인 이유는 default port이기 때문이다. 이는 azure portal에서 확인할 수 있다. 

http://localhost:9090에서 prometheus가 잘 올라간 것을 확인할 수 있다. 

![prometheus-up-and-running](https://i.imgur.com/mfUlkUK.png)

사용법은 아직 잘 모르겠다. 천천히 알아가보도록 한다. 
port forwarding으로 grafana도 확인할 수 있다. 

```bash
kubectl port-forward -n CUSTOM_NAMESPACE promethus-grafana-5b8fc75d5c-2sfmk 3000
```

로그인 아이디와 비밀번호가 필요한데, 이건 아래 명령어로 Secret 을 추출해서 사용하도록 한다.
```bash

kubectl get secret -n CUSTOM_NAMESPACE promethus-grafana-5b8fc75d5c-2sfmk -o=jsonpath='{.data.admin-user}' |base64 -d

kubectl get secret -n CUSTOM_NAMESPACE promethus-grafana-5b8fc75d5c-2sfmk -o=jsonpath='{.data.admin-password}' |base64 -d
```

http://localhost:3000에서 grafana가 잘 올라간 것을 확인할 수 있다. 

![grafana-up-and-running](https://i.imgur.com/Vvu1z6E.png)

이걸로 다양한 것들을 할 수 있다고 하는데, 그것도 천천히 알아보도록 한다.