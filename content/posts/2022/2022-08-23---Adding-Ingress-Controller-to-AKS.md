---
title: Azure Kubernetes(AKS)에서 Ingress Controller 적용법
date: "2022-08-23T15:41:37.121Z"
template: "post"
draft: false
slug: "/azure/adding-ingress-controller-to-aks"
category: "Kubernetes"
tags:
  - "Kubernetes"
  - "Azure"

description: "Ingress Controller를 활용한 MSA구축"
---

기존에는 하나의 VM에 여러개의 컨테이너를 띄우고, Nginx에서 해당 컨테이너들로 request를 redirect했다. 쿠버네티스도 마찬가지로 Nginx에서 적절한 pod로 클라이언트의 요청을 redirect 하려고 한다. 

쿠버네티스 용어로 클라이언트가 서버에 요청을 보낼 수 있도록 하는 것을 [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)라고 한다. 

![how-ingress-works](https://i.imgur.com/nE0AAaP.png)

클라이언트는 load balancer로 request를 보내고, 이 load balancer가 ingress에 해당 request를 전달하면, ingress는 routing rule에 따라 service로 request를 전달하고, service는 적절한 pod로 해당 request를 전달한다. 

pod는 일반적으로 yml파일의 replica설정으로 여러개가 돌고있기 때문에, routing rule이 request를 전달하는 service는 load balancer로 해두었다. 외부에서 소통을 할 수 없도록 [internal load balancer](https://docs.microsoft.com/en-us/azure/aks/internal-lb)로 설정했고. 그렇게 하기 위해서 yml파일의 metadata에 설정을 추가했다. 그리고 부여된 external IP를 고정시키기 위해 `loadBalancerIP`를 지정했다. 


```yml
metadata:
  name: SERVICE_NAME
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  loadBalancerIP: EXTERNAL_IP_ADDRESS
  ports:
  - port: 8000
  selector:
    app: SERVICE_NAME
```

쿠버네티스는 클라우드마다 설정을 따로 해야한다. 같은 일을 하는데 왜 다르게 설정해야 하는지는 의문이다. AWS는 슬래시 뒤에 단어가 aws이다. AWS는 사용자가 많아서인지 쿠버네티스 공식 문서에도 상세하게 나와있는데, Azure관련 설정은 Azure문서에서 자세히 다룬다. 

그리고 Azure내 Virtual Network에서 연동할 수 있도록 [Private Link Service](https://cloud-provider-azure.sigs.k8s.io/topics/pls-integration/) 설정을 추가했다. 

```yml
metadata:
  name: SERVICE_NAME
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
    service.beta.kubernetes.io/azure-pls-create: "true"
```

이제 해당 service의 외부 IP로는 접근이 불가능하다. 이제 nginx로 ingress를 설정하고, 해당 pod로 필요한 request를 redirect 해본다. 

[helm](https://helm.sh/)이라는 패키지를 설치해야하는데, 쿠버네티스를 관리하는데 도움을 주는 툴이라고 한다. 처음써본다. [공식문서](https://docs.microsoft.com/en-us/azure/aks/ingress-basic?tabs=azure-cli)에서 시키는대로 하면 ingress-controller pod가 잘 올라간 것을 확인할 수 있다. 

![ingress-controller-working](https://i.imgur.com/NMiiVYR.png)

하지만 기존에 올렸던 github repository와 연결된 pod들과 namespace가 다르다는 것을 깨달았다. 그래서 공식문서를 따라서 올렸던 namespace를 삭제하고 다시 설치를 시도했다. namespace만 바꿔서 `helm install`을 하려고 하니 지속적으로 아래와 같은 에러가 발생했다. 

![helm-installation-failed](https://i.imgur.com/2TPbYpc.png)

StackOverflow나 github issue들에서 성공했다는 모든것들(?)을 시도했는데 제대로 먹히지 않았다. 에러 메세지에 있는 `meta.helm.sh/release-namespace`를 열심히 찾아봤지만 실패했다.(나중에 Azure Portal에서 찾았다..........) 

이것저것 시도해보다가 에러 메세지를 다시 보니 처음에 설정한 namespace기록이 남아있는 것 같아서 공식문서대로 `helm install`을 다시 한 후에, 같은 명령어에 `helm uninstall`을 하고, namespace를 바꿔서 `helm install`을 했더니 성공했다. 

![helm-install-success](https://i.imgur.com/cRKW3qL.png)

그리고 [static ip](https://docs.microsoft.com/en-us/azure/aks/static-ip#create-a-static-ip-address)를 사용하기위해 발급받아서 추가로 설정해줬다. 

### 공식문서를 보는 것은 좋지만, 생각없이 따라하면 오늘처럼 6시간을 날릴 수 있다. 

이제 예시에 나온 것처럼 ingress controller yml파일을 생성해본다. 
```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: CUSTOM_INGRESS_NAME
  namespace: CUSTOM_NAMESPACE
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: CUSTOM_DOMAIN
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: CUSTOM_SERVICE_NAME
            port:
              number: 8000
```

설정을 적용하고 
```bash
kubectl apply -f ingress.yml
```

curl에서 로그인을 통해 200을 확인한다 
![sign-in-success](https://i.imgur.com/nwkyIhg.png)

